# Agents SDK

## Introduction

This review focuses on the **OpenAI Agents SDK** as an agentic orchestration layer for building production agent workflows. The SDK is best understood as a lightweight, developer-operated framework built around agents, tools, handoffs or agents-as-tools, guardrails, sessions, context management, human approval flows, MCP integration, sandbox-related capabilities, and tracing. OpenAI describes the SDK as a small set of primitives for building agentic applications, including agents, handoffs or agents-as-tools, guardrails, and tracing support. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/ "OpenAI Agents SDK"))

From a security perspective, the Agents SDK should not be treated as a fully managed enterprise agent-control plane. It provides strong orchestration primitives, but the application owner remains responsible for authorization, tenant isolation, tool permissions, credential handling, data-retention design, audit strategy, and downstream system governance. This distinction matters because the SDK can coordinate model calls, tool calls, handoffs, sessions, and traces, but those primitives do not automatically define the organization’s full security boundary.

## Keywords

**Benefits** — Responses API, Agents SDK, lightweight orchestration, model flexibility, tool calling, hosted tools, local tools, function tools, agents-as-tools, handoffs, guardrails, tool guardrails, sessions, local context, LLM-visible context, MCP, approval flows, tracing, Admin API, projects, service accounts, audit logs.

**Drawbacks** — application-owned governance, distributed access control, tool-amplification risk, credential exposure risk, sensitive trace data, session poisoning risk, MCP trust-boundary complexity, hosted-tool retention constraints, no single built-in enterprise agent identity lifecycle, and guardrail coverage that depends on workflow placement.

# Core Capabilities

The following capabilities form the main technical basis for building agentic applications with the OpenAI Agents SDK.

1. **Lightweight agent runtime** — The SDK provides an agent loop that can manage tool invocation, return tool results to the model, and continue execution until the task is complete. OpenAI positions the SDK as higher-level than the Responses API, while still allowing developers to use the Responses API directly for lower-level flows. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/ "OpenAI Agents SDK"))
2. **Agent orchestration** — The SDK supports delegation through handoffs and agents-as-tools. This enables modular agent workflows, but it also creates additional trust boundaries because each delegated agent may receive different context, tools, instructions, and approval requirements.
3. **Model and provider flexibility** — The SDK can use OpenAI models and also supports non-OpenAI providers through its model/provider abstractions. This increases architectural flexibility, but also means model behavior, logging, retention, and security guarantees may differ across providers.
4. **Tool execution** — Tools allow agents to fetch data, call external APIs, run code, use hosted OpenAI tools, use local/runtime execution tools, wrap Python functions, call other agents as tools, and use workspace-scoped Codex tasks. OpenAI’s tools documentation explicitly separates hosted tools, local/runtime tools, function tools, agents-as-tools, and experimental Codex tooling. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/tools/ "Tools - OpenAI Agents SDK"))
5. **Human approval flows** — The SDK supports approval-based flows for tools and agents-as-tools. For example, `Agent.as_tool(..., needs_approval=...)` can pause a run and require approval or rejection before resuming. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/tools/ "Tools - OpenAI Agents SDK"))
6. **Code Interpreter and hosted execution tools** — OpenAI supports Code Interpreter through the Responses API and Agents SDK, allowing agents to execute Python code in sandboxed containers for data analysis, math, coding, file processing, graph generation, and iterative problem solving.
7. **Sessions** — Sessions provide built-in session memory for maintaining conversation history across multiple agent runs, avoiding manual history passing between turns. This is useful for multi-turn applications, but it also makes stored conversation history part of the security model. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/sessions/ "Overview - OpenAI Agents SDK"))
8. **Context management** — OpenAI distinguishes between local application context and LLM-visible context. Local context is passed through `RunContextWrapper` and is not automatically sent to the model, while LLM-visible context is the information the model actually sees when generating output. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/context/ "Context management - OpenAI Agents SDK"))
9. **Guardrails** — Guardrails can validate user input, final agent output, and custom function-tool invocations. A critical detail is that guardrails do not all run at the same workflow points: input guardrails run only for the first agent in the chain, output guardrails run only for the final-output agent, and tool guardrails run around custom function-tool invocations. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/guardrails/ "Guardrails - OpenAI Agents SDK"))
10. **MCP integration** — The SDK supports MCP-based tool integration and multiple MCP transport patterns. MCP standardizes how applications expose tools and context to language models, which improves interoperability but also expands the external integration surface. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/mcp/ "Model context protocol (MCP) - OpenAI Agents SDK"))
11. **Tracing and observability** — The SDK includes tracing for LLM generations, tool calls, handoffs, guardrails, and custom events. Tracing is enabled by default and can be disabled globally, in code, or per run. OpenAI also states that tracing is unavailable for organizations operating under Zero Data Retention policies. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/tracing/ "Tracing - OpenAI Agents SDK"))
12. **Administrative controls** — OpenAI provides organization-level and project-level administrative capabilities, including management of users, projects, service accounts, API keys, and audit logs. These controls support platform governance, but they do not replace application-level authorization inside the agent workflow. ([OpenAI Help Center](https://help.openai.com/en/articles/9687866-admin-and-audit-logs-api-for-the-api-platform "Admin and Audit Logs API for the API Platform | OpenAI Help Center"))

# Security Assessment

## 1. Agent Definition as a Security Boundary

A production agent definition should be reviewed like a policy-bearing service component. The security question is not only “what can the model answer?” but also “what can the agent cause to happen?”

This is especially important because tools turn model output into external action. An agent with access to CRM updates, payment systems, ticket creation, file operations, code execution, browser automation, or MCP servers is not merely producing text. It is operating a workflow surface that can affect real systems.

Security review should therefore cover:
- which tools are available;
- which tools have side effects;
- which tools require approval;
- which credentials are reachable;
- which user or tenant the run represents;
- which data can enter the model context;
- which data can be persisted in sessions, files, traces, vector stores, containers, or third-party services.

## 2. Tool Amplification Risk

The SDK’s tool model is powerful because it allows agents to fetch data, call APIs, run code, use hosted tools, and call other agents. However, this also creates the primary risk surface. OpenAI’s documentation states that tools let agents take actions such as fetching data, running code, calling external APIs, and using a computer. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/tools/ "Tools - OpenAI Agents SDK"))

