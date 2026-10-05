MCP-Powered-Research-Strategy-Assistant

mcp-agent is a Python framework for building AI agents with Model Context Protocol (MCP).

The idea is to keep agent development simple:

Connect an LLM to MCP servers, give it instructions, and build workflows using normal Python.

It can be used for small agents as well as larger workflows with multiple agents, human input, retries, and durable execution.

⸻

What does it do?

mcp-agent takes care of a lot of the plumbing needed to build an agent application.

Feature	What it does
MCP support	Connect agents to MCP servers and their tools
LLMs	Use OpenAI, Anthropic, Google, Azure, Bedrock, and others
Agent workflows	Build routers, planners, parallel workflows, etc.
Multiple agents	Let agents work together on a task
Human input	Pause a workflow and wait for approval
Durable execution	Run long workflows using Temporal
Observability	Logging, tracing and token tracking
MCP servers	Expose your own agent as an MCP server
Cloud	Deploy agents to mcp-agent Cloud

⸻

Quick start

1. Create a project

We recommend using uv for Python projects.

mkdir my-agent
cd my-agent
uv init
uv add "mcp-agent[openai]"

2. Add your API key

export OPENAI_API_KEY="your-api-key"

You can also put your secrets in mcp_agent.secrets.yaml.

3. Create an agent

Create main.py:

import asyncio
from mcp_agent.app import MCPApp
from mcp_agent.agents.agent import Agent
from mcp_agent.workflows.llm.augmented_llm_openai import OpenAIAugmentedLLM
app = MCPApp(name="hello_world")
async def main():
    async with app.run():
        agent = Agent(
            name="finder",
            instruction="Use filesystem and fetch to answer questions.",
            server_names=["filesystem", "fetch"],
        )
        async with agent:
            llm = await agent.attach_llm(OpenAIAugmentedLLM)
            answer = await llm.generate_str(
                "Summarize README.md in two sentences."
            )
            print(answer)
if __name__ == "__main__":
    asyncio.run(main())

Run it:

uv run main.py

The basic idea looks like this:

             +----------------+
             |      Agent     |
             +-------+--------+
                     |
              +------+------+
              |             |
        +-----v-----+ +-----v-----+
        | Filesystem| |   Fetch   |
        | MCP Server| | MCP Server|
        +-----------+ +-----------+
                     |
                +----v----+
                |   LLM   |
                +---------+

⸻

How it works

There are three main pieces:

                 mcp-agent
                     |
        +------------+------------+
        |            |            |
      Agent        LLM       MCP Servers
        |            |            |
        +------------+------------+
                     |
                  Result

Agent

An agent contains instructions and a list of MCP servers it can use.

agent = Agent(
    name="researcher",
    instruction="Research topics using web and filesystem access.",
    server_names=["fetch", "filesystem"],
)

LLM

The LLM is responsible for deciding what to do with the available tools.

llm = await agent.attach_llm(OpenAIAugmentedLLM)

You can then ask it to perform a task:

result = await llm.generate_str(
    "Find information about MCP and summarize it."
)

MCP servers

MCP servers provide the actual tools.

For example:

filesystem
fetch
slack
jira
database
github

You can use existing MCP servers or create your own.

⸻

MCP servers

MCP servers are configured in mcp_agent.config.yaml.

For example:

execution_engine: asyncio
mcp:
  servers:
    fetch:
      command: "uvx"
      args: ["mcp-server-fetch"]
    filesystem:
      command: "npx"
      args:
        - "-y"
        - "@modelcontextprotocol/server-filesystem"
        - "/path/to/your/files"
openai:
  default_model: gpt-4o

An agent can then use these servers:

agent = Agent(
    name="finder",
    instruction="Use files and the web to answer questions.",
    server_names=["filesystem", "fetch"],
)

The nice part is that the agent does not need to know how the MCP server works internally.

⸻

Agent workflows

For simple applications, one agent may be enough.

For more complicated tasks, mcp-agent provides a few common workflow patterns.

Parallel

Run several agents at the same time and combine their results.

                  Request
                     |
          +----------+----------+
          |          |          |
        Agent A    Agent B    Agent C
          |          |          |
          +----------+----------+
                     |
                  Result

Useful when different agents can work independently.

⸻

Router

Send a request to the most appropriate agent.

                    Request
                       |
                     Router
                 /      |      \
                /       |       \
          Research    Coding    Writing

Useful when your application has several specialised agents.

⸻

Orchestrator

One agent creates a plan and coordinates other agents.

                 Orchestrator
                 /     |     \
                /      |      \
           Worker A  Worker B  Worker C

Useful for larger tasks where the work needs to be broken into smaller pieces.

⸻

Evaluator / Optimizer

