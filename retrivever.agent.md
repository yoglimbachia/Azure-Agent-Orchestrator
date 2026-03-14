# Agent: Retriever

## Identity

| Field | Value |
|---|---|
| Name | `retriever` |
| Type | Sub-Agent |
| Runtime | Azure Durable Functions (activity) |
| Backend | Azure AI Search (vector + hybrid retrieval) |

## Purpose

Performs Retrieval-Augmented Generation (RAG) by searching enterprise knowledge stores via Azure AI Search and enriching the orchestration context with relevant documents, citations, and data.

## Responsibilities

1. **Query Formulation** — Transform the plan step into an optimized search query (keyword, vector, or hybrid).
2. **Index Search** — Execute the query against one or more Azure AI Search indices.
3. **ACL Filtering** — Apply security filters so users only see data they are authorized to access (FR-4).
4. **Result Enrichment** — Return ranked results with citations (document ID, title, relevance score).
5. **Context Assembly** — Package retrieved content into a context window for downstream agents.

## Skills Used

| Skill | Spec |
|---|---|
| RAG Retrieval | [rag-retrieval.skill.md](../skills/rag-retrieval.skill.md) |

## Inputs

| Input | Source | Description |
|---|---|---|
| Search query / plan step | Orchestrator | What to retrieve |
| User claims | Orchestrator (from Entra token) | For ACL-based filtering |
| Index name(s) | Configuration | Target search indices |

## Outputs

| Output | Description |
|---|---|
| Results array | Ranked documents with `id`, `title`, `snippet`, `score`, `source` |
| Citations | Links/IDs to original AI Search items |
| Context payload | Assembled text block for LLM context window |

## Data Sources

| Source | Azure Service | Index Type |
|---|---|---|
| Enterprise documents | Azure Storage (Blob) | Vector + keyword |
| Agent memory / logs | Azure Cosmos DB | Keyword |
| Knowledge articles | External / LOB systems (via APIM) | Hybrid |

## Search Configuration

| Parameter | Value |
|---|---|
| Search mode | Hybrid (vector + BM25 keyword) |
| Vector dimensions | Per embedding model (e.g., 1536 for ada-002) |
| Top-K results | 5 (configurable) |
| ACL field | `allowed_groups` (filtered by user's Entra group claims) |
| Semantic ranker | Enabled |

## Guardrails Applied

- ACL filtering ensures RBAC-compliant results (FR-4, FR-11).
- No raw document content returned without citation metadata.
- PII in results is flagged for optional redaction by Policy Agent.

## Relevant Requirements

- FR-4 (Enterprise Memory / RAG)
- NFR-2 (Performance — search latency), NFR-3 (Scalability — replica tuning)