A single user request can trigger multiple model decisions, tool calls, delegated agent calls, and state updates. If the available tools have broad permissions, the agent can amplify a small prompt into a high-impact operational action.

Recommended security stance:
- treat every tool as a privilege boundary;
- separate read-only tools from write/action tools;
- require approval for destructive or externally visible operations;
- validate all tool inputs server-side;
- validate and sanitize all tool outputs before reintroducing them into model context;
- do not rely on prompt instructions as the only tool-control mechanism.

## 3. Guardrail Boundary Limitations

Guardrails are useful, but they are not a universal workflow firewall. OpenAI documents that input guardrails run only for the first agent in the chain, output guardrails run only for the final-output agent, and tool guardrails run around custom function-tool invocations. OpenAI also explicitly recommends tool guardrails when checks are needed around each custom function-tool call in workflows involving managers, handoffs, or delegated specialists. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/guardrails/ "Guardrails - OpenAI Agents SDK"))

This has a direct security implication: an application may appear guarded while still leaving intermediate steps, delegated agents, or tool outputs insufficiently checked.

A stronger design should use:
- input guardrails for initial user intent and policy screening; 
- output guardrails for final-response validation;
- tool guardrails for tool argument and tool output validation;
- approval gates for high-impact tools;
- application-side authorization checks that do not depend on model compliance.

## 4. Context Separation and Leakage Risk

The SDK’s distinction between local context and LLM-visible context is security-positive. Local context can hold user IDs, authorization state, request metadata, dependencies, and internal service clients without automatically exposing them to the model. OpenAI states that the context object is not sent to the LLM and is purely local to the application. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/context/ "Context management - OpenAI Agents SDK"))

However, local context is only safe if the application keeps it local. It can still leak if a developer serializes it into run state, logs it, returns it through a tool, places it into instructions, or includes it in LLM-visible history.

