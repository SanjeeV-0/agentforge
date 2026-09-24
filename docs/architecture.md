# AgentForge Architecture

## 1. Purpose

AgentForge is a production-oriented multi-agent runtime demonstrated through a
live football intelligence and forecasting application.

The system ingests football match events, maintains live match state,
coordinates specialist agents, produces quantitative forecasts, validates
agent outputs, persists execution state, and supports recovery and replay.

Football is the first application of the runtime. The underlying runtime
should remain reusable for other agentic applications.

---

## 2. Core Objective

Given a live football match:

1. Ingest match events.
2. Normalize provider-specific data into canonical events.
3. Maintain the current match state.
4. Determine whether an event requires deeper analysis.
5. Orchestrate specialist agents.
6. Allow agents to communicate through structured messages.
7. Support agent handoffs.
8. Generate quantitative Home/Draw/Away forecasts using a dedicated
   statistical/ML forecasting component.
9. Validate forecast and agent outputs.
10. Generate an explanation for meaningful forecast changes.
11. Persist execution state and checkpoints.
12. Recover from transient, permanent, and logical failures.
13. Persist appropriate memories and artifacts.
14. Support live, replay, and simulation modes.
15. Provide observability and evaluation data.

---

## 3. Design Principles

### 3.1 LLMs are not the source of truth for numerical forecasts

The forecasting component produces numerical probabilities.

Agents reason about the forecast, contextual information, and evidence.

### 3.2 State and memory are different

State contains everything required to continue the current execution.

Memory contains reusable information across executions.

### 3.3 Events are the primary driver

The system is event-driven.

Not every event should trigger an expensive multi-agent workflow.

### 3.4 Provider-specific data stays at the boundary

Football-data providers are accessed through adapters.

The rest of the system consumes canonical events.

### 3.5 Failure recovery is a first-class capability

Runs must be resumable from checkpoints.

Transient failures may be retried.

Logical failures may trigger recovery or handoff.

### 3.6 Agents must have bounded responsibilities

Agents should not arbitrarily modify system state, memory, forecasts,
or external systems.

### 3.7 Observability is part of the runtime

Agent execution, tool calls, state transitions, retries, handoffs,
latency, tokens, and errors should be traceable.

---

## 4. High-Level Architecture

```text
                         User / Dashboard
                                |
                                v
                            FastAPI
                                |
                                v
                       AgentForge Runtime
                                |
             +------------------+------------------+
             |                  |                  |
             v                  v                  v
        Event System        Agent System       Persistence
             |                  |                  |
             v                  v                  v
       Match State          LangGraph          PostgreSQL
             |                  |              Blob Storage
             v                  |              Vector/Search
       Forecast Engine         |
             |                 A2A
             |                  |
             +--------+---------+
                      |
                      v
                Validation
                      |
                      v
                 Explanation