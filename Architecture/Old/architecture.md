# Re-sourcer Solution Architecture

## 1. Problem Space & Vision
Re-sourcer accelerates and governs resource allocation in a consultancy environment by transforming unstructured Statements of Work (SOW), Work Breakdown Structures (WBS), and skill inventories into actionable, explainable recommendations. The system reduces time-to-fill, improves utilisation, and embeds Responsible AI & compliance.

## 2. Core Domains
- Ingestion & Normalization: Compass One (SOW, WBS), ESXP (forecast, utilisation), Engage360 (skills), Excel WBS uploads, partner data.
- Retrieval & Enrichment: Hybrid RAG (vector + keyword) over clause chunks, skill graph expansion, historical placements and performance signals.
- Matching & Ranking: Availability, skill similarity, proficiency weighting, historical success, diversity and policy constraints.
- Workflow & Transactions: Generate recommendations, create resource requests, perform bookings, update allocation state.
- Governance & Responsible AI: Bias mitigation, privacy, audit, explainability, policy/rate enforcement.
- Observability & Feedback: Telemetry, KPIs, acceptance signals, drift/fairness monitoring.

## 3. Key Quantitative KPIs
- Time-to-fill (request to booking) – target < 30 min (chat latency P95 < 4s)
- Suggestion acceptance rate – target > 60%
- Utilisation uplift – +5–10% billable hours
- Forecast accuracy delta – reduce variance by >15%
- Diversity balance vs target – maintain within ±5% threshold
- Manual intervention / override rate – < 20%
- System reliability – > 99.5% successful tool calls

## 4. Architecture Option 1: Copilot Studio + MCP Gateway (Aggregator)
Best for fast adoption & centralized governance.
```mermaid
flowchart LR
    U["Teams / M365 Chat<br>RM / PM / Exec"] --> CS["Copilot Studio<br>(Intent + Prompt Orchestrator)"]
    CS -->|Grounding Query| RAG["Azure AI Search<br>Compass One Index (Vector+Keyword)"]
    CS -->|Tool Calls (MCP)| GW["MCP Gateway<br>(Unified Server)"]

    subgraph Integration["Gateway Internal Adapters"]
        GW --> C1["Compass One Adapter"]
        GW --> E1["ESXP Adapter"]
        GW --> G1["Engage360 Adapter"]
        GW --> X1["Excel/WBS Upload Handler"]
    end

    C1 --> CO["Compass One APIs / Docs"]
    E1 --> ES["ESXP APIs / Forecast DB"]
    G1 --> EN["Engage360 Skills API"]
    X1 -->|Parse + Extract| IE["Ingestion Engine"]

    IE --> STG["Raw & Parsed Store (Blob + Queue)"]
    STG --> PIPE["Feature Engineering / Skill Normalization"]

    PIPE --> MATCH["Matching & Ranking Engine"]
    GW --> MATCH
    MATCH --> POL["Policy & Rate Engine"]
    POL --> RES["Recommendation Assembly"]

    RES --> AUD["Audit & Explainability Store"]
    GW --> AUD
    AUD --> OBS["App Insights / Log Analytics / Metrics"]

    RES --> CS
    CS --> U

    subgraph Security["Identity & Governance"]
       SEC["Azure Entra ID / RBAC / Conditional Access"] --> GW
       SEC --> CS
       SEC --> POL
    end

    subgraph Data["Responsible AI / Monitoring"]
       FAIR["Bias Checks"] --> POL
       PRIV["PII Scrub / Mask"] --> IE
    end

    classDef primary fill:#2563eb,color:#fff;
    classDef data fill:#4b5563,color:#fff;
    classDef logic fill:#10b981,color:#fff;
    classDef gov fill:#7c3aed,color:#fff;
    classDef out fill:#f59e0b,color:#fff;

    class CS,GW primary
    class RAG,STG,PIPE data
    class MATCH,POL,RES logic
    class AUD,OBS,FAIR,PRIV gov
    class U out
```
Strengths: Single endpoint, cross-system joins, unified audit. Trade-offs: Potential gateway bottleneck – scale horizontally.

