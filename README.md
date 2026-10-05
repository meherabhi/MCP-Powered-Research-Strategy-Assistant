MCP Agent 

mcp-agent is a simple Python framework for building AI agents using the Model Context Protocol (MCP).

The main idea is simple:

MCP is enough to build useful agents, and simple patterns are often better than complicated agent architectures.

It helps you connect LLMs to tools, MCP servers, workflows, and other agents without having to build everything from scratch.

What does it do?

With mcp-agent, you can:

Feature	What it means
Build agents	Create agents that can use tools and MCP servers
Connect MCP servers	Use tools, resources, prompts, and other MCP features
Use different LLMs	Connect providers such as OpenAI, Anthropic, Google, and others
Build workflows	Run agents in parallel, route requests, plan tasks, and more
Run agents reliably	Use Temporal for long-running and durable agents
Human input	Pause an agent and ask a person for input
Create MCP servers	Turn your agents and workflows into MCP servers
Deploy to the cloud	Deploy agents and MCP applications using mcp-agent Cloud

Learn more in the documentation.

⸻

Quick start

Install

Using uv:

uv add "mcp-agent"

Or using pip:

pip install mcp-agent

You can also install provider-specific dependencies when needed.

A small example

from mcp_agent.app import MCPApp
from mcp_agent.agents.agent import Agent
from mcp_agent.workflows.llm.augmented_llm_openai import OpenAIAugmentedLLM
app = MCPApp(name="my-agent")
async def main():
    async with app.run() as agent_app:
        context = agent_app.context
        agent = Agent(
            name="assistant",
            instruction="You are a helpful assistant.",
            server_names=["fetch"],
        )
        async with agent:
            llm = await agent.attach_llm(OpenAIAugmentedLLM)
            result = await llm.generate_str(
                "Tell me something interesting about MCP."
            )
            print(result)
if __name__ == "__main__":
    import asyncio
    asyncio.run(main())

See the getting started guide for a complete example.

⸻

How does it work?

The basic structure looks like this:

                 Your Application
                       |
                       v
                  mcp-agent
                       |
          +------------+------------+
          |                         |
          v                         v
       Agent                    Workflow
          |                         |
          +------------+------------+
                       |
                       v
                     LLM
                       |
                       v
                  MCP Servers
                       |
             +---------+---------+
             |         |         |
            Tools   Resources  Prompts

You can use one agent, several agents, or combine agents into larger workflows.

⸻

Agents

An Agent connects an LLM with one or more MCP servers.

For example, an agent could have access to:

* filesystem tools
* web search
* databases
* APIs
* internal company tools
* other MCP servers

Agents can also be exposed as MCP servers themselves.

Read more about agents.

⸻

LLMs

mcp-agent provides an Augmented LLM interface.

This gives the LLM access to:

* MCP tools
* MCP resources
* prompts
* agent instructions
* application context

Different LLM providers can be used without changing the rest of your application.

See the Augmented LLM documentation.

⸻

MCP servers

MCP is the main way mcp-agent connects agents to tools and data.

It supports the MCP features you would expect, including:

* Tools
* Resources
* Prompts
* Notifications
* OAuth
* Sampling
* Elicitation
* Roots

You can connect to existing MCP servers or build your own.

Learn more about MCP.

Connecting to MCP servers

You can configure MCP servers in your application configuration.

For example:

mcp:
  servers:
    fetch:
      command: "uvx"
      args:
        - "mcp-server-fetch"

See connecting to MCP servers.

⸻

Agent workflows

One of the main reasons to use mcp-agent is that agents can be combined into workflows.

The project focuses on simple patterns that are easy to understand and modify.

More examples are available in the workflow documentation.

Parallel workflows

Run multiple agents at the same time.

             Input
               |
       +-------+-------+
       |       |       |
       v       v       v
    Agent A Agent B Agent C
       |       |       |
       +-------+-------+
               |
               v
             Result

This is useful when several independent tasks can be completed at the same time.

See the Map-Reduce example.

⸻

Router

A router decides which agent should handle a request.

                Request
                   |
                   v
                 Router
              /    |    \
             /     |     \
            v      v      v
         Agent A Agent B Agent C

This is useful when different types of requests need different agents.

See the Router documentation.

⸻

Intent classifier

An intent classifier first figures out what the user wants and then sends the request to the right workflow.

See the Intent Classifier documentation.

⸻

Orchestrator and workers

One agent can coordinate several other agents.

                 Orchestrator
                /      |      \
               /       |       \
              v        v        v
           Worker A  Worker B  Worker C

The orchestrator can break a large task into smaller tasks and distribute them to other agents.

⸻

Deep research

For research-heavy tasks, agents can:

1. Plan the research
2. Search for information
3. Delegate work
4. Review results
5. Combine the results
6. Produce a final answer

See the Deep Research pattern.

⸻

Evaluator-optimizer

