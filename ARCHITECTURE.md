Project Forge — Architecture

Status: Initial design
Version: 0.1.0
Principle: One control centre. Multiple independent agents. Shared context only when permitted.

1. Purpose

Forge is a provider-independent orchestration platform that lets the user communicate with multiple AI agents through one interface.

Forge coordinates agents such as Matrix and JARVIS without merging their identities, conversation histories, or private context.

Forge is the control plane—not necessarily the AI brain.

2. Core principles

1. One interface: The user communicates with Forge.
2. Separate histories: Each agent retains its own conversations.
3. Controlled shared memory: Projects can have shared context, but access is explicit.
4. Honest delivery: Forge never claims an agent received or completed a task without confirmation.
5. Provider independence: Models and providers can be replaced without redesigning Forge.
6. User authority: The user controls routing, permissions, and consequential actions.
7. Traceability: Important operations are recorded in an audit log.

3. System architecture

Forge Core

Coordinates the system, manages requests, and maintains task state.

Forge Router

Determines which agent or agents are suitable for a request. Routing decisions must be visible and explainable.

Forge Memory

Maintains three distinct memory scopes:

- Agent memory: Private to one agent.
- Conversation memory: Belongs to one agent conversation.
- Project memory: Shared only with authorized agents.

Forge Connectors

Provides standardized connections to external agents, models, APIs, and local processes.

Forge Audit

Records requests, routing decisions, delivery updates, permission changes, and results.

Forge Dashboard

Provides the unified interface for conversations, projects, agents, tasks, and system status.

4. Agent identity and conversations

Every agent has a unique "agent_id".

Every conversation has a unique "thread_id" and belongs to one agent.

Example:

- Matrix → Matrix conversation history
- JARVIS → JARVIS conversation history
- Research Agent → Research Agent conversation history

Forge must never silently combine these histories.

An agent can access shared project context only when its permissions allow it.

5. Message lifecycle

Every outgoing task follows an explicit state machine:

"CREATED → QUEUED → SENT → DELIVERED → ACKNOWLEDGED → RUNNING → COMPLETED"

Possible alternative terminal states:

- "FAILED"
- "CANCELLED"
- "EXPIRED"

Not every connector supports every state. Forge must report the strongest state it can actually verify.

For example, a queued message is not a delivered message. A delivered message is not a completed task.

6. Multi-agent coordination

Forge may send a task to one agent or several agents.

For multi-agent tasks:

1. Forge identifies suitable agents.
2. It records the intended recipients.
3. It dispatches the task through their connectors.
4. It collects responses independently.
5. It preserves each response's source and status.
6. It produces a combined report.

Forge must not invent missing responses or attribute one agent's answer to another.

7. Permissions and safety

Permissions are defined per agent, project, connector, and operation.

Potential permission categories:

- Read project context
- Read agent conversations
- Write project memory
- Send tasks to agents
- Modify project files
- Execute local operations
- Access external services

Consequential operations require user approval before execution.

Secrets and credentials must never be placed in ordinary conversation memory or exposed to unauthorized agents.

8. Audit records

Each important operation should record:

- Event ID
- Timestamp
- User request or task ID
- Source agent or user
- Target agent
- Action
- Delivery state
- Permission decision
- Result or error

Audit records should be append-only wherever practical.

9. Provider independence

Forge communicates through connector interfaces rather than depending on one model provider.

A connector should expose a consistent contract for:

- Agent identification
- Message submission
- Delivery status
- Response retrieval
- Capability reporting
- Error reporting

This allows Matrix, JARVIS, and future agents to use different models and runtimes.

10. Initial implementation strategy

Build in stages:

Stage 1 — Foundation

- Project structure
- Configuration
- Agent registry
- Task and message data models

Stage 2 — Memory

- Separate conversations
- Project memory
- Permission checks

Stage 3 — Routing

- Agent capability registry
- Single-agent routing
- Multi-agent dispatch

Stage 4 — Connectors

- Mock connector
- Matrix connector
- JARVIS connector

Stage 5 — Interface and audit

- Unified dashboard
- Task status display
- Audit history

11. Architectural rule

Forge must always distinguish between what it requested, what it transmitted, what an agent acknowledged, and what an agent actually completed.

Trustworthy coordination is more important than pretending everything worked.