## 5. Architecture Option 2: Direct MCP Servers + Multi-Agent Orchestration
Modular; each system exposed separately.
```mermaid
flowchart TB
    U["Teams / Chat"] --> CS["Copilot Studio"]
    CS -->|Tool call| MGR["Orchestration Agent (Conversation)"]

    subgraph MCP["MCP Servers"]
        MGR --> MCP1["MCP Compass One"]
        MGR --> MCP2["MCP ESXP"]
        MGR --> MCP3["MCP Engage360"]
        MGR --> MCP4["MCP WBS/Excel"]
    end

    MGR --> ENRICH["Enrichment Agent<br>(Skill Graph + History)"]
    ENRICH --> MATCH["Matching Agent"]
    MATCH --> OPT["Optimization Agent<br>(Capacity / Scenario)"]
    OPT --> POLICY["Policy Agent"]
    POLICY --> EXPLAIN["Explainability Agent"]
    EXPLAIN --> MGR

    MGR --> CS
    CS --> U

    subgraph Stores
        HIST["Historical Placement DB"]
        SKILL["Skill Graph / Vector Store"]
        AUD["Audit & Metrics"]
        HIST --> ENRICH
        SKILL --> ENRICH
        POLICY --> AUD
        MATCH --> AUD
    end

    subgraph RAI["Responsible AI"]
        FAIR["Bias Evaluator"]
        PRIV["Data Minimizer"]
        FAIR --> POLICY
        PRIV --> MCP2
    end

    classDef agent fill:#0e7490,color:#fff;
    classDef mcp fill:#2563eb,color:#fff;
    classDef store fill:#4b5563,color:#fff;
    classDef rai fill:#7c3aed,color:#fff;
    classDef user fill:#f59e0b,color:#fff;

    class MGR,ENRICH,MATCH,OPT,POLICY,EXPLAIN agent
    class MCP1,MCP2,MCP3,MCP4 mcp
    class HIST,SKILL,AUD store
    class FAIR,PRIV rai
    class U user
```
Strengths: High extensibility. Trade-offs: More hops, latency tuning required.

## 6. Architecture Option 3: Event-Driven Microservices + Agent Mesh
For high throughput & batch operations.
```mermaid
flowchart LR
    subgraph Frontend
        U["Teams / Portal / API"] --> CHAT["Conversation Service"]
        CHAT --> AUTH["Auth / RBAC"]
    end

    CHAT --> CMDQ["Command Bus (Service Bus / Kafka)"]

    subgraph Services
        ING["Ingestion Service"] --> EVT["Event: SOW.Parsed"]
        SKL["Skill Extraction Service"] --> EVT2["Event: SkillGraph.Updated"]
        MAT["Matching Service"] --> EVT3["Event: Match.Completed"]
        POLS["Policy Service"] --> MAT
        SIM["Simulation Service"] --> MAT
        BOOK["Booking Service"] --> EVT4["Event: Allocation.Booked"]
        EXP["Explainability Service"] --> MAT
        FEED["Feedback Collector"] --> MAT
    end

    ING --> INDEX["AI Search Index"]
    SKL --> GRAPH["Skill Graph DB"]
    MAT --> CACHE["Recommendation Cache (Redis)"]
    BOOK --> SYS1["ESXP"]
    ING --> SYS2["Compass One"]
    SKL --> SYS3["Engage360"]

    OBS["Telemetry / Metrics"] --> DASH["Adoption Dashboard"]

    subgraph ResponsibleAI
        BIAS["Bias Monitor"] --> POLS
        DRIFT["Model Drift Monitor"] --> MAT
        GOVERN["Policy Config Repo"]
        GOVERN --> POLS
    end

    CMDQ --> MAT
    CMDQ --> ING
    CMDQ --> BOOK

    classDef svc fill:#2563eb,color:#fff;
    classDef store fill:#4b5563,color:#fff;
    classDef evt fill:#10b981,color:#fff;
    classDef rai fill:#7c3aed,color:#fff;
    classDef out fill:#f59e0b,color:#fff;

    class ING,SKL,MAT,POLS,SIM,BOOK,EXP,FEED svc
    class INDEX,GRAPH,CACHE store
    class EVT,EVT2,EVT3,EVT4 evt
    class BIAS,DRIFT,GOVERN rai
    class U,CHAT out
```
Strengths: Resilient, scalable. Trade-offs: Operational complexity.

