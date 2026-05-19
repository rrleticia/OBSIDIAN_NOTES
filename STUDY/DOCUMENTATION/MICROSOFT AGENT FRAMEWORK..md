# Agent Framework

## Introduction

This review focuses on **Microsoft Agent Framework** as an agentic application framework for building agents and multi-agent workflows in .NET and Python. Microsoft positions Agent Framework around two primary capability groups: **Agents**, which use LLMs to process inputs, call tools and MCP servers, and generate responses; and **Workflows**, which connect agents and functions for multi-step tasks with type-safe routing, checkpointing, and human-in-the-loop support. The framework also includes model clients, sessions, context providers, middleware, and MCP clients as foundational building blocks. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/overview/?pivots=programming-language-python "Microsoft Agent Framework Overview | Microsoft Learn"))

From a security perspective, Microsoft Agent Framework should not be treated as a complete enterprise agent-control plane by itself. It provides strong orchestration, workflow, middleware, state, tool, and observability primitives, but Microsoft explicitly places responsibility on the application owner for reviewing third-party systems, data-sharing boundaries, permissions, approvals, responsible AI mitigations, safety systems, quality, reliability, security, and trustworthiness. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/overview/?pivots=programming-language-python "Microsoft Agent Framework Overview | Microsoft Learn"))

## Keywords

**Benefits** — Agents, workflows, .NET, Python, Microsoft Foundry, Azure OpenAI, OpenAI, Anthropic, Ollama, GitHub Copilot, Copilot Studio, function tools, tool approval, Code Interpreter, File Search, Web Search, hosted MCP tools, local MCP tools, Foundry toolboxes, sessions, context providers, memory, RAG, middleware, OpenTelemetry, type-safe workflows, checkpointing, human-in-the-loop, A2A, service-managed Foundry agents.

**Drawbacks** — application-owned responsible AI mitigations, third-party system responsibility, provider-dependent feature coverage, credential exposure risk, MCP trust-boundary complexity, tool-amplification risk, state persistence risk, sensitive telemetry risk, session/context poisoning risk, workflow checkpoint sensitivity, and no automatic replacement for business-level authorization.

# Core Capabilities

The following capabilities form the main technical basis for building agentic applications with Microsoft Agent Framework.

