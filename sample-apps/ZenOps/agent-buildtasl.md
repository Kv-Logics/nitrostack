Yes. Below are **engineering specifications**, not product descriptions. You can paste each one into a coding agent (Cursor, Claude Code, Codex, GPT-5.5, etc.) and it should be able to implement them.

---

# Agent 1 — Planner Agent

## Purpose

The Planner Agent is the orchestrator of the multi-agent system.

It **never performs analysis itself**. Instead, it converts a natural language request into an execution plan and determines which agents and MCP tools should be invoked.

---

## Responsibilities

* Interpret user intent
* Identify missing context
* Break the request into executable tasks
* Determine execution order
* Select required MCP servers
* Dispatch tasks to downstream agents
* Aggregate intermediate outputs

---

## Inputs

```typescript
interface PlannerInput {
  userQuery: string;
  conversationHistory: ChatMessage[];
  currentIncident?: Incident;
  uiContext: UIState;
}
```

---

## Output

```typescript
interface ExecutionPlan {
  intent: string;

  tasks: Task[];

  requiredAgents: AgentName[];

  requiredTools: MCPTool[];

  expectedOutputs: string[];

  executionOrder: number[];
}
```

---

## Example

Input

```text
Compare Option A vs Option B.
```

Planner returns

```json
{
  "intent":"scenario_comparison",

  "requiredAgents":[
      "research",
      "analysis",
      "execution"
  ],

  "requiredTools":[
      "MES",
      "Maintenance",
      "Quality",
      "Simulation"
  ],

  "tasks":[
      "retrieve_batch",
      "retrieve_quality",
      "retrieve_machine_history",
      "run_simulation",
      "compare_results"
  ]
}
```

---

## LLM Prompt

```text
You are the Planner Agent.

Your only responsibility is to create an execution plan.

Do not answer the engineer.

Never invent manufacturing facts.

Determine

• user intent

• missing information

• required MCP tools

• required agents

• execution order

Return structured JSON.
```

---

## Technologies

* GPT-5.5
* LangGraph / OpenAI Agents SDK
* Pydantic
* FastAPI

---

# Agent 2 — Research Agent

## Purpose

Retrieve all evidence from enterprise systems.

No reasoning.

No recommendations.

Pure retrieval.

---

## Responsibilities

* Query MCP servers
* Normalize responses
* Merge retrieved context
* Remove duplicates
* Produce structured evidence

---

## MCP Servers

* MES
* ERP
* Maintenance
* Quality
* IoT Sensors
* Knowledge Base

---

## Input

```typescript
interface ResearchInput{

executionPlan:ExecutionPlan;

incidentId:string;

}
```

---

## Output

```typescript
interface EvidenceBundle{

batchHistory

machineHistory

sensorReadings

qualityData

maintenanceLogs

historicalIncidents

constraints

}
```

---

## Example

Planner asks

```text
Retrieve production history.
```

Research Agent

↓

MES MCP

↓

```text
Batch 204

Queue Delay

3.5 hrs

Humidity

64%

Machine

7
```

↓

Maintenance MCP

↓

```text
Machine 7

Alert

Bearing vibration
```

↓

Quality MCP

↓

```text
Yield

82%
```

↓

Returns Evidence Bundle

---

## Prompt

```text
You are the Research Agent.

Retrieve evidence only.

Never infer causes.

Never recommend actions.

Use MCP tools only.

Return structured evidence.
```

---

## Technologies

* FastAPI

* MCP SDK

* PostgreSQL

* ChromaDB

---

# Agent 3 — Analysis Agent

## Purpose

This is the reasoning engine.

It transforms evidence into decisions.

---

## Responsibilities

* Root Cause Analysis
* Correlation
* Counterfactual reasoning
* Simulation
* Recommendation ranking
* Business impact

---

## Input

```typescript
interface AnalysisInput{

EvidenceBundle evidence;

ExecutionPlan plan;

}
```

---

## Output

```typescript
interface AnalysisResult{

rootCauses

simulationResults

recommendations

confidenceScores

businessImpact

supportingEvidence

}
```