## 7. Architecture Option 4: Hybrid Retrieval + Optimization (LLM + Solver)
Adds global optimisation for allocation.
```mermaid
flowchart TB
    USER["RM / PM Query"] --> CONVO["Conversation Layer (Copilot / Chat API)"]
    CONVO --> RETRIEVE["Retrieval Layer (Hybrid: Vector + Structured)"]

    RETRIEVE --> CTX["Context Assembly"]
    CTX --> LLM["LLM Reasoning (Prompt + Tool Use)"]
    LLM --> MATCHFN["Matching Function (Custom)"]
    MATCHFN --> SOLVER["Optimization Solver (OR-Tools)"]

    SOLVER --> RANK["Ranked Allocation Sets"]
    RANK --> EXPLAIN["Explanation Generator"]
    EXPLAIN --> CONVO
    RANK --> BOOKAPI["Booking API"]

    BOOKAPI --> ESXP
    RETRIEVE --> COMPASS["Compass One"]
    RETRIEVE --> ENG360["Engage360"]
    RETRIEVE --> HIST["Historical DB"]
    RETRIEVE --> SKILLG["Skill Graph"]

    subgraph Guardrails
       POLICY["Policy Engine"]
       FAIR["Bias / Fairness"]
       PRIV["PII Filter"]
       POLICY --> MATCHFN
       FAIR --> SOLVER
       PRIV --> RETRIEVE
    end

    subgraph Observability
       TELE["Telemetry"]
       AUD["Audit Trail"]
       TELE --> DASH["KPI Dashboard"]
       LLM --> AUD
       SOLVER --> AUD
    end

    classDef main fill:#2563eb,color:#fff;
    classDef logic fill:#10b981,color:#fff;
    classDef data fill:#4b5563,color:#fff;
    classDef gov fill:#7c3aed,color:#fff;
    classDef out fill:#f59e0b,color:#fff;

    class CONVO,LLM main
    class MATCHFN,SOLVER,RANK,EXPLAIN,BOOKAPI logic
    class RETRIEVE,CTX,HIST,SKILLG data
    class POLICY,FAIR,PRIV gov
    class USER out
```
Strengths: Optimal solutions; explainable. Trade-offs: Constraint complexity.

## 8. Architecture Option 5: MVP (Lean Start)
```mermaid
flowchart LR
    U["RM in Teams"] --> BASIC["Copilot Studio (Single Intent)"]
    BASIC --> IDX["Azure AI Search (SOW Text + Roles)"]
    BASIC --> SCRIPT["Matching Script (Azure Function)"]
    SCRIPT --> ESXP
    SCRIPT --> ENG360
    SCRIPT --> SUG["Top 5 Candidates"]
    SUG --> BASIC
```

## 9. Core Data Model (Conceptual)
- Resource: { ResourceID, Name, Skills[], Proficiency, RateCard, Geo, AvailabilityCalendar, HistoricalProjects[] }
- RoleRequirement: { RoleID, ProjectID, SkillWeights[], StartDate, EndDate, EffortHours, Seniority, Constraints }
- SOWClauseChunk: { ChunkID, ProjectID, Text, Vector, Type, EffectiveDate }
- Recommendation: { ReqID, Candidates[{ResourceID, ScoreBreakdown, AvailabilityWindow, PolicyFlags[], Explanation}] }
- Booking: { BookingID, ResourceID, RoleID, ProjectID, Status, CreatedAt, ApprovedBy }
- PolicyRule: { RuleID, Category, Expression, Version, Active }
- AuditEvent: { EventID, Type, Actor, InputsHash, OutputSummary, Timestamp, LatencyMs }