1. **Agent runtime** — Agent Framework supports agents that use LLMs to process input, call tools and MCP servers, and generate responses. Microsoft documents a structured runtime model that coordinates user interaction, model inference, and tool execution. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/overview/?pivots=programming-language-python "Microsoft Agent Framework Overview | Microsoft Learn"))
2. **Workflow orchestration** — Workflows are predefined execution structures that can include agents, functions, external integrations, and human interactions. Unlike agents, which dynamically choose steps based on LLM reasoning, workflows define the execution path more explicitly. However inteligent agents may be incorporated into workflows given necessity. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/workflows/ "Microsoft Agent Framework Workflows | Microsoft Learn"))
3. **Model and provider flexibility** — The framework supports several providers, including Azure OpenAI, OpenAI, Microsoft Foundry, Anthropic, Ollama, GitHub Copilot, Copilot Studio, A2A, and custom providers. Microsoft also documents that feature coverage differs by provider. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/providers/ "Providers Overview | Microsoft Learn"))
4. **Microsoft Foundry integration** — Agent Framework supports both code-first Foundry Responses agents and service-managed Foundry agents. The code-first pattern lets the application provide the model, instructions, tools, and conversation loop; the service-managed pattern supports versioned agent definitions managed through Foundry portal or service APIs. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/providers/microsoft-foundry?utm_source=chatgpt.com "Microsoft Foundry"))
5. **Tool execution** — Tools extend agents by allowing interaction with external systems, code execution, data search, web search, MCP servers, and hosted tool configurations. Microsoft documents tool categories including function tools, tool approval, Code Interpreter, File Search, Web Search, hosted MCP tools, local MCP tools, and Foundry toolboxes. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/ "Tools Overview | Microsoft Learn"))
6. **Human approval flows** — Function tools can require human approval before execution. In Python, Microsoft documents `approval_mode="always_require"` for approval-required tools, and the caller is responsible for collecting the approval or rejection and passing it back into the agent run. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/tool-approval "Using function tools with human in the loop approvals | Microsoft Learn"))
7. **Code Interpreter and hosted execution tools** — Code Interpreter allows agents to write and execute code in a sandboxed environment for tasks such as data analysis, mathematical computation, and file processing. Availability depends on the underlying provider. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/code-interpreter "Code Interpreter | Microsoft Learn"))
8. **Sessions and conversation state** — `AgentSession` is the state container used across agent runs. It can contain local session state, a local session identifier, a service-side conversation identifier, and mutable state shared with context or history providers. Microsoft warns that sessions are agent/service-specific and that reusing a session with a different agent configuration or provider can lead to invalid context. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/session "Session | Microsoft Learn"))
9. **Context providers, memory, and RAG** — Context providers run around each invocation to add context before execution and process data after execution. They can support memory enrichment, history loading, dynamic instructions, retrieved documents, RAG, and custom state extraction. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/context-providers "Context Providers | Microsoft Learn"))
10. **Storage and history control** — Storage controls where conversation history lives, how much history is loaded, and how reliably sessions can be resumed. Microsoft documents local session state, service-managed storage, custom history providers, reducers for in-memory history, and full session serialization for persistence across restarts. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/storage "Storage | Microsoft Learn"))
11. **Middleware** — Middleware can intercept, inspect, modify, or enhance agent behavior. Microsoft documents agent-run middleware, function-calling middleware, and chat-client middleware, with use cases including logging, security validation, error handling, and result transformation. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/middleware/ "Agent Middleware | Microsoft Learn"))
12. **Pipeline architecture** — The Python `Agent` pipeline includes agent middleware and telemetry, raw agent logic, context providers, function invocation, chat middleware and telemetry, and a provider-specific raw chat client. This layered architecture is important because different controls apply at different points in the execution path. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/agent-pipeline "Agent Pipeline Architecture | Microsoft Learn"))
13. **MCP integration** — Agent Framework supports MCP servers as a way to provide tools and contextual data to LLMs. Microsoft documents hosted MCP tools managed by Foundry and local/custom MCP tools. Hosted MCP can include server-side managed execution, persistent agents, allowed tool lists, and approval workflows. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/hosted-mcp-tools "MCP and Foundry Agents | Microsoft Learn"))
14. **Workflow checkpointing** — Workflows can save state through checkpoints. Microsoft documents that checkpoints capture the state of executors, pending messages, pending requests and responses, and shared state; Python checkpoint storage options include in-memory, file, and Cosmos DB-backed storage. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/workflows/checkpoints?utm_source=chatgpt.com "Microsoft Agent Framework Workflows - Checkpoints"))
15. **Observability** — Agent Framework integrates with OpenTelemetry and emits traces, logs, and metrics according to OpenTelemetry GenAI semantic conventions. Microsoft warns that enabling sensitive data can expose prompts, responses, function-call arguments, and tool results in logs and traces. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/observability "Observability | Microsoft Learn"))
16. **Administrative controls** — Microsoft Agent Framework does not itself provide a standalone organization-admin layer. When used with Microsoft Foundry, administrative controls are provided through Microsoft Foundry, Azure RBAC, Microsoft Entra ID, and Azure governance services. These controls include management of Foundry resources and projects, role assignments for users, groups, service principals, managed identities, agent identities, project connections, policies, monitoring, and auditability. These platform controls support enterprise governance, but they do not replace application-level authorization inside the agent workflow.

# Security Assessment

## 1. Agent Definition as a Security Boundary

A production Microsoft Agent Framework agent should be reviewed like a policy-bearing service component. The security question is not only “what can the model answer?” but also “what can this agent cause to happen?”

This matters because Agent Framework agents may have function tools, MCP tools, Code Interpreter access, File Search, Web Search, service-managed history, context providers, and external provider integrations. Microsoft explicitly describes tools as mechanisms that let agents interact with external systems, execute code, search data, and more. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/ "Tools Overview | Microsoft Learn"))

