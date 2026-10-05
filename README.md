mcp-agent

mcp-agent is a Python framework for building AI agents using Model Context Protocol (MCP).

The main idea is simple: connect an LLM to MCP servers and build useful agent workflows without writing all the MCP connection and orchestration code yourself.

It supports simple agents as well as more complex workflows with multiple agents, retries, human input, and durable execution.

What can you do with mcp-agent?

With mcp-agent, you can:

* Connect an agent to one or more MCP servers
* Give agents access to tools such as files, web pages, APIs, Slack, Jira, etc.
* Use different LLM providers
* Run multiple agents in parallel
* Route a request to the right agent
* Create planner/worker workflows
* Add evaluation and retry steps
* Pause a workflow and wait for human input
* Run workflows using Temporal for durable execution
* Expose your agent as an MCP server

The framework is built around normal Python code, so you don’t need to build your application as a large graph of nodes and edges.

⸻

A simple example

Here is a small agent that can read files and fetch web pages:

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

The agent gets its tools from the MCP servers:

Agent
  |
  +-- filesystem MCP server
  |
  +-- fetch MCP server
  |
  +-- LLM

You don’t have to manually manage the MCP server connections.

⸻

Getting started

We recommend using uv for Python projects.

Create a project:

mkdir my-agent
cd my-agent
uv init
uv add "mcp-agent[openai]"

Set your OpenAI API key:

export OPENAI_API_KEY="your-api-key"

Then create your Python file and run it:

uv run main.py

You can also install the package with pip:

pip install mcp-agent

Other LLM providers are supported through optional packages.

For example:

uv add "mcp-agent[openai,anthropic,google]"

⸻

MCP servers

MCP servers provide the tools that an agent can use.

For example, you might have:

Agent
 |
 +-- Filesystem
 +-- Fetch
 +-- Slack
 +-- Jira
 +-- Database

You configure the servers in mcp_agent.config.yaml.

A simple configuration could look like this:

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

API keys and other secrets can be kept in:

mcp_agent.secrets.yaml

Don’t commit this file to git.

⸻

Agents

An agent is basically an instruction plus a set of MCP servers it can use.

For example:

agent = Agent(
    name="researcher",
    instruction="Research topics using web and filesystem access.",
    server_names=["fetch", "filesystem"],
)

You can then attach an LLM:

async with agent:
    llm = await agent.attach_llm(OpenAIAugmentedLLM)
    result = await llm.generate_str(
        "Find information about MCP and summarize it."
    )
    print(result)

⸻

Different LLM providers

mcp-agent can work with different model providers.

For example:

* OpenAI
* Anthropic
* Google
* Azure
* AWS Bedrock

The LLM is separate from the MCP servers, so you can change the model without changing the tools your agent uses.

⸻

Agent workflows

Sometimes one agent is enough.

For larger tasks, you can combine agents into workflows.

Some of the patterns included in mcp-agent are:

Parallel

Give the same task to multiple agents and combine their results.

             +-- Agent A --+
Request -----+-- Agent B --+---- Result
             +-- Agent C --+

Router

Send a request to the agent that is best suited for it.

                 +-- Research agent
Request --> Router
                 +-- Coding agent
                 +-- Writing agent

Orchestrator

One agent creates a plan and assigns work to other agents.

              Orchestrator
             /      |      \
            /       |       \
       Agent A   Agent B   Agent C

Evaluator / Optimizer

Generate something, evaluate it, and improve it if necessary.

Generate
   |
   v
Evaluate
   |
   +---- Good ----> Done
   |
   +---- Bad ------> Improve

There are also patterns for deep research, intent classification, and multi-agent handoffs.

⸻

Durable workflows

For simple applications, mcp-agent can run using normal Python asyncio.

For longer-running applications, you can use Temporal.

Set:

execution_engine: temporal

This allows workflows to survive things like:

* Process restarts
* Temporary failures
* Long waits
* Retries
* Human approval steps

The workflow code can stay mostly the same.

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

The workflow can pause and continue after the person responds.

This can be useful for things like:

* Approving emails
* Reviewing generated content
* Confirming an action
* Approving a deployment

⸻

Using an agent as an MCP server

You can also expose your application as an MCP server.

For example:

from mcp_agent.server import create_mcp_server_for_app
@app.tool
def grade_story(story: str) -> str:
    return "Report..."
server = create_mcp_server_for_app(app)
server.run_stdio()

This means another MCP client can use your application as an MCP server.

For example:

Claude / Cursor / Other MCP Client
                |
                v
          Your MCP Server
                |
                v
          mcp-agent app
                |
        +-------+-------+
        |               |
     Agent          MCP tools

⸻

Configuration

Most projects use two configuration files.

mcp_agent.config.yaml

This contains things such as:

* MCP servers
* LLM configuration
* Logging
* Execution engine
* OpenTelemetry settings

Example:

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

This is where you can keep API keys and other secrets.

For example:

openai:
  api_key: "${OPENAI_API_KEY}"

Keep this file out of source control.

⸻

Logging and observability

mcp-agent supports structured logging and OpenTelemetry.

A basic logging configuration is:

logger:
  transports: [console]
  level: info

You can also enable OpenTelemetry:

otel:
  enabled: true
  exporters:
    - console

There is also support for tracking token usage from LLM calls.

⸻

CLI

The project includes a CLI.

Create a new project:

uvx mcp-agent init

Log in to the cloud:

uvx mcp-agent login

Deploy an application:

uvx mcp-agent deploy my-agent

⸻

Cloud deployment

You can deploy mcp-agent applications to mcp-agent Cloud.

Cloud deployment provides a managed environment for running agents and workflows.

For example:

uvx mcp-agent login
uvx mcp-agent deploy my-agent

See the cloud documentation for more information.

⸻

Project structure

A small project might look like this:

my-agent/
|
+-- main.py
+-- mcp_agent.config.yaml
+-- mcp_agent.secrets.yaml
+-- pyproject.toml

As the project grows, you can split agents and workflows into separate Python files.

⸻

Documentation

The full documentation is available at:

https://docs.mcp-agent.com

Some useful pages:

* Getting started: https://docs.mcp-agent.com/get-started/overview
* SDK overview: https://docs.mcp-agent.com/mcp-agent-sdk/overview
* MCP integration: https://docs.mcp-agent.com/mcp/overview
* Agent workflows: https://docs.mcp-agent.com/mcp-agent-sdk/effective-patterns/overview
* Durable agents: https://docs.mcp-agent.com/mcp-agent-sdk/advanced/durable-agents
* Cloud: https://docs.mcp-agent.com/cloud/overview

There is also a full documentation file for LLMs:

https://docs.mcp-agent.com/llms-full.txt

⸻

Examples

The repository contains examples for different use cases.

You can find them here:

https://github.com/lastmile-ai/mcp-agent/tree/main/examples

Some examples cover:

* Basic agents
* MCP servers
* Multiple agents
* Agent workflows
* Temporal
* Human input
* Authentication
* Cloud deployment

⸻

Why use mcp-agent?

There are many agent frameworks available.

The main reason to use mcp-agent is if you want to build your application around MCP without having to write all the MCP connection and workflow code yourself.

The main things it provides are:

* Simple Python API
* MCP server support
* Multiple LLM providers
* Reusable agent workflows
* Multi-agent patterns
* Human input
* Durable execution with Temporal
* Logging and observability
* Ability to expose your own agent as an MCP server

The goal is to keep the code simple and let developers build agents using normal Python.

⸻

Contributing

Contributions are welcome.

You can contribute by:

* Fixing bugs
* Adding examples
* Improving documentation
* Adding features
* Reporting issues

See CONTRIBUTING.md for more information.

⸻

License

This project is licensed under the Apache 2.0 License.

⸻

Links

* Documentation: https://docs.mcp-agent.com
* GitHub: https://github.com/lastmile-ai/mcp-agent
* MCP: https://modelcontextprotocol.io
* Examples: https://github.com/lastmile-ai/mcp-agent/tree/main/examples