## 10. Multi-Agent Role Definitions
- Ingestion Agent: Transforms SOW/WBS into structured RoleRequirements + clause vectors.
- Skill Graph Agent: Maintains related skill edges, enabling fuzzy expansion.
- Matching Agent: Combines embeddings + structured filters to create candidate pools.
- Optimization Agent: Solves allocation (maximize coverage, minimize cost/functions).
- Policy Agent: Applies constraints (rate, geo, double-booking, compliance).
- Explainability Agent: Generates human-readable rationale with score breakdown.
- Forecast Simulation Agent: Projects utilisation impact if booked.
- Feedback Agent: Captures accept/reject signals to improve models.

## 11. Responsible AI Integration
- Bias tracking across geo, seniority, diversity attributes.
- Score breakdown transparency (SkillMatch%, Availability%, History%, PolicyAdjustments%).
- Privacy: Attribute minimization & masking for non-essential flows.
- Override & Escalation: Manual adjustments logged to audit.
- Lifecycle: Fairness & drift review cadence (monthly/quarterly).

## 12. Key Sequence: Find Resources
```mermaid
sequenceDiagram
autonumber
participant RM as Resource Manager
participant CS as Copilot Studio
participant GW as MCP Gateway / Orchestrator
participant RAG as Azure AI Search
participant CO as Compass One
participant ES as ESXP
participant ENG as Engage360
participant MATCH as Matching Engine
participant POL as Policy Engine
participant AUD as Audit Store

RM->>CS: "Find resources for Project Atlas"
CS->>RAG: Query project context (Atlas)
RAG-->>CS: Roles, clauses, constraints
CS->>GW: Tool call: search_resources(project=Atlas)
GW->>CO: Fetch role metadata & budgets
GW->>ES: Fetch availability & utilization
GW->>ENG: Fetch skill profiles
CO-->>GW: Role + SOW constraints
ES-->>GW: Availability calendar
ENG-->>GW: Skills list
GW->>MATCH: Assemble candidate pool
MATCH->>POL: Apply rate, geo, compliance rules
POL-->>MATCH: Adjusted scores + flags
MATCH-->>GW: Ranked list + explanations
GW->>AUD: Log request, inputs, outputs
GW-->>CS: Candidate set
CS-->>RM: Display recommendations + actions
```

## 13. Key Sequence: Upload Excel WBS
```mermaid
sequenceDiagram
autonumber
participant Arch as Architect
participant Portal as Upload Portal / Teams Plugin
participant ING as Ingestion Service
participant PARSE as Parser (Excel -> JSON)
participant FEAT as Feature Engineering
participant IDX as Azure AI Search
participant DB as RoleRequirements DB

Arch->>Portal: Upload WBS.xlsx
Portal->>ING: File metadata + blob URL
ING->>PARSE: Parse Excel sheets
PARSE-->>ING: Structured roles {RoleID, Skills, Hours}
ING->>FEAT: Normalize skills, map synonyms
FEAT-->>ING: Standardized roles + skill vectors
ING->>DB: Persist RoleRequirements
ING->>IDX: Index role + contextual clauses
IDX-->>ING: Index confirmation
ING-->>Portal: "WBS ingestion complete"
```

