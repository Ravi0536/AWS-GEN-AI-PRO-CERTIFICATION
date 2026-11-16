# AIP-C01 — Exam Strategy and Question Patterns

AWS Certified Generative AI Developer – Professional. This is the master guide: how the exam is built, how to decode a question, and the cross-domain rules that settle most answers. Read this first, then the five domain documents.

> Exam facts and every task/skill statement here were checked against the official AWS exam guide and certification page on 2026-11-14. The practice questions in these documents are original and written to match the exam's style; they are not real exam items.

---

<!-- contents:start -->
## Contents

- [1. Exam at a glance](#1-exam-at-a-glance)
- [2. How the exam writes questions](#2-how-the-exam-writes-questions)
  - [The constraint qualifier decides the answer](#the-constraint-qualifier-decides-the-answer)
  - [Five distractor families](#five-distractor-families)
  - [Multiple-response questions](#multiple-response-questions)
- [3. The five-pass method for any question](#3-the-five-pass-method-for-any-question)
- [4. Problem-layer diagnosis (the most reused skill on the exam)](#4-problem-layer-diagnosis-the-most-reused-skill-on-the-exam)
- [5. Service-selection anchors](#5-service-selection-anchors)
- [6. Core decision trees](#6-core-decision-trees)
  - [Customization ladder (cheapest and fastest first)](#customization-ladder-cheapest-and-fastest-first)
  - [Capacity and pricing mode](#capacity-and-pricing-mode)
  - [Where does the agent run?](#where-does-the-agent-run)
- [7. Study sequence](#7-study-sequence)
- [8. Architecture diagrams](#8-architecture-diagrams)
- [Sources](#sources)

All documents and the A to Z topic index: [INDEX.md](INDEX.md)

---
<!-- contents:end -->

## 1. Exam at a glance

| Item | Value |
|---|---|
| Questions | 75 (65 scored + 10 unscored, not identified) |
| Time | 180 minutes (2.4 min per question) |
| Passing score | 750 on a 100–1,000 scale, compensatory (no per-domain minimum) |
| Question types | Multiple choice (1 of 4), multiple response (2+ of 5 or more, no partial credit) |
| Cost | 300 USD |
| Guessing | No penalty. Never leave a blank |

| Domain | Weight | Scored questions (approx.) | Document |
|---|---|---|---|
| 1. FM Integration, Data Management, and Compliance | 31% | 20 | [Domain 1](01-Domain1-FM-Integration-Data-Compliance.md) |
| 2. Implementation and Integration | 26% | 17 | [Domain 2](02-Domain2-Implementation-Integration.md) |
| 3. AI Safety, Security, and Governance | 20% | 13 | [Domain 3](03-Domain3-Safety-Security-Governance.md) |
| 4. Operational Efficiency and Optimization | 12% | 8 | [Domain 4](04-Domain4-Operational-Efficiency-Optimization.md) |
| 5. Testing, Validation, and Troubleshooting | 11% | 7 | [Domain 5](05-Domain5-Testing-Validation-Troubleshooting.md) |

Domains 1 and 2 are 57% of the score. RAG (Tasks 1.4, 1.5) and agents (Task 2.1) are the two largest single topics. Because scoring is compensatory, depth in Domains 1–3 can carry a weaker Domain 4 or 5, but not the reverse.

**Explicitly out of scope:** model training from scratch, advanced ML techniques, feature engineering. If an answer requires building or training a model when a managed FM capability exists, it is almost always a distractor.

---

## 2. How the exam writes questions

Every question has the same four parts. Train yourself to find them in this order:

1. **Scenario** — company, workload, what exists today. Mostly noise, but it hides constraints (Region, data type, existing stack).
2. **Symptom or goal** — what is wrong or what must be built.
3. **Constraint qualifier** — the one phrase that picks the winner. Read this before the options.
4. **Options** — usually two are plausible, one is over-engineered, one is technically wrong.

### The constraint qualifier decides the answer

| Qualifier in the stem | What wins | What loses |
|---|---|---|
| "LEAST operational overhead" / "minimal management" | Fully managed Bedrock feature (Knowledge Bases, Guardrails, Agents, Flows, Prompt Management, Evaluations) | Self-hosted on EC2/EKS, custom Lambda pipelines, SageMaker endpoints |
| "MOST cost-effective" | Smaller model, batch inference, Flex tier, prompt caching, model cascade, distillation | Provisioned Throughput for spiky load, largest model everywhere |
| "LOWEST latency" / "real time" | Streaming, latency-optimized inference, Priority tier, smaller model, caching, parallel calls | Batch, async, multi-step chains, reranking added to an already slow path |
| "without code changes" / "without redeploying" | AppConfig, Prompt Management versions and aliases, agent aliases, inference profiles, API Gateway stage variables | Hard-coded model IDs, environment variables that need a redeploy |
| "data must not leave the Region / jurisdiction" | In-Region inference, geographic inference profile matching the jurisdiction, VPC endpoints, Outposts | Global inference profile, cross-geography routing |
| "must not traverse the public internet" | Interface VPC endpoints (PrivateLink) with endpoint policies | NAT gateway, public endpoint with IP allow lists |
| "auditable" / "compliance" / "regulators" | CloudTrail, model invocation logging, Glue Data Catalog lineage, model cards, guardrail trace | Application-level print logging only |
| "deterministic" / "provably correct" | Automated Reasoning checks, text-to-SQL, JSON Schema structured output, rules in code | "Lower the temperature" alone, prompt instructions alone |
| "reduce hallucinations" | Grounding with a knowledge base + contextual grounding check + citations | Fine-tuning, larger model |
| "frequently changing data" | RAG with incremental sync | Fine-tuning (stale the day it finishes) |
| "consistent style / domain vocabulary / output format" at scale | Fine-tuning or distillation | Bigger context window stuffing |
| "human approval required" | Step Functions `waitForTaskToken`, agent user confirmation, return of control | A prompt asking the model to "double check" |
| "existing open-source agent code" / "any framework" | AgentCore Runtime | Rewriting as Bedrock Agents |

### Five distractor families

Learn to name the distractor. Once you can name it, you can discard it in seconds.

1. **The hand-rolled rebuild.** Lambda + custom chunking + custom embeddings loop when Knowledge Bases does it. Wrong whenever the stem says least overhead. Right only when the stem names a requirement the managed feature cannot meet (custom chunk logic, unsupported store, special parser).
2. **The wrong layer.** Fixing a retrieval problem with a prompt change, a safety problem with temperature, a throttling problem with a bigger model, a cost problem with Provisioned Throughput.
3. **The traditional-ML reflex.** SageMaker training jobs, Clarify, Model Monitor, or Ground Truth offered where Bedrock Evaluations, Guardrails, or fine-tuning via Bedrock is the GenAI-native answer. SageMaker is right only for self-hosted or open-weight models, LoRA adapters, model registry, and model cards.
4. **The plausible-but-nonexistent feature.** A service used for something it does not do: Macie scanning prompts in real time (it scans S3), Guardrails encrypting data, CloudTrail capturing prompt bodies (that is model invocation logging), Comprehend generating embeddings.
5. **The half answer.** Solves the symptom but ignores the qualifier. Example: adds retries for throttling when the stem said "sustained 3x traffic growth" (needs cross-Region inference or quota/capacity change, not just backoff).

### Multiple-response questions

- There is no partial credit. Treat each option as an independent true/false statement against the stem.
- The correct options are usually **complementary layers**, not alternatives. Typical pair: one control at the input and one at the output; one for detection and one for prevention; one for storage and one for retrieval.
- If two options do the same job in different ways, at most one is correct.
- The stem tells you the count ("Select TWO"). If you have three candidates, the one that repeats a layer already covered is the one to drop.

---

## 3. The five-pass method for any question

1. **Read the last sentence first.** It holds the actual question and the qualifier.
2. **Classify the problem layer:** data → retrieval → prompt → model → orchestration → safety → operations. The correct answer acts on the layer where the problem is.
3. **Predict the answer before reading the options.** If your prediction is there, it is right about 80% of the time.
4. **Eliminate by distractor family.** Two options usually fall immediately.
5. **Break ties with the qualifier.** Between two working answers, the one that matches the qualifier with the fewest moving parts wins.

**Time plan:** first pass 110 minutes (about 90 seconds each, flag anything over 2 minutes), second pass 50 minutes on flagged items, 20 minutes to recheck multiple-response questions. Long scenario questions are not harder; skim the scenario and read the last sentence.

---

## 4. Problem-layer diagnosis (the most reused skill on the exam)

A large share of questions across all five domains reduce to "which layer is broken?" Memorize this table.

| Symptom in the stem | Layer | First fix to look for |
|---|---|---|
| Answers are fluent but cite wrong or unrelated passages | Retrieval | Chunking strategy, hybrid search, reranker, metadata filter |
| Right documents retrieved, answer still wrong or invented | Generation | Contextual grounding check, prompt instructions to answer only from context, lower temperature |
| Answers are correct but outdated | Data freshness | Incremental sync, event-driven ingestion, scheduled refresh |
| Answer is cut off mid-sentence | Output limit | `max_tokens`, check `stopReason = max_tokens` |
| Model ignores information that is in the documents | Context window | Too many or too large chunks, truncation, reduce top-k or use hierarchical chunking |
| Exact product codes, IDs, or acronyms not found | Lexical gap | Hybrid search (keyword + vector) |
| Multi-part question gets a partial answer | Query handling | Query decomposition |
| Vague or conversational follow-up retrieves junk | Query handling | Query reformulation / rewrite with conversation history |
| Answers differ between runs | Sampling | Temperature, top-p, fixed prompt version, structured output |
| Quality dropped after a prompt edit | Prompt governance | Prompt versions, regression test on golden dataset, rollback |
| Quality dropped with no deployment | Drift | Model version change, data drift, embedding mismatch after re-index |
| `ThrottlingException` (429) | Capacity | Backoff with jitter, cross-Region inference, quota increase, Provisioned Throughput / Reserved tier |
| `ValidationException` on long inputs | Input size | Token counting, chunk or summarize input |
| `AccessDeniedException` | Permissions | Model access, IAM action, SCP, inference profile needs permission on every destination Region |
| Agent loops or calls the wrong tool | Agent design | Tool descriptions, fewer tools per agent, max iterations, trace analysis |
| Harmful or off-topic output reaches users | Safety | Guardrail on input and output, denied topics |
| Costs spiking with stable traffic | Tokens | Prompt bloat, missing cache, context growth in long conversations |

---

## 5. Service-selection anchors

These pairings appear verbatim in the exam guide's skill statements. When a stem describes the left column, the right column is the expected answer.

| Need | Expected service or feature |
|---|---|
| Switch models or providers without code change | AWS AppConfig (+ Lambda, API Gateway) |
| Survive a model or Region disruption | Step Functions circuit breaker, Bedrock cross-Region inference, graceful degradation to a fallback model or cached answer |
| Deploy fine-tuned or domain model you host | SageMaker AI endpoints, Model Registry for versions, LoRA adapters |
| Validate data quality before ingestion | AWS Glue Data Quality, SageMaker Data Wrangler, Lambda, CloudWatch metrics |
| Audio into an FM pipeline | Amazon Transcribe first |
| Extract entities, intent, or PII before the FM | Amazon Comprehend |
| Sequential prompt chain with branching, no code | Amazon Bedrock Flows (called Prompt Flows in the exam guide) |
| Parameterized prompt templates, versions, approval | Amazon Bedrock Prompt Management |
| Conversation history store | Amazon DynamoDB |
| Clarification loop or human approval | AWS Step Functions |
| Multi-agent orchestration, AWS native | Strands Agents, AWS Agent Squad |
| Agent-to-tool standard | Model Context Protocol (MCP) |
| Lightweight stateless MCP server | AWS Lambda |
| Complex or stateful MCP server | Amazon ECS |
| ReAct loop with explicit control | Step Functions |
| Stopping conditions, timeouts, boundaries for agents | Step Functions (max iterations), Lambda (timeouts), IAM (resource boundaries), circuit breakers |
| Real-time token delivery to a browser | Bedrock streaming APIs + WebSocket or server-sent events, API Gateway |
| Async, high-volume FM calls | Amazon SQS + SDK |
| Trace a request across services | AWS X-Ray |
| Analyze prompts and responses in logs | CloudWatch Logs Insights over model invocation logs |
| On-premises data residency | AWS Outposts |
| Edge, ultra-low latency over 5G | AWS Wavelength |
| CI/CD for GenAI components | CodePipeline + CodeBuild with automated evaluation gates |
| Documents, images, audio, video into structured output | Amazon Bedrock Data Automation |
| Detect PII in stored documents | Amazon Macie (S3) |
| Detect PII in text in flight | Amazon Comprehend, Guardrails sensitive information filter |
| Fine-grained access to source data | AWS Lake Formation |
| Data lineage and source registration | AWS Glue Data Catalog |
| Document model limitations | SageMaker Model Cards |
| Audit of API activity | AWS CloudTrail |
| Automated evaluation at scale | Bedrock Evaluations with LLM-as-a-judge |
| Reasoning trace of an agent | Bedrock Agents trace, AgentCore Observability |

---

## 6. Core decision trees

### Customization ladder (cheapest and fastest first)

1. Prompt engineering (instructions, examples, structure)
2. RAG (private, changing, or citable knowledge)
3. Agents and tools (the model must act or fetch live data)
4. Fine-tuning (behaviour, tone, format, domain vocabulary; needs labeled pairs)
5. Distillation (keep quality of a large model at the cost and latency of a small one)
6. Continued pre-training (unlabeled domain corpus; rarely the answer)

The exam rewards climbing only as far as the requirement forces you. "Knowledge" problems stop at RAG. "Behaviour" problems go to fine-tuning. "Cost at stable quality" goes to distillation.

### Capacity and pricing mode

| Workload shape | Choice |
|---|---|
| Spiky, unpredictable, interactive | On-demand Standard tier, add cross-Region inference for headroom |
| Latency-critical, customer-facing, willing to pay more | Priority tier or latency-optimized inference |
| Not time-sensitive but still synchronous (agentic background work, evaluations) | Flex tier |
| Large offline jobs, results in hours | Batch inference (about 50% lower price, S3 in and out) |
| Steady, predictable high volume, or custom model needing dedicated capacity | Provisioned Throughput / Reserved capacity |
| Repeated long prefix (system prompt, document, tool list) | Prompt caching |
| Mixed easy and hard questions | Intelligent Prompt Routing or a model cascade |

### Where does the agent run?

| Stem says | Choice |
|---|---|
| Fully managed, configure in console, action groups, minimal code | Amazon Bedrock Agents |
| Existing LangGraph / CrewAI / Strands code, long-running sessions, session isolation, any model | Amazon Bedrock AgentCore Runtime |
| Code-first, model-driven, AWS-native SDK | Strands Agents (deploy on AgentCore, Lambda, ECS) |
| Route user requests to specialist agents and keep per-agent history | AWS Agent Squad |
| Deterministic steps, audit trail, human approval, explicit retries | AWS Step Functions |
| Fixed sequence of prompts with simple branching, no code | Amazon Bedrock Flows |

---

## 7. Study sequence

| Week | Focus | Output |
|---|---|---|
| 1 | Domain 1 Tasks 1.4–1.5 (vector stores, retrieval) | Build one Knowledge Base, change chunking and search type, observe the difference |
| 2 | Domain 1 Tasks 1.1–1.3, 1.6 (design, FM selection, data, prompts) | One Flow with a condition node, one managed prompt with two versions |
| 3 | Domain 2 Tasks 2.1, 2.4 (agents, APIs) | One agent with an action group and return of control; one Strands agent with an MCP tool |
| 4 | Domain 2 Tasks 2.2, 2.3, 2.5 (deployment, enterprise, tooling) | Streaming through API Gateway or a Lambda function URL |
| 5 | Domain 3 | Guardrail with every policy type, test with `ApplyGuardrail`; enable invocation logging |
| 6 | Domains 4 and 5 | One Bedrock evaluation job (LLM-as-a-judge and RAG); a CloudWatch dashboard on token metrics |
| 7 | Official practice set, review every wrong answer by problem layer | List of weak skills mapped to the skill numbers |

Hands-on work matters more for this exam than for most. Many options differ only in a parameter or API name you will recognize instantly once you have used it.

---

## 8. Architecture diagrams

All diagrams are in one multi-page draw.io file: [diagrams/AIP-C01-Architectures.drawio](diagrams/AIP-C01-Architectures.drawio). Open it at app.diagrams.net (File → Open from → Device) or with the draw.io extension for VS Code. The same architectures appear as Mermaid diagrams inside each domain document.

| Page | Architecture | Domain |
|---|---|---|
| 1 | Managed RAG: ingestion and query path | 1 |
| 2 | Advanced retrieval: query transformation, hybrid search, rerank | 1 |
| 3 | Dynamic model selection and resilience | 1 |
| 4 | Multimodal data processing pipeline | 1 |
| 5 | Agentic architecture with AgentCore and MCP | 2 |
| 6 | GenAI gateway for enterprise consumption | 2 |
| 7 | Real-time streaming and asynchronous integration | 2 |
| 8 | Defense-in-depth safety pipeline | 3 |
| 9 | Private, governed GenAI environment | 3 |
| 10 | Cost and latency optimization: cache and cascade | 4 |
| 11 | GenAI observability stack | 4 |
| 12 | Evaluation and CI/CD quality gate | 5 |

---

## Sources

- [AWS Certified Generative AI Developer – Professional (AIP-C01) exam guide](https://docs.aws.amazon.com/aws-certification/latest/ai-professional-01/ai-professional-01.html)
- [AWS certification page: exam overview](https://aws.amazon.com/certification/certified-generative-ai-developer-professional/)
- [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/)
- [Amazon Bedrock AgentCore Developer Guide](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/)