Security review should therefore cover:

- which model provider is used;
- which tools are available;
- which tools have side effects;
- which tools require approval;
- which MCP servers are connected;
- which credentials are passed to tools or providers;
- whether the agent is code-first or service-managed;
- what session and history storage model is used;
- what context providers inject into the model;
- what telemetry captures;
- what workflow checkpoints persist.

## 2. Tool Amplification Risk

Agent Framework’s tool system is powerful because it allows agents to call custom functions, execute code, search files, search the web, and integrate with hosted or local MCP servers. This is also the primary risk surface. A small prompt can trigger multiple model calls, tool calls, MCP calls, workflow steps, or external actions. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/?utm_source=chatgpt.com "Tools Overview"))

A stronger implementation should:

- separate read-only tools from write/action tools;
- require approval for destructive or externally visible operations;
- validate tool inputs outside the model;
- validate and sanitize tool outputs before reintroducing them into context;
- avoid granting broad credentials to tools;
- avoid assuming prompt instructions are sufficient access control;
- monitor function-calling middleware and telemetry for abnormal tool behavior.

## 3. Human Approval Is Useful but Application-Owned

Microsoft supports human-in-the-loop approval for function tools, including approval-required function calls that return a user-input request instead of executing immediately. However, the caller of the agent is responsible for detecting approval requests, presenting tool name and arguments to the user, collecting approval or rejection, and passing the response back to the agent. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/tool-approval "Using function tools with human in the loop approvals | Microsoft Learn"))

This means approval is not a passive feature that automatically secures an application. The application must implement the approval UX and policy correctly.

Recommended security stance:

- use approval for high-impact tools;
- show the exact tool name and arguments to the approver;
- apply server-side authorization before honoring approval;
- log approval decisions;
- treat approval as necessary but not sufficient;
- avoid letting the same model-generated content define both the action and the approval policy.

## 4. Middleware as a Security Control Point

Middleware is one of Agent Framework’s strongest security-relevant primitives. Microsoft documents middleware for logging, security validation, error handling, result transformation, agent-run interception, function-call interception, and chat-client interception. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/middleware/ "Agent Middleware | Microsoft Learn"))

The key advantage is that middleware lets developers implement controls outside the model’s reasoning process. This is important because prompt instructions can be bypassed, misunderstood, or contradicted by tool outputs.

Recommended security stance:

- use agent middleware for input/output validation;
- use function-calling middleware for tool argument and tool-result checks;
- use chat-client middleware for provider-level policy enforcement;
- keep security checks deterministic where possible;
- ensure streaming and non-streaming paths are both covered;
- avoid placing all guardrail behavior only in instructions.

## 5. Context Providers and Memory Injection Risk

Context providers are powerful because they proactively inject memory, conversation history, retrieved documents, dynamic instructions, and other context before the model runs. Microsoft documents that context providers can also extract state after runs and even dynamically add or remove tools on each turn. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/context-providers "Context Providers | Microsoft Learn"))

The security concern is that context providers can silently change what the model sees or what tools are available. If context comes from untrusted documents, user profiles, memory stores, vector databases, or external systems, it can introduce prompt injection, sensitive-data leakage, or cross-tenant contamination.

Recommended security stance:

- treat all retrieved context as untrusted input;
- isolate memory by user and tenant;
- validate context provider outputs before model exposure;
- avoid injecting secrets or privileged authorization data into model-visible context;
- document which providers can add tools or middleware;
- separate audit/evaluation history from primary context loading.

## 6. Session and Storage Persistence Risk

Sessions and storage are central to multi-turn applications. Agent Framework supports local session state, service-managed conversation storage, custom history providers, session serialization, and history reducers. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/conversations/storage "Storage | Microsoft Learn"))

The risk is that stored history can preserve hostile instructions, sensitive content, stale permissions, or poisoned tool outputs. If that state is reused later, the attack can persist across turns or across restarts.

Security review should account for:

- whether history is local or service-managed;
- whether history is persisted across restarts;
- whether session IDs are tenant-isolated;
- whether reducers can preserve injected instructions;
- whether custom history providers enforce access control;
- whether serialized sessions are protected at rest;
- whether restored sessions use the same provider and agent configuration.

A strong implementation should treat session history as untrusted context, not trusted memory.

## 7. MCP as an External Trust Boundary

Microsoft documents MCP as a standard for providing tools and contextual data to LLMs, and Agent Framework supports both hosted MCP tools and local/custom MCP servers. For non-Microsoft MCP services, Microsoft states that prompt content or other data may be passed to the non-Microsoft service, that the application may receive data from it, and that the developer is responsible for the use of those services and data. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/tools/local-mcp-tools "Using MCP Tools | Microsoft Learn"))

This makes MCP a major trust boundary. MCP servers should be reviewed like production APIs, not like passive plugins.

Recommended security stance:

- connect only trusted MCP servers;
- prefer first-party service-hosted MCP servers over proxies;
- restrict allowed tools where possible;
- use approval for sensitive MCP tools;
- log MCP calls for audit;
- review retention, location, and handling of data sent to MCP servers;
- avoid passing unnecessary user, tenant, or credential data;
- scope API keys and OAuth tokens narrowly.

## 8. Credential Handling and Azure Identity Choices

Microsoft repeatedly warns that `DefaultAzureCredential` is convenient for development but requires careful consideration in production, recommending a specific credential such as `ManagedIdentityCredential` to avoid latency issues, unintended credential probing, and security risks from fallback mechanisms. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/ "Microsoft Agent Framework Agent Types - Microsoft Foundry | Microsoft Learn"))

This has direct production implications. Agent applications often sit close to sensitive APIs, model endpoints, storage systems, and MCP servers. Credential selection should therefore be part of the agent security review.

Recommended security stance:

- prefer managed identity for Azure production workloads;
- avoid broad API keys shared across agents;
- do not place secrets in instructions, prompts, history, or traces;
- scope credentials per environment and workload;
- rotate secrets used by MCP headers or provider clients;
- review whether credentials are persisted, logged, or passed through tool resources.

## 9. Workflow Control Improves Predictability but Adds State Risk

Workflows are useful when the process has well-defined steps and the developer needs explicit control over execution order. Microsoft describes workflows as predefined sequences that can include agents, human interactions, and external integrations, with graph-based routing, type safety, checkpointing, and multi-agent orchestration. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/workflows/ "Microsoft Agent Framework Workflows | Microsoft Learn"))

This improves predictability compared with fully open-ended agent planning. However, workflows also create durable orchestration state. Checkpoints can capture executor state, pending messages, pending requests and responses, and shared state. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/workflows/checkpoints?utm_source=chatgpt.com "Microsoft Agent Framework Workflows - Checkpoints"))

Recommended security stance:

- treat checkpoints as sensitive application state;
- encrypt durable checkpoint storage;
- isolate checkpoint storage by tenant and workflow;
- avoid storing secrets in workflow state;
- validate resumed workflow state before continuing execution;
- audit pending human requests and approvals;
- review agent-to-agent message passing inside workflows.

## 10. Observability and Sensitive Telemetry

Observability is valuable for debugging, reliability, incident response, and monitoring. Agent Framework integrates with OpenTelemetry and can emit traces, logs, and metrics. However, Microsoft warns that sensitive telemetry may include prompts, responses, function-call arguments, and results, and that sensitive data should only be enabled in development or testing. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/observability "Observability | Microsoft Learn"))

This creates a clear governance tradeoff. Better observability can mean more sensitive data exposure.

Recommended security stance:

- decide telemetry policy before production launch;
- keep sensitive-data telemetry disabled in production unless explicitly justified;
- redact prompts, responses, function arguments, and tool results;
- avoid duplicate telemetry from both agent and chat-client layers unless needed;
- align telemetry with data retention and compliance requirements;
- monitor tool-call and workflow events without over-collecting user content.

## 11. Provider Variability and Security Consistency

Agent Framework supports many providers, but provider capabilities differ. Microsoft’s provider comparison shows differences across function tools, structured outputs, Code Interpreter, File Search, MCP tools, and background responses. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/providers/ "Providers Overview | Microsoft Learn"))