---

## Internal Modules

### Root Cause Module

Outputs

```text
Humidity

94%

Queue Delay

91%

Machine 7

42%
```

---

### Simulation Module

Runs

```text
Scenario A

↓

Scenario B

↓

Scenario C
```

---

### Recommendation Module

Ranks

```text
Reduce Queue Delay

96%

Install Humidity Control

94%

Replace Machine

61%
```

---

## Prompt

```text
You are the Analysis Agent.

You receive enterprise evidence.

Determine

root causes

counterfactuals

recommended actions

business impact

Always explain confidence.

Never fabricate evidence.

Every conclusion must reference supporting evidence.
```

---

## Technologies

* GPT-5.5

* Python

* NetworkX

* Pandas

* NumPy

---

# Agent 4 — Execution Agent

## Purpose

Present results and control the workbench.

This agent never performs reasoning.

---

## Responsibilities

* Generate reports
* Highlight UI
* Open Timeline
* Open Replay
* Focus Graph
* Notify Manager
* Export PDF
* Produce chat response

---

## Input

```typescript
interface ExecutionInput{

AnalysisResult analysis;

UIState currentUI;

}
```

---

## Output

```typescript
interface ExecutionOutput{

assistantMessage

uiActions[]

generatedReports[]

notifications[]

}
```

---

## Example

Engineer

```text
Show evidence.
```

Execution Agent returns

```json
{

"assistantMessage":"Queue delay has the strongest causal influence.",

"uiActions":[

"OPEN_TIMELINE",

"HIGHLIGHT_QUEUE_DELAY",

"OPEN_GRAPH",

"OPEN_SIMULATION"

]

}
```

Frontend simply executes

```typescript
dispatch(uiActions)
```

---

## Report Generator

Outputs

```text
Incident Summary

↓

Root Cause

↓

Evidence

↓

Recommendation

↓

Business Impact

↓

Risk

↓

Executive Summary
```

---

## Prompt

```text
You are the Execution Agent.

Present results.

Never perform analysis.

Use the supplied AnalysisResult.

Generate concise explanations.

Produce UI actions.

Generate reports.

Never invent evidence.
```

---

## Technologies

* FastAPI

* ReportLab

* React Action Dispatcher

---

# Complete Agent Pipeline

```text
                    Engineer
                        │
                        ▼
                Planner Agent
            (Intent + Task Planning)
                        │
                        ▼
               Research Agent
        (MCP Data Collection Layer)
                        │
                        ▼
               Analysis Agent
 (Reasoning, Root Cause, Simulation, Ranking)
                        │
                        ▼
               Execution Agent
 (UI Actions, Reports, Explanations)
                        │
                        ▼
          ForgeOps Decision Workbench
```

# Recommended project structure

```text
backend/
│
├── agents/
│   ├── planner/
│   │   ├── planner.py
│   │   ├── prompt.md
│   │   ├── models.py
│   │   └── service.py
│   │
│   ├── research/
│   │   ├── research.py
│   │   ├── mcp_client.py
│   │   ├── retriever.py
│   │   └── models.py
│   │
│   ├── analysis/
│   │   ├── analysis.py
│   │   ├── root_cause.py
│   │   ├── simulator.py
│   │   ├── recommender.py
│   │   └── models.py
│   │
│   └── execution/
│       ├── execution.py
│       ├── report_generator.py
│       ├── ui_actions.py
│       └── models.py
│
├── mcp/
│   ├── mes_server.py
│   ├── quality_server.py
│   ├── maintenance_server.py
│   ├── erp_server.py
│   ├── sensors_server.py
│   └── simulation_server.py
│
├── api/
├── database/
├── schemas/
└── main.py
```

This separation is ideal for a hackathon because each teammate can own one agent independently, while the interfaces between agents (`ExecutionPlan`, `EvidenceBundle`, `AnalysisResult`, and `ExecutionOutput`) remain clean and well-defined. It also makes the MCP orchestration easy to demonstrate to judges, since each agent has a single, clearly explainable responsibility.