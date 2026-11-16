# AIP-C01 — Topics to Revisit

Topics you asked about while studying, with where each sits in the exam guide and where to read more. New questions are added to the bottom of the table. Change the status as you go: **To revisit** → **In progress** → **Done**.

Back to the [index](INDEX.md).

## Tracker

| # | Asked | Topic | Domain, task, skill | Status |
|---|---|---|---|---|
| 1 | 2026-11-15 | Fine-tuning | Domain 1, Task 1.2, Skill 1.2.4 (named in the guide) | To revisit |
| 2 | 2026-11-15 | Knowledge distillation | Not named. Fits Domain 4, Task 4.1, Skill 4.1.2 and Domain 2, Task 2.2, Skill 2.2.3 | To revisit |
| 3 | 2026-11-15 | Pre-training / continued pre-training | Not named. Closest is Domain 1, Task 1.2, Skill 1.2.4 | To revisit |
| 4 | 2026-11-15 | Reranking | Domain 1, Task 1.5, Skill 1.5.4 (named). Also Domain 4, Skill 4.2.2 and Domain 5, Skill 5.2.4 | To revisit |
| 5 | 2026-11-15 | Prompt routing (intelligent model routing) | Domain 2, Task 2.4, Skill 2.4.4 (routing is named; the Bedrock feature is not). Also Domain 4, Skill 4.1.2 and Domain 2, Skill 2.2.3 | To revisit |
| 6 | 2026-11-15 | When to use Strands Agents vs Bedrock Flows vs Step Functions workflows | Strands: Domain 2, Skills 2.1.1, 2.1.6, 2.5.5. Flows: Domain 1, Skill 1.6.6 and Domain 2, Skill 2.5.2. Step Functions: Domain 2, Skills 2.1.2, 2.1.3, 2.1.5, 2.4.4, plus Domain 1, Skills 1.2.3, 1.5.5, 1.6.2 | To revisit |
| 7 | 2026-11-15 | Improving relevance vs precision in GenAI applications | Precision: Domain 1, Skill 1.4.2 (metadata). Relevance: Domain 1, Skill 1.5.4 (search architectures) and Skill 1.5.5 (query handling). Also Domain 4, Skill 4.2.2 and Domain 5, Skills 5.1.6 and 5.2.4 | To revisit |
| 8 | 2026-11-15 | Amazon Comprehend: features, when to use it, when not to | Named in Domain 1, Skill 1.3.4 (entities) and Skill 1.6.2 (intent); Domain 3, Skill 3.1.4 (pre-processing filter), Skills 3.2.2 and 3.2.3 (PII) | To revisit |
| 9 | 2026-11-15 | Amazon Bedrock Data Automation (BDA): features, when to use it, when not to | Named in Domain 2, Task 2.5, Skill 2.5.3 (automated data processing workflows). Also fits Domain 1, Skill 1.3.2 (multimodal pipelines) and Skill 1.5.1 (parser choice before chunking); Domain 2, Skill 2.1.5 (human review); Domain 5, Skill 5.2.4 (tables and figures not retrieved) | To revisit |

## Study notes per topic

### 1. Fine-tuning