This matters because a security design built for one provider may not transfer cleanly to another. For example, tool approval, hosted tools, storage behavior, or service-managed history may not be available in the same way across providers.

Recommended security stance:

- review security controls per provider, not only per framework;
- avoid assuming feature parity across Azure OpenAI, OpenAI, Foundry, Anthropic, Ollama, and Copilot integrations;
- test approval, storage, telemetry, and tool behavior after provider changes;
- document provider-specific retention and logging assumptions;
- avoid mixing providers in a workflow without explicit trust-boundary review.

## 12. Administrative and Platform Controls Are Necessary but Not Sufficient

Microsoft Agent Framework can integrate with Microsoft Foundry and Azure identity, and service-managed Foundry agents can be versioned and managed through Foundry portal or service APIs. This helps operational governance, especially when teams want centrally managed agent definitions. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/agents/providers/microsoft-foundry?utm_source=chatgpt.com "Microsoft Foundry"))

However, platform-level controls do not automatically enforce business-level permissions inside the application. A Foundry project, Azure credential, or versioned agent definition does not by itself decide whether an end user may access a customer record, issue a refund, update a ticket, or call a production API.

Recommended security stance:

- use Azure identity and Foundry project boundaries for platform governance;
- enforce business authorization in application code or middleware;
- separate development, staging, and production projects;
- version and review high-risk agent definitions;
- monitor tool and MCP usage;
- avoid using one broad credential across unrelated agents or tenants.

# Guidelines

## When to Use Microsoft Agent Framework

Use Microsoft Agent Framework when:

- the team wants .NET or Python agent development;
- the application needs both agents and explicit workflows;
- the workflow benefits from type-safe routing, checkpointing, and human-in-the-loop behavior;
- the team wants middleware as a control layer for logging, validation, policy, and transformation;
- the application needs Microsoft Foundry integration;
- the team needs provider flexibility across Microsoft and non-Microsoft model providers;
- the application needs MCP integration with clear review of external trust boundaries;
- session, memory, RAG, and context providers can be designed safely;
- observability can be governed with appropriate telemetry controls;
- the organization is ready to own application-level authorization and responsible AI mitigations.

## When to Avoid Microsoft Agent Framework

Avoid or delay using Microsoft Agent Framework when:

- the team expects the framework to automatically solve enterprise security governance;
- tool authorization cannot be enforced outside the model;
- high-impact tools would run without approval or deterministic checks;
- MCP servers cannot be reviewed, constrained, or monitored;
- session and checkpoint storage cannot be isolated or protected;
- context providers may mix data across users or tenants;
- sensitive telemetry cannot be controlled;
- provider differences are not understood;
- third-party systems would receive data without legal, compliance, or retention review;
- the application cannot implement its own responsible AI mitigations and safety systems.

# Conclusion

Microsoft Agent Framework is a strong choice for teams that want a structured, production-oriented agent framework with support for agents, workflows, tools, MCP, middleware, sessions, context providers, checkpointing, human-in-the-loop patterns, and OpenTelemetry-based observability.

Its main security strength is that it gives developers multiple control points: middleware for interception, workflows for explicit orchestration, sessions and storage for state management, context providers for memory and RAG, and approval workflows for sensitive tool calls.

Its main weakness is that these controls are primitives, not a complete security model. Microsoft’s own documentation makes clear that developers remain responsible for reviewing third-party systems, controlling data movement, configuring permissions and approvals, implementing responsible AI mitigations, and ensuring application quality, reliability, security, and trustworthiness. ([Microsoft Learn](https://learn.microsoft.com/en-us/agent-framework/overview/?pivots=programming-language-python "Microsoft Agent Framework Overview | Microsoft Learn"))

From a senior AI security perspective, Microsoft Agent Framework should be adopted when the organization is prepared to treat agents and workflows as privileged application runtimes. The framework can provide strong architecture and control surfaces, but the real security boundary must still be designed around identity, authorization, tool permissions, context handling, telemetry, storage, and external integrations.