Generate a result, evaluate it, and improve it when needed.

       Generate
           |
           v
        Evaluate
        /      \
      Good      Bad
       |         |
       v         v
      Done     Improve
                  |
                  +----> Evaluate

⸻

Other patterns

mcp-agent also includes patterns for:

* Deep research
* Intent classification
* Multi-agent handoffs / swarm
* Map-reduce
* Workflow composition

See the workflow documentation.

⸻

Durable execution

For small applications, asyncio is usually enough.

For longer-running workflows, you can use Temporal as the execution engine.

Change:

execution_engine: asyncio

to:

execution_engine: temporal

This gives workflows support for things like:

* Retries
* Long-running tasks
* Pause and resume
* Human approval
* Recovery after failures
* Durable workflow history

The goal is that you don’t have to completely rewrite your agent when moving from a simple application to a production workflow.

⸻

Human input

Some workflows need a person to approve something before continuing.

For example:

response = await self.context.request_human_input(
    HumanInputRequest(
        prompt="Approve the draft?",
        required=True,
    )
)

The workflow can pause until the user responds.

This can be useful for:

* Approving generated content
* Reviewing emails
* Confirming actions
* Approving deployments

⸻

Build your own MCP server

You can also expose an mcp-agent application as an MCP server.

For example:

from mcp_agent.server import create_mcp_server_for_app
@app.tool
def grade_story(story: str) -> str:
    return "Report..."
server = create_mcp_server_for_app(app)
server.run_stdio()

This allows MCP clients such as Claude, Cursor, or your own applications to call your agent.

       MCP Client
           |
           v
    Your MCP Server
           |
           v
      mcp-agent
           |
      +----+----+
      |         |
    Agent     Tools

⸻

Configuration

Most applications use two configuration files.

mcp_agent.config.yaml

Used for application configuration.

For example:

execution_engine: asyncio
logger:
  transports: [console]
  level: info
mcp:
  servers:
    fetch:
      command: "uvx"
      args: ["mcp-server-fetch"]
openai:
  default_model: gpt-4o-mini

mcp_agent.secrets.yaml

Used for API keys and other secrets.

openai:
  api_key: "${OPENAI_API_KEY}"

Do not commit this file to git.

You can also use environment variables instead.

⸻

Supported LLM providers

mcp-agent is not tied to a single model provider.

Currently supported providers include:

* OpenAI
* Anthropic
* Google
* Azure
* AWS Bedrock

Install the providers you need:

uv add "mcp-agent[openai,anthropic,google]"

⸻

Logging and observability

Basic logging can be enabled in the configuration:

logger:
  transports: [console]
  level: info

You can also enable OpenTelemetry:

otel:
  enabled: true
  exporters:
    - console

Token usage can also be tracked programmatically.

⸻

CLI

mcp-agent includes a small CLI for creating and deploying projects.

Create a project:

uvx mcp-agent init

Log in:

uvx mcp-agent login

Deploy:

uvx mcp-agent deploy my-agent

⸻

Cloud

You can deploy an mcp-agent application to mcp-agent Cloud.

uvx mcp-agent login
uvx mcp-agent deploy my-agent

Cloud deployment provides a managed environment for running agents and durable workflows.

See the Cloud documentation for details.

⸻

Project structure

A small project can be as simple as:

my-agent/
├── main.py
├── mcp_agent.config.yaml
├── mcp_agent.secrets.yaml
└── pyproject.toml

As the application grows, agents and workflows can be moved into separate files.

⸻

Installation

Using uv:

uv add mcp-agent

Using pip:

pip install mcp-agent

For OpenAI:

uv add "mcp-agent[openai]"

⸻

Examples

The repository contains examples for different use cases.

Examples:
https://github.com/lastmile-ai/mcp-agent/tree/main/examples

Some of the examples include:

* Basic agents
* MCP servers
* Multiple agents
* Agent workflows
* Temporal
* Human input
* Authentication
* Cloud deployment

⸻

Documentation

The full documentation is available at:

https://docs.mcp-agent.com

Useful pages:

* Getting started
* SDK overview
* MCP integration
* Agent workflows
* Durable agents
* Cloud

For LLMs, the complete documentation is also available as:

https://docs.mcp-agent.com/llms-full.txt

⸻

Why mcp-agent?

There are already many frameworks for building AI agents.

The main reason to use mcp-agent is if you want to build around MCP without having to manually handle all the MCP connections and workflow plumbing.

It tries to keep things fairly simple:

* Write Python
* Connect MCP servers
* Give agents instructions
* Combine agents when needed
* Use Temporal when workflows become more complicated

You don’t need to build your application as a complicated graph just to add some branching logic.

⸻

Contributing

Contributions are welcome.

You can help by:

* Fixing bugs
* Adding examples
* Improving documentation
* Adding features
* Reporting issues

See CONTRIBUTING.md to get started.

⸻

License

Apache 2.0