Recommended security stance:
- keep credentials and privileged authorization metadata in local context only;
- never place secrets in model-visible instructions or conversation history;
- treat tool outputs as untrusted before exposing them to the model;
- avoid logging full context objects unless redaction is guaranteed;
- define a clear boundary between application state and model-visible state.

## 5. Session Memory and Prompt-Injection Persistence

Sessions are valuable for multi-turn applications because they maintain conversation history across agent runs. OpenAI describes sessions as built-in memory for preserving conversation history without manual handling between turns. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/sessions/ "Overview - OpenAI Agents SDK"))

The security concern is that session memory can preserve hostile or misleading content. If malicious user input, compromised tool output, or poisoned external content is stored in session history, it may influence later turns. This creates a persistence layer for prompt injection.

Security review should account for:
- what gets stored in session history;
- how long session data is retained;
- whether users and tenants are isolated by session ID;
- whether session history is trimmed, summarized, compacted, or filtered;
- whether injected instructions can survive compaction or summarization.

A strong implementation should treat session history as untrusted input, not as trusted memory.

## 6. MCP as an External Trust Boundary

MCP improves interoperability by standardizing how tools and context are exposed to language models. The security tradeoff is that each MCP server becomes an external integration boundary. OpenAI’s MCP documentation describes MCP as a way for applications to expose tools and context, and the Agents SDK supports multiple MCP transports. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/mcp/ "Model context protocol (MCP) - OpenAI Agents SDK"))

For security review, MCP servers should be treated like production APIs, not like passive plugins. Each server should be reviewed for authentication, authorization, network exposure, logging, schema validation, rate limiting, error handling, and least-privilege tool exposure.

For sensitive MCP operations, OpenAI documents approval flows using `require_approval`, including tool-specific approval policies and approval callbacks. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/mcp/ "Model context protocol (MCP) - OpenAI Agents SDK"))

Recommended security stance:

- expose the smallest possible MCP tool surface;
- require approval for sensitive MCP calls;
- validate MCP outputs before using them as context;
- avoid sending unnecessary user or tenant data to MCP servers; 
- document the retention and logging behavior of each external MCP provider.

## 7. Hosted Tools, Local Tools, and Execution Environments

The SDK supports both hosted OpenAI tools and local/runtime tools. Hosted tools may run alongside the model on OpenAI servers, while local/runtime tools execute in the developer’s environment. OpenAI also documents hosted container shell behavior, including network policy options such as disabled or allowlist modes and domain-scoped secrets in allowlist mode. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/tools/ "Tools - OpenAI Agents SDK"))

This means execution security depends heavily on tool type.

Hosted tools require review of OpenAI platform retention, container lifecycle, and data handling. Local tools require review of the developer’s runtime, filesystem access, network access, host permissions, and secrets exposure. Shell and computer-use capabilities should be considered high risk because they can interact with broader execution environments.

Recommended security stance:

- disable network access unless required;
- prefer allowlists over open egress;
- scope secrets to specific domains or tools;
- isolate tool execution from production infrastructure;
- monitor tool calls and command execution;
- require approval for filesystem, shell, browser, or external write actions.

## 8. Tracing, Observability, and Sensitive Data