- **Exam angle:** deployment and lifecycle, not training. When to choose it over RAG, how to serve and version the result, how to roll back or retire it.
- **Rule:** behaviour problems (tone, format, domain vocabulary) go to fine-tuning. Knowledge problems (fresh data, citations) go to RAG.
- **In the study pack:** [1.2.4 Customization, deployment, lifecycle](01-Domain1-FM-Integration-Data-Compliance.md#124-customization-deployment-lifecycle) · [Customization ladder](00-Exam-Strategy-and-Question-Patterns.md#customization-ladder-cheapest-and-fastest-first)
- **Go deeper:** serving a custom model (Provisioned Throughput vs on-demand), LoRA adapters as SageMaker inference components, Model Registry approval flow, blue/green rollback.
- **AWS docs:** [Customize your model in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html)

### 2. Knowledge distillation

- **Exam angle:** the answer to "keep large-model quality at small-model cost and latency" for a narrow task.
- **Rule:** a cost problem at stable quality is distillation. A teacher model generates training data; a smaller student is tuned to match. Production invocation logs can be the source data.
- **In the study pack:** [4.1.2 Cost-effective model selection](04-Domain4-Operational-Efficiency-Optimization.md#412-cost-effective-model-selection) · [1.2.4 Customization, deployment, lifecycle](01-Domain1-FM-Integration-Data-Compliance.md#124-customization-deployment-lifecycle) · [Optimized approaches (Skill 2.2.3)](02-Domain2-Implementation-Integration.md#optimized-approaches-skill-223)
- **Go deeper:** distillation vs fine-tuning vs Intelligent Prompt Routing vs a model cascade, and which one a "reduce cost without losing quality" stem points to.
- **AWS docs:** [Model distillation in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/model-distillation.html)

### 3. Pre-training / continued pre-training

- **Exam angle:** mostly a distractor. Training from scratch is out of scope for the exam.
- **Rule:** continued pre-training needs a large unlabeled domain corpus and adapts the model to domain language. It does not give citations or fresh facts, so it loses to RAG for knowledge questions and to fine-tuning when labeled pairs exist.
- **In the study pack:** [1.2.4 Customization, deployment, lifecycle](01-Domain1-FM-Integration-Data-Compliance.md#124-customization-deployment-lifecycle) · [Customization ladder](00-Exam-Strategy-and-Question-Patterns.md#customization-ladder-cheapest-and-fastest-first)
- **Go deeper:** labeled vs unlabeled data requirements for each customization method, and the "smaller pre-trained models for specific tasks" wording in Skill 2.2.3.
- **AWS docs:** [Customize your model in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html)

### 4. Reranking

- **Exam angle:** the fix for "right document retrieved, but ranked low". Retrieve a wide candidate set, rerank, pass only the top few to the model.
- **Rule:** right document, wrong rank is the reranker. Right concept, wrong exact term is hybrid search.
- **Trade-off:** better relevance and fewer generation tokens, at the cost of extra latency and a per-query charge.
- **In the study pack:** [1.5.3 and 1.5.4 Search architectures](01-Domain1-FM-Integration-Data-Compliance.md#153-and-154-search-architectures) · [Advanced retrieval diagram](01-Domain1-FM-Integration-Data-Compliance.md#architecture-advanced-retrieval) · [4.2.2 Retrieval performance](04-Domain4-Operational-Efficiency-Optimization.md#422-retrieval-performance) · [Fault isolation for RAG](05-Domain5-Testing-Validation-Troubleshooting.md#fault-isolation-for-rag)
- **Go deeper:** the standalone Rerank API vs reranking inside Retrieve, how many candidates to retrieve before reranking, and reranking combined with metadata filters and hybrid search.
- **AWS docs:** [Reranker models in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/rerank.html)

### 5. Prompt routing (intelligent model routing)

- **Exam angle:** choosing how requests reach the right model. Skill 2.4.4 lists four routing styles: static routing in application code, dynamic content-based routing with Step Functions, intelligent routing based on metrics, and API Gateway request transformations.
- **Bedrock Intelligent Prompt Routing:** one serverless endpoint that predicts each model's response quality for a request and routes between models in the **same model family**. You can use a default router or configure your own with a fallback model and a response quality difference threshold.
- **Rule:** "mixed easy and hard questions, cut cost without losing quality, least overhead" is Intelligent Prompt Routing. Routing across different providers or to specialist models needs your own router (application code, Step Functions, or a GenAI gateway).
- **In the study pack:** [Routing (Skill 2.4.4)](02-Domain2-Implementation-Integration.md#routing-skill-244) · [4.1.2 Cost-effective model selection](04-Domain4-Operational-Efficiency-Optimization.md#412-cost-effective-model-selection) · [Optimized approaches (Skill 2.2.3)](02-Domain2-Implementation-Integration.md#optimized-approaches-skill-223) · [Capacity and pricing mode](00-Exam-Strategy-and-Question-Patterns.md#capacity-and-pricing-mode)
- **Go deeper:** prompt routing vs a model cascade (route up front vs escalate after a failed check) vs distillation; the same-family limit; how the fallback model and quality threshold change cost.
- **AWS docs:** [Intelligent prompt routing in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-routing.html)

### 6. When to use Strands Agents vs Bedrock Flows vs Step Functions workflows

"Bedrock Workflows" is taken here to mean **Amazon Bedrock Flows**, which the exam guide still calls "Prompt Flows".

- **Exam angle:** who controls the sequence of steps. The model (Strands), a fixed prompt chain you draw (Flows), or a deterministic state machine you define (Step Functions).

| | Strands Agents | Amazon Bedrock Flows | AWS Step Functions |
|---|---|---|---|
| Who decides the next step | The model, at run time | You, at design time (nodes and conditions) | You, at design time (states and choices) |
| How it is built | Code (an SDK that runs in your own process) | Visual builder, no code | State machine definition |
| Best for | Open-ended tasks where the path is not known in advance; tool use; multi-agent patterns in code | Sequential prompt chains with simple branching, knowledge base and agent steps, inside Bedrock | Long-running, cross-service orchestration with retries, error handling, human approval and an execution history |
| Stem signals | "Agent decides which tools", "code-first", "AWS-native SDK", "custom tool behaviour" | "No code", "business analysts", "prompt chain", "branch on model output" | "Deterministic", "auditable", "approval", "stopping conditions", "circuit breaker", "ReAct as a workflow" |
| Weak at | Guaranteed, repeatable step order; hard limits unless you add them | Waiting on humans, complex error handling, orchestration outside Bedrock | Open-ended reasoning; it only does what the states say |

- **Rules:**
  - Path unknown until run time → an agent (Strands when the stem says code-first or SDK).
  - Fixed prompt chain, no code → Flows.
  - Must be deterministic, auditable, pausable for approval, or span many services → Step Functions.
  - They combine: Step Functions can wrap an agent to enforce iteration limits, timeouts and approvals; a Flow can call an agent or a Lambda function as one node.
- **In the study pack:** [The agent platform map](02-Domain2-Implementation-Integration.md#the-agent-platform-map) · [Strands Agents and Agent Squad](02-Domain2-Implementation-Integration.md#strands-agents-and-agent-squad) · [ReAct with Step Functions](02-Domain2-Implementation-Integration.md#react-and-structured-reasoning-with-step-functions) · [Human collaboration](02-Domain2-Implementation-Integration.md#human-collaboration-skill-215) · [Bedrock Flows in Task 1.6](01-Domain1-FM-Integration-Data-Compliance.md#task-16--prompt-engineering-and-governance) · [Where does the agent run?](00-Exam-Strategy-and-Question-Patterns.md#where-does-the-agent-run) · [Domain 1 tip 8: Flows vs Step Functions](01-Domain1-FM-Integration-Data-Compliance.md#tips-and-tricks)
- **Go deeper:** Strands multi-agent patterns (agents-as-tools, swarm, graph, workflow) and where a Strands agent is deployed (AgentCore Runtime, Lambda, ECS); Flows node types, versions and aliases; Step Functions `waitForTaskToken`, Retry and Catch, and the optimized Bedrock integration.
- **Docs:** [Amazon Bedrock Flows](https://docs.aws.amazon.com/bedrock/latest/userguide/flows.html) · [Step Functions with Amazon Bedrock](https://docs.aws.amazon.com/step-functions/latest/dg/connect-bedrock.html) · [Strands Agents SDK](https://strandsagents.com/docs/user-guide/sdk/quickstart/overview/)

### 7. Improving relevance vs precision in GenAI applications

- **Exam angle:** the stem describes a retrieval or answer-quality symptom, and you must tell which quality is failing before choosing the fix. The exam guide uses the words deliberately: Skill 1.4.2 says metadata improves "search **precision**"; Skill 1.5.4 says hybrid search and rerankers improve "**relevance** and accuracy".

**The four terms**

| Term | Question it answers | Bedrock evaluation metric |
|---|---|---|
| Precision | Of what was retrieved, how much is useful? (noise) | Context relevance (retrieve-only job) |
| Recall | Of what is useful, how much was retrieved? (misses) | Context coverage (needs ground truth) |
| Relevance of ranking | Are the most useful chunks at the top? | Context relevance, inspected by position |
| Answer relevance | Does the response address the question that was asked? | Helpfulness, completeness; Guardrails relevance threshold |

**Levers by goal**

| Goal | Levers | Cost of the lever |
|---|---|---|
| Raise precision (cut noise) | Metadata filters (manual or implicit); reranker then keep only the top few; lower number of results; hybrid search for exact terms; separate indexes per domain; tighter chunks | May drop something useful (lower recall) |
| Raise recall (stop missing things) | Higher number of results; hybrid search; query expansion or rewrite; query decomposition; better parser for tables and scans; more complete ingestion | More noise and more tokens |
| Improve ranking relevance | Reranker model; embedding model that fits the domain; chunking strategy that keeps ideas whole (semantic, hierarchical) | Reranker adds latency and a per-query charge; chunking or embedding changes need re-ingestion |
| Improve answer relevance | Prompt instructions to answer only the question from the context; query rewrite with conversation history; contextual grounding check with a relevance threshold | Stricter thresholds block more answers |
| Get precision and recall together | **Retrieve wide, rerank narrow:** fetch 25 to 50 candidates, rerank, pass the top 3 to 5 | One extra model call |

**Exam tips and tricks**

1. **Match the word in the stem.** "Precision", "too many irrelevant passages", "wrong product or wrong year returned" → metadata filtering. "Relevance", "best passage is not at the top" → reranker. "Exact codes or IDs missed" → hybrid search.
2. **Three symptoms, three fixes.** Noise in the results is a precision problem (filter, rerank, fewer results). Missing information is a recall problem (hybrid, expansion, more results). Right chunk but low in the list is a ranking problem (reranker).
3. **Raising the number of results is a trade, not a fix.** It helps recall and hurts precision, cost and latency. An option that only raises it to 100 is usually the distractor.
4. **Retrieve wide then rerank narrow** is the standard answer when the stem wants both better relevance and lower generation cost.
5. **Filters are applied before or during the search**, so they improve precision and speed together. A reranker runs after the search and adds latency. If the stem says "without adding latency", prefer the filter.
6. **Users do not type filters?** Implicit metadata filtering derives them from the query.
7. **Retrieval problems are never fixed by a larger generation model, fine-tuning, or temperature.** Discard those options first.
8. **Measure before changing.** Low context relevance means a precision problem; low context coverage means a recall problem. A retrieve-only RAG evaluation job tells you which.
9. **Grounding and relevance are separate thresholds** in the contextual grounding check. Grounding asks "is it supported by the source?"; relevance asks "does it answer the query?". A factually correct but off-topic answer fails relevance, not grounding.
10. **Chunking, parser and embedding changes need re-ingestion** before you test again. Query-time changes (search type, filters, reranker, number of results) do not.
11. **In a shared index, the metadata filter is also the security boundary.** The same control answers a precision question in Domain 1 and a tenant-isolation question in Domain 3.
12. **Answer precision** (exact, consistent output rather than retrieval) is a different layer: structured output with a schema, lower temperature, and text-to-SQL for exact numbers.

- **In the study pack:** [1.4.2 Metadata frameworks](01-Domain1-FM-Integration-Data-Compliance.md#142-metadata-frameworks) · [1.5.3 and 1.5.4 Search architectures](01-Domain1-FM-Integration-Data-Compliance.md#153-and-154-search-architectures) · [1.5.1 Chunking](01-Domain1-FM-Integration-Data-Compliance.md#151-chunking) · [1.5.5 Query handling](01-Domain1-FM-Integration-Data-Compliance.md#155-query-handling) · [Advanced retrieval diagram](01-Domain1-FM-Integration-Data-Compliance.md#architecture-advanced-retrieval) · [4.2.2 Retrieval performance](04-Domain4-Operational-Efficiency-Optimization.md#422-retrieval-performance) · [Metrics vocabulary](05-Domain5-Testing-Validation-Troubleshooting.md#metrics-vocabulary) · [Fault isolation for RAG](05-Domain5-Testing-Validation-Troubleshooting.md#fault-isolation-for-rag) · [5.2.4 Retrieval system issues](05-Domain5-Testing-Validation-Troubleshooting.md#524-retrieval-system-issues) · [Hallucination controls](03-Domain3-Safety-Security-Governance.md#hallucination-controls-skill-313-choose-by-required-strength) · [Problem-layer diagnosis](00-Exam-Strategy-and-Question-Patterns.md#4-problem-layer-diagnosis-the-most-reused-skill-on-the-exam)
- **Go deeper:** run one retrieve-only evaluation job and read context relevance against context coverage; try the same query with semantic search, hybrid search, a filter, and a reranker, and compare the returned chunks.
- **AWS docs:** [Configure queries: results, search type, filters, reranking, decomposition](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-test-config.html) · [RAG evaluation metrics](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-metrics.html) · [Reranker models](https://docs.aws.amazon.com/bedrock/latest/userguide/rerank.html)

### 8. Amazon Comprehend: features, when to use it, when not to

- **Exam angle:** Comprehend is the exam's "classic NLP helper around the FM". It appears in five skill statements, always as a supporting step: extracting entities to improve input quality (1.3.4), recognizing intent (1.6.2), acting as the **pre-processing filter** in defense in depth (3.1.4), and detecting PII (3.2.2, 3.2.3). It analyses text you pass to it and returns labels, scores and offsets. It never generates, stores, blocks or scans on its own.

**Features**

| Feature | What it returns | Exam use |
|---|---|---|
| Entity recognition | People, places, organizations, dates, quantities | Enrich documents with metadata before ingestion; normalize inputs (Skill 1.3.4) |
| Key phrases | Main noun phrases | Tags and metadata for filtering |
| Dominant language | Language code with a score | Route to a language-specific prompt, model or index |
| Sentiment and targeted sentiment | Positive, negative, neutral, mixed (overall or per entity) | Escalation rules, prioritizing tickets, business metrics |
| Syntax | Parts of speech | Rarely the answer |
| **PII detection** | PII type, confidence score, character offsets (real time), or a redacted copy (asynchronous batch job) | Detect or redact PII before text reaches the FM, the vector store or the logs (Skills 3.2.2, 3.2.3) |
| **Toxicity detection** | Scores per category (hate speech, harassment, insult, profanity, sexual, violence, graphic) | Moderation of prompts or outputs in a custom workflow |
| **Custom classification** | Your own categories | **Intent recognition** and routing (Skill 1.6.2); document type classification |
| **Custom entity recognition** | Your own entity types | Domain terms such as part numbers or policy IDs |
| Flywheels | Managed retraining of custom models | Keeping a custom classifier current |
| Topic modeling, event detection, prompt safety classification | n/a | **No longer available to new customers since 30 April 2026.** AWS points to Bedrock models for topics and events and to Bedrock Guardrails for prompt safety |

**When to use it**

| Stem says | Why Comprehend |
|---|---|
| "Pre-processing filter" before the model in a layered safety design | It is the service the exam guide names for that layer |
| "Detect or redact PII in documents before ingestion", "in a Lambda or Glue step" | Text is in your pipeline, not in a model call |
| "Redact PII across a large set of documents in S3" | Asynchronous PII redaction job writes redacted copies |
| "Extract entities or key phrases to build metadata" | Cheap, deterministic, no prompt to maintain |
| "Detect the language and route" | Dominant language detection |
| "Classify the user's intent and route to the right flow" at high volume | Custom classifier endpoint: low latency, predictable cost |
| "Lowest cost, high volume, deterministic NLP" where an FM is also offered | Purpose-built API beats an FM call on cost and consistency |

**When not to use it**

| Stem says | Use instead | Why not Comprehend |
|---|---|---|
| "Discover which S3 buckets contain sensitive data" | Amazon Macie | Comprehend does not scan storage by itself |
| "Mask PII in model responses", "block prompts with SSNs", least code | Bedrock Guardrails sensitive information filter | Guardrails acts inline with the model call; Comprehend needs your own code to act on the result |
| "Detect prompt injection or jailbreaks" | Guardrails prompt attack filter | Comprehend's prompt safety classifier is closed to new customers |
| "Restrict the assistant to approved subjects" | Guardrails denied topics | Comprehend has no topic policy |
| "Check the answer is supported by the source" | Contextual grounding check | Comprehend does not compare text to a source |
| "Generate, summarize, rewrite, translate" | An FM (Amazon Translate for translation) | Comprehend only analyses |
| "Create embeddings", "semantic search" | Embedding model, Knowledge Bases, OpenSearch | Not a vector or search service |
| "Text inside scanned images or PDFs", "audio" | Textract or Bedrock Data Automation, Transcribe, then Comprehend | Insight APIs take text input |
| "Medical terms, medications, PHI" | Amazon Comprehend Medical | Separate service |
| "Nuanced or open-ended classification with few examples" | An FM with a prompt | Custom classifier needs labelled training data |

**Exam tips and tricks**

1. **Comprehend detects; your code acts.** It returns types, scores and offsets. Blocking, masking or routing is Lambda logic. An option that says Comprehend "blocks" or "prevents" something by itself is wrong.
2. **Real-time PII detection returns offsets, not redacted text.** A redacted copy comes only from the asynchronous batch job. Real-time redaction means Lambda uses the offsets to mask.
3. **Three PII services, three locations.** Macie: data at rest in S3. Comprehend: text passing through your pipeline. Guardrails: the prompt and the response of a model call.
4. **In a defense-in-depth question, Comprehend is the first layer** (pre-processing), Guardrails is the model layer, Lambda is post-processing, API Gateway filters the response.
5. **Comprehend plus Guardrails is not redundant.** In a "Select TWO" question they are complementary layers at different points.
6. **"Least operational overhead" for safety on a Bedrock model call points to Guardrails,** not a Comprehend-plus-Lambda build. Comprehend wins when the text is outside a model call or the stem names pre-processing.
7. **Intent recognition by Comprehend means custom classification.** For a conversational bot with slots, Amazon Lex is the alternative; for a few fuzzy intents, a small FM.
8. **Entities become metadata.** Entity extraction during ingestion feeds metadata filtering, which is how it improves retrieval precision.
9. **Watch the limits.** PII detection covers English and Spanish; toxicity detection is English only and takes short text segments; real-time calls have small size limits, so large documents go through asynchronous jobs.
10. **Data handling differs from Bedrock.** Comprehend may store content to improve its models unless you opt out with an AWS Organizations AI services opt-out policy. Bedrock does not use your content for training.
11. **Comprehend is not a compliance scanner or an access control.** It does not replace Macie, IAM, Lake Formation or encryption.

- **In the study pack:** [PII: which service at which stage](03-Domain3-Safety-Security-Governance.md#pii-which-service-at-which-stage) · [Defense in depth](03-Domain3-Safety-Security-Governance.md#defense-in-depth-skill-314-the-layered-pipeline-in-the-exam-guide) · [Service boundaries table](03-Domain3-Safety-Security-Governance.md#what-this-domain-is-really-testing) · [Guardrails policy matrix](03-Domain3-Safety-Security-Governance.md#guardrails-policy-matrix) · [Task 1.3 data processing](01-Domain1-FM-Integration-Data-Compliance.md#task-13--data-validation-and-processing) · [Multimodal pipeline diagram](01-Domain1-FM-Integration-Data-Compliance.md#architecture-multimodal-data-processing-pipeline) · [Task 1.6 interactive systems](01-Domain1-FM-Integration-Data-Compliance.md#task-16--prompt-engineering-and-governance)
- **Go deeper:** call PII detection on a sample and mask it yourself from the offsets; run one asynchronous redaction job; compare the result with a Guardrails sensitive information filter on the same text.
- **AWS docs:** [What is Amazon Comprehend](https://docs.aws.amazon.com/comprehend/latest/dg/what-is.html) · [Detecting PII entities](https://docs.aws.amazon.com/comprehend/latest/dg/how-pii.html) · [Trust and safety (toxicity)](https://docs.aws.amazon.com/comprehend/latest/dg/trust-safety.html) · [Feature availability change](https://docs.aws.amazon.com/comprehend/latest/dg/comprehend-availability-change.html)

### 9. Amazon Bedrock Data Automation (BDA): features, when to use it, when not to

- **Exam angle:** BDA is the exam's "one managed API that turns unstructured content into structured output". The guide names it once, in Skill 2.5.3 ("Amazon Bedrock Data Automation to manage automated data processing workflows"), next to Step Functions for orchestrating document processing. It also fits the multimodal pipelines of Skill 1.3.2, the parser choice in a knowledge base, and human review of low-confidence results (Skill 2.1.5). It takes documents, images, audio and video and returns JSON. It extracts; it does not retrieve, answer questions, orchestrate or act.

**Features**

| Feature | What it does | Exam use |
|---|---|---|
| **Standard output** | Default insights per modality, no schema needed. Documents: text with layout (markdown, tables as CSV), page or element granularity, bounding boxes, summaries, figure captions. Images: summary, detected text, content moderation, ad (IAB) categories, logos. Video: scene summaries, transcript, on-screen text, moderation. Audio: transcript and summary | Prepare content for RAG; media analysis; "no custom fields needed" |
| **Custom output with blueprints** | A blueprint is a list of fields, each with a natural-language instruction, a type (string, number, boolean, array) and an inference type: **explicit** (read from the file) or **inferred** (derived from what is there). Build one from a prompt, by hand, or take one from the catalog (W-2, pay stub, driver's license) | "Extract these exact fields", "downstream system expects a fixed JSON" |
| Normalization and validation | Written into the field instructions: map label variants to one key ("SSN", "Tax ID"), force formats and allowed values | "Dates in one format", "consistent keys across vendors" |
| **Projects** | Hold the standard output settings per modality plus the blueprints. Up to 40 document blueprints per project; one each for image, audio, video. Stages `LIVE` and `DEVELOPMENT`; blueprint versions are immutable snapshots | One resource for many file types; safe change management |
| **Document splitting and blueprint matching** | With the splitter on, one file holding several document types (a lending packet) is split, each part is classified and matched to the right blueprint. Up to 3,000 pages | "Mixed packet in one PDF" |
| **Confidence scores and visual grounding** | Each extracted field carries a confidence score and the bounding box where the value was found | Threshold routes low confidence to human review; "show where the value came from" |
| Asynchronous API | `InvokeDataAutomationAsync`: input in S3, output in S3, returns an invocation ARN. Track with `GetDataAutomationStatus` or an EventBridge job event | Event-driven pipelines; all audio and video |
| Synchronous API | `InvokeDataAutomation`: small inputs, result inline. Images, and short documents up to 10 pages and 50 MB with no splitter (the API page still says images only; the limits page lists documents too) | Low-latency single item |
| **Knowledge Bases parser** | One of three parsers: default (text only, free), BDA (no prompt, priced per page or image), foundation model (customizable prompt, priced per token). Extracted figures can be stored in S3 and returned in source attribution | "Answers miss tables, charts, scanned pages" |
| Audio and video in a knowledge base | BDA converts speech and visuals to text, then the text is embedded. Text queries, full `RetrieveAndGenerate` | Searchable meeting, call and training recordings |
| Cross-Region inference | **Required.** Every call passes a data automation profile ARN (geographic such as `us.data-automation-v1`, or the opt-in global profile). Data is stored in the source Region; processing can run in another Region of the geography | Data residency questions |
| Security | Customer managed KMS key and encryption context on the project and the invocation; CloudTrail records the calls | "Encrypt extracted output with our key" |

**Limits worth knowing:** documents are PDF, TIFF, JPEG, PNG, DOCX up to 500 MB; no password-protected PDFs; printed and handwritten text; document languages are English, German, Spanish, French, Italian, Portuguese; vertical text is not supported. Images are JPEG or PNG up to 5 MB. Audio and video run up to 240 minutes.

**When to use it**

| Stem says | Why BDA |
|---|---|
| "Extract fields from invoices, claims, forms or IDs with the least development effort" | Blueprints replace a hand-built OCR, classification and FM chain |
| "A single upload contains several document types" | Splitter plus several blueprints in one project |
| "Send uncertain extractions to a reviewer", "reduce manual review" | Confidence score per field feeds a threshold |
| "Auditors must verify each extracted value against the source" | Visual grounding (bounding boxes) |
| "One service for documents, images, audio and video" | Single API across the four modalities |
| "Knowledge base answers miss data in tables, charts or scanned pages", least effort | BDA as the data source parser |
| "Make call recordings or training videos searchable by text" | BDA parser produces transcripts and visual descriptions |
| "Summarize scenes, detect unsafe content, find logos in video" | Standard output for video |
| "No ML expertise and no labelled training data" | Fields are described in natural language; nothing is trained |

**When not to use it**

| Stem says | Use instead | Why not BDA |
|---|---|---|
| "Customize the parsing prompt", "domain-specific parsing instructions" | Foundation model parser | BDA parser takes no prompt |
| "Search by image", "visually similar products" | Multimodal embeddings | BDA turns media into text; only text queries work |
| "Text-only documents, lowest cost" | Default parser | Choosing BDA parses and bills every PDF, even text-only ones |
| "Answer a user's question about an uploaded image or PDF in a chat turn" | Multimodal FM through Converse | BDA is an extraction job, not a conversational call |
| "Live captions", "transcribe a call as it happens" | Amazon Transcribe streaming | BDA processes files, not streams |
| "Processing must stay in one named Region" | An in-Region service or model | BDA requires cross-Region inference |
| "Clean, validate or transform tabular data" | AWS Glue, Glue Data Quality, SageMaker Processing | BDA is for unstructured content |
| "Custom libraries or models in a heavy batch transform" | SageMaker Processing | BDA is not a compute environment |
| "Retries, approvals, branching across many steps" | Step Functions, with BDA as one task | BDA does not orchestrate |
| "Entities, sentiment or PII in plain text" | Amazon Comprehend | Text analysis, not extraction from files |
| "Generate, compare or reason across documents" | An FM, with Knowledge Bases for retrieval | BDA stops at structured output |

**Exam tips and tricks**

1. **BDA is the "least development effort" answer for unstructured to structured.** The distractor is a chain of Textract, Transcribe, Rekognition, Lambda and an FM that you build and maintain.
2. **Standard output means generic insights; blueprint means your fields.** "Specific fields", "schema" or "target system format" in the stem points to a blueprint. Custom output always arrives with standard output.
3. **Blueprints are prompts, not trained models.** No labelled data and no training job. Compare with Comprehend custom entity recognition or fine-tuning, which need data.
4. **Explicit reads, inferred reasons.** "Invoice total printed on the page" is explicit. "Is this claim complete?" or a computed value is inferred.
5. **Asynchronous is the default shape.** S3 in, S3 out, completion by EventBridge event. A Lambda function that waits and polls is the wrong design; let the event or a Step Functions task continue the flow.
6. **Confidence scores and visual grounding are BDA's hallucination safeguards.** Low confidence goes to a human; high confidence goes straight through. That is the Skill 2.1.5 pattern.
7. **Tables and charts missing from RAG answers is a parsing problem, not a chunking problem.** Change the parser to BDA or a foundation model and re-ingest.
8. **BDA parser against FM parser: prompt and price.** BDA has no prompt and charges per page. The FM parser lets you edit the prompt and charges per token. "Least effort" favours BDA; "custom instructions" favours the FM parser.
9. **BDA against multimodal embeddings: what does the user type?** Text questions about speech and video content point to BDA. Image queries and visual similarity point to multimodal embeddings.
10. **Multimodal ingestion works on Amazon S3 data sources.** Confluence, SharePoint, Salesforce and the web crawler skip multimodal files.
11. **Cross-Region inference is mandatory.** A geographic profile keeps processing inside the geography; the global profile is opt-in and must be allowed in IAM. A strict single-Region rule rules BDA out.
12. **BDA and Step Functions are partners.** Skill 2.5.3 names both: Step Functions orchestrates the document workflow, BDA does the extraction step, Lambda applies business rules, and a knowledge base or an agent consumes the result.
13. **Use project stages and blueprint versions for safe rollout.** Test in `DEVELOPMENT`, serve from `LIVE`, pin a blueprint version in the invocation.
14. **Limits appear as distractors.** Password-protected PDFs, vertical text, unsupported languages and oversized images fail; the fix is pre-processing or another service, not a retry.

- **In the study pack:** [Task 2.5 integration patterns (BDA in one paragraph)](02-Domain2-Implementation-Integration.md#task-25--application-integration-patterns-and-development-tools) · [Task 1.3 data processing](01-Domain1-FM-Integration-Data-Compliance.md#task-13--data-validation-and-processing) · [Multimodal pipeline diagram](01-Domain1-FM-Integration-Data-Compliance.md#architecture-multimodal-data-processing-pipeline) · [Chunking and parsing](01-Domain1-FM-Integration-Data-Compliance.md#151-chunking) · [Human collaboration](02-Domain2-Implementation-Integration.md#human-collaboration-skill-215) · [Retrieval system issues](05-Domain5-Testing-Validation-Troubleshooting.md#524-retrieval-system-issues)
- **Go deeper:** create a project with one catalog blueprint and one custom blueprint, run a mixed PDF with the splitter on, and read the confidence scores and bounding boxes in the output; then switch a knowledge base data source from the default parser to BDA and compare answers on a table-heavy PDF.
- **AWS docs:** [What is Bedrock Data Automation](https://docs.aws.amazon.com/bedrock/latest/userguide/bda.html) · [How it works](https://docs.aws.amazon.com/bedrock/latest/userguide/bda-how-it-works.html) · [File requirements and limits](https://docs.aws.amazon.com/bedrock/latest/userguide/bda-limits.html) · [Knowledge Bases parsing options](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-advanced-parsing.html) · [Choosing a multimodal processing approach](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-multimodal-choose-approach.html) · [Cross-Region support](https://docs.aws.amazon.com/bedrock/latest/userguide/bda-cris.html)
