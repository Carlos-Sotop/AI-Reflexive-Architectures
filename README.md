
# Reflexive AI: A Framework for Layered Intelligence

This framework explores how to build AI systems that improve themselves through layered memory, reflection, and selective forgetting. Inspired by cognitive neuroscience and informed by practical constraints, it defines a structure where agents evaluate not just actions, but the quality of their own introspection.

---

## Architecture Overview

The system is structured in layers, each with a distinct cognitive role and memory type.

                                                    ┌─────────────────────────────┐
                                                    │      Shared Memory Layer    │
                                                    │         (long-term)         │
                                                    │                             │
                                                    │    • Strategic Knowledge    │
                                                    │    • Global Context         │
                                                    │    • Prompt/Test History    │
                                                    │    • Reflection Logs        │
                                                    └─────────────────────────────┘
        ┌────────────────────────────┐                    │
        │           Level 2:         │                    │
        │         Meta-Watcher       │◄───────────────────┘
        │                            │                    │
        │  (Reflects on reflection)  │                    │
        │ • Heuristic Memory         │                    │
        └────────────┬───────────────┘                    │
                     ↓                                    │
        (Reflects on how reflection is done;              │
         edits/refines Level 1's prompts/logic)           │
                                                          │
        ┌───────────────────────────┐                     │
        │           Level 1:        │◄────────────────────┘                   
        │      Reflexive Watcher    │───┐
        │                           │   │
        │ (Monitors + tunes agents) │   │
        │ • Reflection Logs         │   │
        └────────────┬──────────────┘   │
                     ↓                  │
        (Evaluates agents; edits        │
         prompts, tunes temp, A/B)      │
                                        ↓
                        ┌───────────────────────────┐  
                        │          Level 0:         │   
                    ┌── │        Orchestrator       │──┐
                    │   │                           │  │
                    │   │  (Interprets & delegates) │  │
                    │   │  • Working Mem            │  │
                    ↓   └───────────────────────────┘  ↓
            ┌────────────────────┐   ┌────────────────────────┐
            │     Strategist     │   │       Tactician        │
            │                    │   │                        │
            │ (Plans & reasons)  │   │ (Executes instructions)│
            │ • Declarative Mem  │   │ • Procedural Mem       │
            └────────────────────┘   └────────────────────────┘
            



## Layer Descriptions

### Level 0 – Execution Agents

| Agent        | Function                                                  | Memory Type                 | Temp |
|--------------|-----------------------------------------------------------|-----------------------------|-------|
| Orchestrator | Interprets user intent and delegates control              | Working + partial long-term | 0.3 |
| Strategist   | Generates context-aware insights and foresight            | Long-term declarative       | 0.7 |
| Tactician    | Performs precise, low-level actions                       | Procedural                  | 0.2 |

Each agent has its own local memory and receives prompt/parameter tuning from above.

---

### Level 1 – Reflexive Watcher

Monitors agent outputs in real time:

- Detects contradictions between agent outputs.
- Flags hallucinations, redundancy, or degraded performance.
- **Edits system prompts** for Level 0 agents.
- **A/B tests** new prompts, temperature ranges, or routing strategies.
- Tracks performance impacts of changes and rolls them back when needed.

Handles short-term forgetting:

```ts
// Short-term forgetting example
function forgetRecentNoise(memoryLog) {
  return memoryLog.filter(entry =>
    entry.importance > 0.5 &&
    entry.age < 7 &&
    !entry.redundant
  );
}

// A/B testing agent prompts
function testAgentPrompts(agentId, promptVariants) {
  return promptVariants.map((prompt, i) => ({
    agentId,
    variant: `v${i + 1}`,
    prompt,
    temperature: 0.4 + i * 0.1,
  }));
}
```

---

### Level 2 – Meta-Watcher

Reflects on the reflection process itself:

- Evaluates the Watcher's own logic and behavior over time.
- Modifies Watcher prompts, logic, or heuristics.
- A/B tests reflection strategies, such as shallow vs. deep analysis or different intervals.
- Handles strategic forgetting of outdated self-evaluation techniques.
- Runs on long intervals (e.g. milestone-based or weekly):

```ts

// Meta-watcher A/B testing reflection logic
const reflectionVariants = [
  { id: 'interval-3d', config: { intervalDays: 3, questionDepth: 1 } },
  { id: 'interval-7d-deep', config: { intervalDays: 7, questionDepth: 2 } }
];

// Strategic forgetting
function pruneReflectionHeuristics(history) {
  return history.filter(h =>
    h.lastUsedRecently && h.performanceImpact > 0.7
  );
}

```

---

## Agent Temperature Strategy

| Layer         | Temperature | Purpose                                 |
|---------------|-------------|-----------------------------------------|
| Meta-Watcher  | 0.5         | Balanced evaluation and strategy tuning |
| Watcher       | 0.3         | Low-variance observation + contradiction tracing |
| Orchestrator  | 0.3         | Reliable routing, avoids hallucination  |
| Strategist    | 0.7         | Abstraction, hypothesis generation      |
| Tactician     | 0.2         | Deterministic execution, no drift       |






Example Agent Call

```ts
const settings = {
  tactician: { temp: 0.2, prompt: "You strictly execute structured actions." },
  strategist: { temp: 0.7, prompt: "You generate high-level strategic insights..." },
};

const res = await openai.chat.completions.create({
  model: 'gpt-4o',
  temperature: settings[agentType].temp,
  messages: [
    { role: "system", content: settings[agentType].prompt },
    { role: "user", content: `${context}\n\n${command}` }
  ],
  response_format: { type: "json_object" }
});

```

---

## Memory Stack

Each layer requires scoped memory access:

| Memory Type         | Holder              | Accessed By             | Term     | Purpose                                                              |
|---------------------|---------------------|--------------------------|----------|----------------------------------------------------------------------|
| Shared Memory        | System (Long-Term)  | **Watcher, Meta-Watcher**| Long     | Stores finalized outcomes, global state, audit logs, strategic truth  |
| Heuristic Memory     | Meta-Watcher        | Meta-Watcher             | Long     | Stores rules for evaluating reflection, scoring functions             |
| Reflection Logs      | Watcher             | Watcher, Meta-Watcher    | Long     | Captures outputs, contradictions, and agent evaluations               |
| Working Memory       | Orchestrator        | Orchestrator             | Short    | Tracks current input state, pending intent, and flow coordination     |
| Declarative Memory   | Strategist          | Strategist               | Long     | Strategy logic, learned insights, reusable mental models              |
| Procedural Memory    | Tactician (optional)| Tactician                | Medium   | Reusable execution patterns and routines                             |





---

## When to Reflect

| Trigger Type           | Layer         | Example                                                                 |
|------------------------|---------------|-------------------------------------------------------------------------|
| Contradiction          | Watcher       | Strategist proposes direction; Orchestrator executes something misaligned |
| Incoherence            | Watcher       | Outputs across agents conflict or diverge in tone, goal, or logic      |
| Execution Drift        | Watcher       | Orchestrator decisions gradually deviate from strategic intent          |
| Milestone Reached      | Meta-Watcher  | A process, session, or plan completes                                  |
| Performance Drop       | Meta-Watcher  | Observed quality or effectiveness decreases over time                  |
| Stagnation             | Meta-Watcher  | No variation or adaptation across recent outputs                       |
| Oversensitivity        | Meta-Watcher  | Watcher triggers frequently with low-value corrections                 |
| Under-reflection       | Meta-Watcher  | Watcher hasn’t triggered in too long despite signs of drift            |

---

## Design Principles

### Layered Reflexivity  
Each layer reflects on the one below it, with bounded authority. Reflection is structured, not recursive.

### Bounded Feedback Loops  
Avoid infinite regress by triggering reflection at meaningful intervals or events, not continuously.

### Prompt Adaptation  
Prompts, temperature, and system behavior are dynamic, modified based on reflection outcomes and empirical feedback.

### Selective Forgetting  
Each layer prunes memory based on relevance, impact, and recency. Forgetting is as important as remembering.

### Neuro-Inspired Mapping  
- **Procedural →** (Tactician role, inspired by Cerebellum)  
- **Declarative →** (Strategist role, inspired by Hippocampus)  
- **Metacognition →** (Watcher & Meta-Watcher roles, inspired by Prefrontal Cortex)


---

## Closing Note

This structure makes it possible for AI systems not just to reason, but to evaluate their own reasoning process, and update the tools with which they reflect. Every layer is editable. Every insight is testable based on user interactions.

In this framework, reflexivity isn’t recursion, it’s distributed governance. 

Intelligence emerges not from depth alone, but from how governance and responsibility is layered across different agents.