Tracing is one of the SDK’s strongest operational features, but it is also a sensitive data surface. OpenAI documents that traces can include LLM generations, tool calls, handoffs, guardrails, and custom events. Tracing is enabled by default, and OpenAI states that it is unavailable for organizations operating under Zero Data Retention. ([OpenAI GitHub](https://openai.github.io/openai-agents-python/tracing/ "Tracing - OpenAI Agents SDK"))

This creates a governance tradeoff. Tracing improves debugging, monitoring, incident response, evaluation, and workflow understanding. At the same time, traces may include prompts, tool inputs, tool outputs, retrieved data, user content, and model outputs.

Recommended security stance:
- decide whether tracing is allowed before production launch;
- redact sensitive inputs and outputs where possible;
- avoid logging secrets, credentials, regulated data, or unnecessary user content;
- align tracing behavior with data-retention requirements;
- document the operational impact if tracing is disabled under ZDR.

## 9. Data Retention and ZDR Constraints

OpenAI states that API data is not used to train or improve OpenAI models unless the customer explicitly opts in. ([OpenAI Plataforma](https://platform.openai.com/docs/guides/your-data "Data controls in the OpenAI platform")) However, data-retention design is still security-relevant because different endpoints and capabilities may store application state differently.

OpenAI’s data-controls documentation states that Zero Data Retention changes behavior for `/v1/responses` and `/v1/chat/completions` by treating `store` as `false`, but also notes that some endpoints or capabilities may still store application state even when ZDR is enabled. ([OpenAI Plataforma](https://platform.openai.com/docs/guides/your-data "Data controls in the OpenAI platform")) The same documentation notes that remote MCP servers are third-party services subject to their own retention policies, and that hosted containers may write temporary application state while the container is active. ([OpenAI Plataforma](https://platform.openai.com/docs/guides/your-data "Data controls in the OpenAI platform"))

Recommended security stance:
- check retention behavior per endpoint and per tool;
- do not assume ZDR covers every capability equally;
- review vector stores, files, conversations, hosted containers, MCP servers, and tracing separately;
- document where application state is stored and when it is deleted;
- avoid sending regulated or unnecessary data into third-party MCP services.

## 10. Administrative and Platform Controls Are Necessary but Not Sufficient

OpenAI’s Admin API and Audit Log API provide useful governance controls. The Admin API can manage organization users, projects, service accounts, and API keys, while the Audit Log API can track events such as API key lifecycle, user and service account lifecycle, login/logout failures, organization configuration changes, and project lifecycle events. ([OpenAI Help Center](https://help.openai.com/en/articles/9687866-admin-and-audit-logs-api-for-the-api-platform "Admin and Audit Logs API for the API Platform | OpenAI Help Center"))

These controls help govern access to the OpenAI platform, but they do not automatically enforce business-level permissions inside the agent application. For example, an OpenAI project boundary does not decide whether a specific end user is allowed to refund an order, access a tenant record, update a ticket, or call a production API.

Recommended security stance:
- use OpenAI projects to separate environments and workloads;
- use service accounts and API keys with least privilege;
- monitor API key usage and audit logs;
- enforce business authorization in the application layer;
- avoid using one broad key across unrelated agents or tenants.

# Guidelines

## When to Use the OpenAI Agents SDK

Use the Agents SDK when:
- the team wants Python-first or TypeScript-first agent orchestration;
- the application needs tool calling, handoffs, sessions, guardrails, tracing, or approval flows;
- the team wants more structure than direct Responses API calls but still wants application-level control;
- workflows require customized orchestration logic;
- the organization can implement its own authorization, tenant isolation, logging, and compliance controls;
- tool execution can be reviewed and constrained before production;
- security teams are comfortable treating the agent application as the primary control plane.

## When to Avoid the OpenAI Agents SDK

Avoid or delay using the Agents SDK when:
- the team expects a fully managed enterprise agent runtime with centralized identity, memory, credentials, and policy governance;
- tool authorization cannot be implemented reliably in the application;
- high-impact tools would execute without approval gates;
- session isolation and session retention are not designed;
- MCP servers cannot be audited or constrained; 
- tracing requirements conflict with retention requirements;
- the application cannot safely separate local context from model-visible context;
- the team cannot monitor or review hosted tools, local tools, shell tools, code execution, or computer-use surfaces.

# Conclusion

The OpenAI Agents SDK is a strong choice for teams that want flexible, code-driven agent orchestration with built-in support for tools, handoffs, sessions, guardrails, MCP, approval flows, and tracing. Its security strength is that it gives developers practical primitives for building controlled agent workflows.

Its main weakness is that it does not remove the need for application-owned security architecture. The SDK can coordinate agent behavior, but the developer must still define the real security model: who the agent acts for, which tools it may call, what data it may access, what must be approved, what is logged, what is retained, and how tenant and business permissions are enforced.

From a senior AI security perspective, the SDK should be adopted when the organization is ready to treat agent orchestration as a privileged application runtime, not merely as a model wrapper.