One agent creates an answer and another agent checks it.

       Task
         |
         v
     Generator
         |
         v
     Evaluator
       /   \
      /     \
   Good?    No
    |        |
    v        v
 Result    Improve
             |
             +----> Generator

This is useful when the quality of the output matters more than simply getting a first answer.

See the Evaluator-Optimizer documentation.

⸻

Swarm

A swarm lets multiple agents work together without requiring one central orchestrator.

See the Swarm documentation.

⸻

Durable agents

Some agent tasks take a long time.

For example:

* research jobs
* data processing
* multi-step workflows
* tasks that need human approval
* jobs that should survive restarts

mcp-agent can use Temporal for durable execution.

This allows workflows to:

* pause
* resume
* retry
* recover after failures
* run for long periods

The agent code does not need to change significantly when moving to durable execution.

Learn more in the durable agents documentation.

Examples are available in the Temporal examples.

⸻

Human input

Agents sometimes need a person to make a decision.

For example:

Agent
  |
  v
Needs approval
  |
  v
Ask human
  |
  v
Continue workflow

mcp-agent supports workflows that pause and wait for human input.

See the human-in-the-loop documentation.

⸻

Build your own MCP server

You can also use mcp-agent to create MCP servers.

This means you can build an agent or workflow and expose it as an MCP server that other applications can connect to.

Your Agent / Workflow
          |
          v
     MCP Server
          |
     +----+----+
     |         |
     v         v
  ChatGPT   Other Apps

See Agent as an MCP Server.

There are also examples in the MCP examples directory.

⸻

Configuration

A basic application configuration can look like this:

execution_engine: asyncio
logger:
  type: file
  path: "mcp-agent.log"
mcp:
  servers:
    fetch:
      command: "uvx"
      args:
        - "mcp-server-fetch"
openai:
  default_model: "gpt-4o"

Configuration can also be done directly in Python.

See the configuration documentation.

Secrets can be configured separately so that API keys do not need to be stored directly in your code.

See specifying secrets.

⸻

Supported LLM providers

mcp-agent supports several LLM providers through its augmented LLM interfaces.

Depending on your setup, you can use providers such as:

* OpenAI
* Anthropic
* Google
* Azure OpenAI
* Amazon Bedrock
* OpenRouter
* Ollama
* LiteLLM

The goal is to keep the agent and workflow code mostly independent of the model provider.

⸻

Logging and observability

mcp-agent provides structured logging and observability support.

You can use:

* structured logs
* OpenTelemetry
* token counting
* workflow-level information
* agent execution information

See the observability documentation.

⸻

CLI

The project includes a command-line interface.

For example:

uvx mcp-agent init

This creates a new project.

You can also deploy an agent:

uvx mcp-agent deploy my-agent

See the CLI documentation.

⸻

Cloud

mcp-agent also has a cloud option for deploying agents and MCP applications.

You can:

* deploy agents
* run agents as MCP servers
* connect applications to deployed agents
* use managed infrastructure

See the Cloud documentation.

Cloud quickstart

Start here:

https://docs.mcp-agent.com/get-started/cloud

There are also cloud examples.

⸻

Authentication

Authentication can be handled using:

* API keys and secrets
* OAuth
* server authentication

See the authentication documentation.

For MCP server authentication, see server authentication.

⸻

Composing applications

Larger applications can be built by combining smaller agents and workflows.

This makes it possible to keep each part of the system relatively simple.

See the composition documentation.

⸻

Project structure

A typical project can look something like:

my-agent/
├── mcp_agent.config.yaml
├── main.py
├── agents/
├── workflows/
├── servers/
└── README.md

The exact structure is up to you.

⸻

Installation

Using uv:

uv add "mcp-agent"

Using pip:

pip install mcp-agent

We recommend using uv for new Python projects.

⸻

Examples

The repository contains examples for different use cases.

You can find them here:

https://github.com/lastmile-ai/mcp-agent/tree/main/examples

Some useful examples include:

* MCP servers
* workflows
* parallel agents
* routers
* deep research
* Temporal
* Cloud
* human-in-the-loop workflows

⸻

Documentation

The main documentation is here:

https://docs.mcp-agent.com

Some useful pages:

* Welcome
* Quickstart
* SDK overview
* Core components
* Agents
* Augmented LLMs
* Workflows
* Configuration
* CLI
* MCP integration
* Cloud

For LLMs and AI coding tools, the project also provides:

* llms.txt
* llms-full.txt
* MCP documentation server

⸻

Why mcp-agent?

There are many ways to build AI agents.

mcp-agent focuses on keeping things simple.

The main ideas are:

1. Use MCP for tools and integrations.
2. Use simple workflow patterns instead of large agent frameworks.
3. Keep agents composable.
4. Make workflows easy to test and understand.
5. Support durable execution when needed.
6. Allow agents to work with different LLM providers.

The workflow patterns are based in part on Anthropic’s Building Effective Agents.

⸻