## 14. Deployment View (Option 1 Example)
```mermaid
graph TD
    subgraph Azure Subscription
        LB["Azure Front Door / AppGW"] --> APIM["API Management (Optional)"]
        APIM --> CSAPP["Copilot Studio Runtime"]
        APIM --> GWAPP["MCP Gateway (AKS / Container App)"]
        GWAPP --> FUNC["Matching & Policy Functions (Azure Functions)"]
        GWAPP --> CACHE["Redis Cache"]
        GWAPP --> BUS["Service Bus (Async)"]

        FUNC --> SEARCH["Azure AI Search Index"]
        FUNC --> VSTORE["Vector Embeddings Store"]
        FUNC --> GRAPHDB["Skill Graph DB"]

        GWAPP --> STG["Blob Storage (Docs/WBS)"]
        STG --> PIPE["Data Factory / Synapse Pipelines"]

        GWAPP --> DB["Cosmos DB (Core Models)"]
        GWAPP --> AUD["Audit Log (Cosmos / Table)"]

        MON["App Insights / Log Analytics"] --> DASH["Power BI Dashboard"]
        SEC["Azure Entra ID / Managed Identities"] --> CSAPP
        SEC --> GWAPP
    end
```

## 15. MCP Tool Schema (Illustrative)
```json
{
  "tools": [
    {
      "name": "search_resources",
      "inputSchema": {
        "type": "object",
        "properties": {
          "projectId": {"type": "string"},
          "roleId": {"type": "string"},
          "startDate": {"type": "string", "format": "date"},
          "endDate": {"type": "string", "format": "date"},
          "maxCandidates": {"type": "integer", "default": 10}
        },
        "required": ["projectId", "roleId", "startDate", "endDate"]
      },
      "outputSchema": {
        "type": "object",
        "properties": {
          "candidates": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "resourceId": {"type": "string"},
                "score": {"type": "number"},
                "skillMatch": {"type": "number"},
                "availabilityWindow": {"type": "string"},
                "rate": {"type": "number"},
                "policyFlags": {"type": "array", "items": {"type": "string"}},
                "explanation": {"type": "string"}
              }
            }
          }
        }
      }
    }
  ]
}
```

## 16. Recommended Roadmap
1. MVP: Ingestion (Compass One + ESXP), heuristic matching, manual booking.
2. Gateway: MCP + vector index, audit, basic policy engine.
3. Expansion: Skill graph, historical weighting, fairness metrics.
4. Optimization: Add solver + simulation agent.
5. Ecosystem: Partner integration adapters, demand forecasting.
6. Learning Loop: Feedback-driven model tuning, drift & fairness automation.

## 17. Risks & Mitigations
| Risk | Mitigation |
|------|------------|
| Data freshness lag | Delta event ingestion, staleness alarms (>2h) |
| Policy drift | Versioned rules, feature flags, rollback plan |
| Latency spikes | Precompute availability snapshots, Redis cache, async pipelines |
| Bias emergence | Weekly fairness reports, threshold adjustments |
| Excel variability | Template validator + fallback LLM parsing |
| Gateway bottleneck | Horizontal scale, circuit breakers, bulk endpoints |

## 18. Technology Stack Summary
- LLM: Azure OpenAI (GPT-4.1 / o3-mini) for reasoning vs cost trade-offs.
- Embeddings: text-embedding-3-large for clauses & skills.
- Matching: Hybrid filters + cosine similarity; candidate pre-filter in SQL/Redis.
- Optimization: OR-Tools (Python) behind MCP tool `optimize_allocation`.
- Storage: Cosmos DB (core), Blob (docs), Redis (hot cache), AI Search (retrieval).
- Graph: Neo4j or Cosmos Gremlin for skill adjacency.
- Telemetry: App Insights, custom KPIs exported to Log Analytics + Power BI.

## 19. Selection Guidance
- Need speed/governance → Option 1
- Need modular experimentation → Option 2
- Need batch & resiliency → Option 3
- Need global optimal allocations → Option 4
- Need quick stakeholder demo → Option 5

## 20. Next Enhancements (Future Appendix)
- Policy DSL (CEL) examples
- Fairness score formula sample
- Cosmos DB container schemas
- Partner adapter contract spec

---
End of Architecture Document
