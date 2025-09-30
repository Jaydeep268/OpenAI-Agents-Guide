# OpenAI Agents SDK: The Complete Implementation Guide
## From Zero to Production Multi-Agent Systems

*By Jaydeep Mandge | AI & Data Science Consultant*  
*September 30, 2025*

---

## 🚨 Breaking News: Swarm is Dead. Long Live Agents SDK.

On September 2025, OpenAI quietly deprecated Swarm and released the **Agents SDK**—a production-ready evolution that changes everything about building multi-agent systems.

**15,000 developers** built with Swarm. Now they need to migrate.

This guide shows you how to build production multi-agent systems with the new Agents SDK, including:
- ✅ Complete code examples
- ✅ Migration guide from Swarm
- ✅ Real-world architectures
- ✅ Production deployment patterns
- ✅ Performance optimization

---

## Table of Contents

1. [Why Agents SDK Replaced Swarm](#why-replaced)
2. [The 5 Core Primitives](#primitives)
3. [Installation & Setup](#installation)
4. [Your First Agent](#first-agent)
5. [Multi-Agent Handoffs](#handoffs)
6. [Session Memory](#sessions)
7. [Guardrails & Safety](#guardrails)
8. [Tracing & Observability](#tracing)
9. [Real-World Implementation: Customer Service](#customer-service)
10. [Production Deployment](#deployment)
11. [Migration from Swarm](#migration)
12. [Best Practices](#best-practices)

---

<a name="why-replaced"></a>
## 1. Why Agents SDK Replaced Swarm

### Swarm's Limitations (Why It Was Deprecated)

| Feature | Swarm | Agents SDK |
|---------|-------|------------|
| **Status** | ⚠️ Experimental | ✅ Production-ready |
| **Support** | None | Official OpenAI |
| **Guardrails** | Manual | Built-in |
| **Session Memory** | Manual | Automatic (SQLite/Redis) |
| **Tracing** | None | Native (Logfire, AgentOps) |
| **LLM Support** | OpenAI only | 100+ LLMs (LiteLLM) |
| **Documentation** | Minimal | Comprehensive |
| **Enterprise Features** | ❌ | Temporal integration, RBAC |

### What Changed?

**Swarm was a proof-of-concept.** It showed the pattern for multi-agent orchestration but left production concerns to developers.

**Agents SDK is production-first.** Everything you need for enterprise deployment is built-in:
- Automatic session persistence
- Input/output validation (guardrails)
- Observability and tracing
- Provider-agnostic (works with any LLM)
- Structured outputs with Pydantic
- Long-running workflows (Temporal integration)

---

<a name="primitives"></a>
## 2. The 5 Core Primitives

The Agents SDK is built on 5 simple primitives that combine to create complex behaviors:

### 1. **Agents** 🤖
An Agent is an LLM configured with:
- **Instructions**: System prompt defining behavior
- **Tools**: Functions the agent can call
- **Model**: Which LLM to use (gpt-4o, claude-3.5-sonnet, etc.)
- **Output Type**: Structured output schema (Pydantic)

```python
from agents import Agent

agent = Agent(
    name="Sales Agent",
    instructions="You help customers with product inquiries",
    model="gpt-4o",
    tools=[get_product_info, check_inventory]
)
```

### 2. **Handoffs** 🔄
Handoffs allow agents to delegate to other agents. This is THE key pattern for multi-agent systems.

```python
sales_agent = Agent(name="Sales", ...)
support_agent = Agent(name="Support", ...)

triage_agent = Agent(
    name="Triage",
    instructions="Route to sales or support based on query",
    handoffs=[sales_agent, support_agent]  # This agent can delegate
)
```

**How it works:**
1. Triage agent receives user query
2. Decides which specialist agent should handle it
3. Calls handoff tool to transfer control
4. Specialist agent takes over with full context

### 3. **Guardrails** 🛡️
Guardrails are safety checks that run before/after agent execution:

```python
from agents import Agent, InputGuardrail, OutputGuardrail

def block_pii(input_text: str) -> str:
    """Block personally identifiable information"""
    if has_pii(input_text):
        raise ValueError("PII detected")
    return input_text

def validate_output(output: str) -> str:
    """Ensure output meets quality standards"""
    if len(output) < 10:
        raise ValueError("Output too short")
    return output

agent = Agent(
    name="Agent",
    instructions="...",
    input_guardrails=[InputGuardrail(block_pii)],
    output_guardrails=[OutputGuardrail(validate_output)]
)
```

### 4. **Sessions** 💾
Sessions automatically maintain conversation history across agent runs:

```python
from agents import SQLiteSession

session = SQLiteSession("user_123")

# Turn 1
result1 = await Runner.run(agent, "What's the weather?", session=session)

# Turn 2 - agent remembers previous context
result2 = await Runner.run(agent, "What about tomorrow?", session=session)
```

**Built-in implementations:**
- `SQLiteSession`: Local file-based persistence
- `RedisSession`: Distributed session storage (coming soon)
- Custom: Implement `Session` protocol for your DB

### 5. **Tracing** 📊
Built-in observability for debugging and optimization:

```python
from agents import Agent, Runner
from agents.tracing import set_tracing_processor

# Use Logfire for tracing
set_tracing_processor("logfire")

result = await Runner.run(agent, "query")
# Automatically traces:
# - LLM calls
# - Tool executions
# - Handoffs
# - Errors
```

**Supported tracing backends:**
- Logfire (Pydantic)
- AgentOps
- Braintrust
- Scorecard
- Keywords AI

---

<a name="installation"></a>
## 3. Installation & Setup

### Prerequisites
- Python 3.10+
- OpenAI API key (or other LLM provider)

### Install Agents SDK

```bash
# Basic installation
pip install openai-agents

# With voice support (for realtime agents)
pip install 'openai-agents[voice]'

# Or using uv (recommended)
uv pip install openai-agents
```

### Set API Key

```bash
export OPENAI_API_KEY="sk-..."
```

### Verify Installation

```python
from agents import Agent, Runner

agent = Agent(
    name="Test",
    instructions="You are
