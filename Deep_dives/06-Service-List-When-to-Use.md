# AIP-C01 — Service List: What Is in Scope and When to Use It

Every AI and ML service on the official in-scope list, with the exam trigger for choosing it and the trigger for choosing something else. The matching API calls are in the [API list](07-API-List-by-Task.md).

Back to the [index](../INDEX.md).

**How to read the "Guide" column**

- A skill number (for example 1.3.4) means the exam guide names the service in that skill statement. These are the highest-yield services.
- **List only** means the service is on the in-scope list but no skill statement names it. Expect it as a supporting option or a distractor more often than as the star of a question.
- **Not named** means a feature of an in-scope service that the guide does not mention by name.
- Scope was checked against the exam guide on 2026-11-17. AWS says the list is non-exhaustive and can change.

## Contents

- [1. Amazon Bedrock](#1-amazon-bedrock)
- [2. Amazon SageMaker AI](#2-amazon-sagemaker-ai)
- [3. SageMaker named services](#3-sagemaker-named-services)
- [4. AWS Glue](#4-aws-glue)
- [5. Amazon Comprehend](#5-amazon-comprehend)
- [6. Amazon Bedrock Data Automation](#6-amazon-bedrock-data-automation)
- [7. Amazon Lex](#7-amazon-lex)
- [8. Amazon Q and related assistants](#8-amazon-q-and-related-assistants)
- [9. Other AI and ML services in scope](#9-other-ai-and-ml-services-in-scope)
- [10. Frameworks and protocols the guide names](#10-frameworks-and-protocols-the-guide-names)
- [11. AI and ML services that are not in scope](#11-ai-and-ml-services-that-are-not-in-scope)
- [12. Supporting services in scope](#12-supporting-services-in-scope)
- [13. Look-alike services and how to split them](#13-look-alike-services-and-how-to-split-them)
- [Sources](#sources)

---

## 1. Amazon Bedrock

The in-scope list has five Bedrock entries: Amazon Bedrock, Amazon Bedrock AgentCore, Amazon Bedrock Knowledge Bases, Amazon Bedrock Prompt Management and Amazon Bedrock Prompt Flows (the product is now called Amazon Bedrock Flows; the guide uses the older name). Amazon Titan is listed separately.

### Models and inference

| Feature | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|
| Foundation models through the Converse API | "One code path for several models", tool use, multi-turn chat, guardrail on the call | Embeddings, image generation or a model-native parameter the unified API lacks → `InvokeModel` | 1.1.2, 1.3.3 |
| Amazon Titan (embeddings, text, image) | "Embedding model chosen on dimensionality and domain fit"; Titan Text Embeddings V2 offers 1,024, 512 or 256 dimensions | Image or mixed queries → a multimodal embedding model | 1.5.2 |
| Cross-Region inference profiles | "Model has limited Regional availability", throttling at peak, resilience with no code | A hard single-Region rule → on-demand in that Region only | 1.2.3 |
| Provisioned Throughput | Steady, predictable high volume; guaranteed capacity; some custom models | Spiky or low traffic → on-demand; non-urgent bulk → batch | 2.2.1, 4.1.3 |
| Batch inference | Large offline jobs, results can wait, lowest cost | Interactive or streaming requests | 4.2.3 |
| Prompt caching | Long repeated prefix (system prompt, document, tool list) | Prompts with no stable prefix; exact-answer reuse → your own response cache | 4.1.4 |
| Service tiers (Priority, Standard, Flex) | Pay more for latency-critical traffic or less for delay-tolerant traffic on the same model | Guaranteed capacity → Provisioned Throughput | Not named |
| Intelligent Prompt Routing | "Route simple prompts to a cheaper model in the same family" with no custom code | Routing across providers or on business rules → your own router (Lambda, Step Functions) | 2.4.4 |
| Model invocation logging | "Record full prompts and responses" to CloudWatch Logs or S3 | "Who called which API" → CloudTrail | Not named |

### Retrieval, agents and orchestration

| Feature | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|
| **Knowledge Bases** | Managed RAG with least overhead: ingestion, chunking, embedding, vector store, retrieval, citations | A chunking or ranking need the managed options cannot meet → custom pipeline on OpenSearch | 1.4.1, 1.5.3, 3.1.3 |
| Bedrock Agents | Managed agent with action groups, knowledge bases, memory, return of control, least code | Code-first control, any framework, long-running sessions → AgentCore Runtime with Strands | 5.1.7 (agent evaluations), 3.4.1 (agent tracing) |
| **AgentCore** (Runtime, Memory, Gateway, Identity, Policy, Observability, Evaluations, Code Interpreter, Browser) | "Deploy an agent built with any framework", session isolation, "expose existing APIs as MCP tools", "agent acts on behalf of a user", managed agent memory | A simple managed agent with no custom code → Bedrock Agents | List only |
| **Prompt Flows** (Flows) | "Sequential prompt chain", conditional branching, "no-code workflow builder" | Retries, human approval, long waits, many AWS integrations → Step Functions | 1.6.6, 2.5.2, 3.4.2 |
| **Prompt Management** | Versioned, parameterized prompt templates, approval workflow, shared across apps | Just storing template files → S3 (also named in the guide) | 1.6.1, 1.6.3, 3.4.2 |
| Data Automation | Documents, images, audio, video into structured output | See [section 6](#6-amazon-bedrock-data-automation) | 2.5.3 |

### Safety, evaluation and customization

| Feature | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|
| **Guardrails** | Filter harmful content, deny topics, mask PII, block prompt attacks, check grounding; `ApplyGuardrail` covers models outside Bedrock | Text outside any model call → Comprehend; data at rest in S3 → Macie | 1.6.1, 3.1.1, 3.1.2, 3.1.4, 3.2.2, 3.2.3, 3.4.3 |
| Automated Reasoning checks | "Provable", "mathematically verified" against written policy rules | Open-ended factuality → contextual grounding check | List only |
| **Model Evaluations** | Compare models or prompts, LLM-as-a-judge, human review, RAG evaluation | Bias on tabular ML models → SageMaker Clarify | 5.1.2, 3.4.2 |
| Fine-tuning, continued pre-training, distillation | Style or format the prompt cannot fix; smaller cheaper model with similar quality | Facts that change → RAG; first attempt → prompt engineering | 1.2.4 (SageMaker wording) |
| Custom Model Import | "Bring our own fine-tuned open-weights model, serverless, no endpoint to manage" | Full container and GPU control → SageMaker AI endpoint | List only |

Depth for each row: [Domain 1](../01-Domain1-FM-Integration-Data-Compliance.md), [Domain 2](../02-Domain2-Implementation-Integration.md), [Domain 3](../03-Domain3-Safety-Security-Governance.md), [Domain 4](../04-Domain4-Operational-Efficiency-Optimization.md), [Domain 5](../05-Domain5-Testing-Validation-Troubleshooting.md).

---

## 2. Amazon SageMaker AI

SageMaker AI is the answer when you must **host, train or govern a model yourself**. Bedrock is the answer when a managed model behind an API is enough. Model training itself is out of scope for the target candidate, so questions focus on deployment, lifecycle and governance.

| Capability | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|
| Real-time endpoint | Open-weights or proprietary model, GPU control, steady low-latency traffic, "hybrid" with Bedrock | Model is available on Bedrock and least overhead is asked → Bedrock | 2.2.1 |
| Asynchronous endpoint | Large payloads, long processing, queue in front, scale to zero | Sub-second responses → real-time endpoint | 2.2.1 |
| Serverless endpoint | Intermittent traffic, small CPU models, tolerate cold starts | Large language models or GPU → real-time endpoint | 2.2.1 |
| Batch transform | Offline scoring of a dataset in S3, no persistent endpoint | Bedrock model → Bedrock batch inference | Not named |
| Inference components with LoRA adapters | "Many fine-tuned variants on one base model", share a GPU, scale each variant | One model per endpoint is enough | 1.2.4 |
| Large model inference containers | "Container optimized for memory, GPU utilization and token throughput", model loading strategy | Small classic ML model | 2.2.2 |
| Endpoint deployment guardrails | Blue/green with canary or linear shifting and automatic rollback on an alarm | Bedrock agent or prompt rollout → aliases and versions | 1.2.4 |
| Production variants and shadow variants | A/B test or shadow test two models on live traffic | Compare FMs offline → Bedrock Model Evaluations | 5.1.2 |
| Request formatting for endpoints | "Structured data preparation for SageMaker AI endpoints" | Bedrock request → Converse message format | 1.3.3 |
| Model Cards | "Programmatic model cards", document intended use and limitations | Dataset lineage → Glue | 3.3.1, 3.4.3 |
| Pipelines | Automated retrain, evaluate, register, deploy with a quality gate | Generic application CI/CD → CodePipeline | 1.2.4 |
| Auto scaling for endpoints | Scale on concurrent requests per model or invocations per instance | Bedrock on-demand (no scaling knob; quotas and tiers) | 4.1.3 |

---

## 3. SageMaker named services

These nine appear by name on the in-scope list, plus Amazon Augmented AI.

| Service | What it is for | Exam trigger | Not this, use that | Guide |
|---|---|---|---|---|
| **Model Registry** | Version, approve and deploy model packages | "Versioning", "approval before deployment", "roll back to the previous model" | Prompt versions → Prompt Management | 1.2.4 |
| **Data Wrangler** | Visual data profiling, quality report, transforms | "Analysts inspect and clean a dataset without code" | Automated rules in an ETL job → Glue Data Quality | 1.3.1 |
| **Processing** | Run your own container or script as a managed batch job | "Custom libraries", heavy batch pre-processing or post-processing | Managed extraction from files → Data Automation | 1.3.2 |
| **JumpStart** | Catalog of open models with one-step deploy and fine-tune | "Deploy an open-source FM quickly", proof of concept on your own endpoint | Model offered serverless on Bedrock → Bedrock | List only |
| **Clarify** | Bias metrics and explainability (SHAP); also FM evaluation in SageMaker | "Detect bias across demographic groups", "explain feature attribution" | Judge quality of generated answers → Bedrock Model Evaluations | List only |
| **Model Monitor** | Scheduled checks on a SageMaker endpoint: data quality, model quality, bias drift, attribution drift | "Detect drift on a hosted model", needs data capture on the endpoint | Bedrock model metrics → CloudWatch and invocation logs | List only |
| **Ground Truth** | Human data labeling, build evaluation or fine-tuning datasets | "Label a dataset", "human-annotated ground truth for evaluation" | Review individual live predictions → Augmented AI | List only |
| **Neo** | Compile and optimize a model for specific hardware or edge devices | "Run on edge hardware", "optimize for a target device" | Large FM serving → large model inference containers | List only |
| **Unified Studio** | One workspace for data, analytics, ML and Bedrock app building | "Single governed environment for data and AI teams" | A service that runs inference | List only |
| **Augmented AI** (A2I) | Human review of live predictions with a managed worker UI and workforce | "Route low-confidence results to human reviewers" | A simple approve or reject step in a workflow → Step Functions callback | List only |

---

## 4. AWS Glue

| Feature | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|
| **Glue Data Quality** | Rule-based validation (completeness, uniqueness, freshness) before data reaches a model; fail the pipeline on a rule | Interactive profiling → Data Wrangler; single-record checks → Lambda | 1.3.1 |
| **Glue Data Catalog** | "Register data sources", central schema and metadata, traceability of what fed the application | Column or row permissions → Lake Formation (built on the catalog) | 3.3.2 |
| Glue jobs and lineage | "Automatically track data lineage", scheduled ETL that cleans and reshapes source data | Extraction from PDFs or media → Data Automation | 3.3.1 |
| Crawlers | Discover schema in S3 and populate the catalog | Find sensitive data in S3 → Macie | 3.3.2 |

Glue prepares and describes **structured and semi-structured data**. It does not embed, retrieve or call a model by itself.

---

## 5. Amazon Comprehend

Classic NLP on text you pass to it: entities, key phrases, language, sentiment, PII detection, toxicity, custom classification and custom entities. It detects; your code acts.

| Use it when the stem says | Use something else when | Guide |
|---|---|---|
| "Pre-processing filter" before the model; PII in text inside a pipeline; entities as metadata; intent classification at volume | Sensitive data at rest in S3 → Macie; filter on the model call → Guardrails; prompt injection → Guardrails prompt attack filter; generate or summarize → an FM | 1.3.4, 1.6.2, 3.1.4, 3.2.2, 3.2.3 |

Full feature table, when-to-use and when-not-to-use tables: [revisit topic 8](../REVISIT-Topics.md#8-amazon-comprehend-features-when-to-use-it-when-not-to).

---

## 6. Amazon Bedrock Data Automation

One managed API that turns documents, images, audio and video into structured JSON, with standard output or custom blueprints, confidence scores and visual grounding.

| Use it when the stem says | Use something else when | Guide |
|---|---|---|
| "Extract fields with least development effort", mixed document packet, low-confidence results to a human, knowledge base misses tables and charts, searchable recordings | Custom parsing prompt → FM parser; search by image → multimodal embeddings; live transcription → Transcribe streaming; strict single Region → in-Region service | 2.5.3 |

Full detail: [revisit topic 9](../REVISIT-Topics.md#9-amazon-bedrock-data-automation-bda-features-when-to-use-it-when-not-to).

---

## 7. Amazon Lex

Managed conversational interface for voice and text, built from intents, slots and fulfillment. On the in-scope list; not named in a skill statement.

| Feature | What it does | Exam trigger |
|---|---|---|
| Intents and slots | Deterministic task dialogue: recognize the intent, collect required values, confirm, fulfill through Lambda | "Collect booking details step by step", "structured transactional dialogue" |
| Voice and channels | Speech in and out, integrates with Amazon Connect and messaging channels | "Contact center IVR", "voice bot" |
| `AMAZON.QnAIntent` | Answers free-form questions with a Bedrock model over a knowledge store (Bedrock knowledge base, Kendra index or OpenSearch) | "Add generative FAQ answers to an existing bot" |
| Assisted slot resolution | Uses a Bedrock model to interpret slot values the built-in recognizer missed | "Users phrase answers in unexpected ways" |
| Descriptive bot builder and utterance generation | Generates a bot draft or sample utterances from a description | "Speed up bot design" |
| Session state | Session attributes and context carried across turns | "Maintain context across turns in a bot" |

| Use Lex when | Use something else when |
|---|---|
| The conversation is a guided task with required fields and a voice or chat channel | Open-ended reasoning with tool calls → Bedrock Agents or AgentCore |
| A contact center needs a bot in the call flow | Employees ask questions over company documents → Amazon Q Business |
| You want deterministic intent handling with a generative fallback | Only intent classification inside a pipeline → Comprehend custom classification (the guide's named choice in 1.6.2) |

---

## 8. Amazon Q and related assistants

| Service | What it is | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|---|
| **Amazon Q Developer** | AI assistant for building on AWS: code generation and refactoring, suggestions, tests, security scans, troubleshooting in the IDE, CLI and console | "Generate and refactor code", "API assistance", "AI component testing", "GenAI-specific error pattern recognition" | A runtime component of your application | 2.5.4, 2.5.6 |
| **Amazon Q Business** | Managed enterprise assistant over company data: 40+ connectors, permissions-aware answers through IAM Identity Center, plugins for actions, admin controls and guardrails | "Employees ask questions across SharePoint, Confluence and Salesforce with existing permissions respected", least build effort | Custom application, own prompts, model choice or customer-facing product → Bedrock Knowledge Bases or Agents | List only |
| **Amazon Q Business Apps** | Small shareable apps that users generate from a natural-language description inside Q Business | "Business users create a reusable task app without code" | Developer-built workflow → Flows or Step Functions | List only |
| **Kiro** | Agentic development environment: specs (requirements, design, tasks), steering files, agent hooks, MCP integrations; IDE and CLI | "Spec-driven development", "turn requirements into reviewed tasks and code" | Runtime agent hosting → AgentCore | List only |
| **Amazon Quick** | Agentic workspace for business users: chat agents, Quick Flows and Quick Automate for workflow automation, research, and Quick Sight for BI (AWS documentation now also uses the name AWS Quick) | "Business users automate a routine task or ask questions of dashboards in natural language" | Application you build and operate → Bedrock | List only |
| **Amazon Quick Sight** | Business intelligence with natural-language questions and generated summaries | "Dashboard of token cost and usage", "ask questions of data" | Operational metrics and alarms → CloudWatch; Grafana dashboards → Managed Grafana | List only |
| Amazon Connect with Amazon Q in Connect | Contact center with real-time agent assistance | "Suggest answers to human agents during a call" | General chatbot → Lex or Bedrock | List only (Connect) |

**The split that decides most Amazon Q questions:** Q Developer helps the people who **build**; Q Business helps the people who **work** in the company; Bedrock is what you use to build the **application** itself.

---

## 9. Other AI and ML services in scope

| Service | What it is for | Use it when the stem says | Use something else when | Guide |
|---|---|---|---|---|
| **Amazon Transcribe** | Speech to text: batch and streaming, speaker labels, custom vocabulary, PII redaction, call analytics | "Audio must be converted before the FM", "live captions" | Audio and video files into structured insights with one API → Data Automation | 1.3.2 |
| **Amazon Textract** | OCR and document structure: text, forms, tables, queries, signatures, expenses, IDs | "Only extracted text or key-value pairs are needed", existing OCR pipeline, synchronous page OCR | Custom fields with least effort across document types → Data Automation | List only |
| **Amazon Rekognition** | Image and video analysis: labels, moderation, text in images, faces | "Detect unsafe images before they reach the model", "label images" at low cost | Reason about an image or answer questions on it → multimodal FM; filter images on a Bedrock call → Guardrails | List only |
| **Amazon Kendra** | Managed enterprise search with connectors, access-control-aware results and a retriever for RAG | "Existing Kendra index", "document-level permissions in search results", many enterprise connectors | Vector RAG with least overhead → Bedrock Knowledge Bases (which can also sit on a Kendra index) | List only |
| **Amazon Augmented AI** | Human review loops | See [section 3](#3-sagemaker-named-services) | | List only |
| **Amazon Titan** | Amazon's first-party models on Bedrock | See [section 1](#1-amazon-bedrock) | | 1.5.2 |

---

## 10. Frameworks and protocols the guide names

These are not on the service list, but skill statements name them.

| Name | What it is | Exam trigger | Guide |
|---|---|---|---|
| **Strands Agents** | Open-source, model-driven agent SDK from AWS; deploy on AgentCore Runtime, Lambda or ECS | "AWS-native orchestration", "custom tool behaviors in code", multi-agent patterns | 2.1.1, 2.1.6, 2.5.5 |
| **AWS Agent Squad** | Open-source framework that classifies a request and routes it to the right specialist agent | "Route between several agents", multi-agent systems | 2.1.1, 2.5.5 |
| **Model Context Protocol** | Open standard for connecting agents to tools and data | "Standard agent-to-tool interface"; stateless server on Lambda, stateful or heavy server on ECS | 1.5.6, 2.1.1, 2.1.7 |
| OpenAPI specification | API-first definition; also the schema for agent action groups and AgentCore Gateway targets | "API-first development" | 2.5.2 |
| AWS Well-Architected Generative AI Lens | Design review questions for GenAI workloads in the Well-Architected Tool | "Standardized review of a GenAI architecture" | 1.1.3 |

Depth: [revisit topic 6](../REVISIT-Topics.md#6-when-to-use-strands-agents-vs-bedrock-flows-vs-step-functions-workflows) and [Domain 2](../02-Domain2-Implementation-Integration.md).

---

## 11. AI and ML services that are not in scope

**Explicitly out of scope** (from the guide's list): Amazon Forecast, Amazon Fraud Detector, Amazon Lookout for Equipment, Lookout for Metrics and Lookout for Vision, Amazon DevOps Guru, Amazon HealthLake, Amazon Monitron, AWS Panorama, AWS DeepRacer, AWS DeepComposer, Amazon CodeGuru. An option built on one of these is almost always a distractor.

**On neither list:** Amazon Translate, Amazon Polly, Amazon Personalize, Amazon Comprehend Medical. They can appear in an option, so know one line each, but do not spend study time on them.

| Service | One line |
|---|---|
| Amazon Translate | Machine translation of text and documents |
| Amazon Polly | Text to speech |
| Amazon Personalize | Recommendations from interaction data |
| Amazon Comprehend Medical | Medical entities and PHI in clinical text |

Other notable exclusions that tempt in architecture options: Amazon Redshift, AWS Batch, Amazon SES, AWS Budgets, AWS Direct Connect, Transit Gateway and VPN are all out of scope.

---

## 12. Supporting services in scope

One line each on the role they play around the model. The guide column shows where a skill statement names them.

**Vector stores and data**

| Service | Role in a GenAI application | Guide |
|---|---|---|
| Amazon OpenSearch Service | Vector, keyword and hybrid search at scale; neural plugin; sharding and multi-index design | 1.4.1, 1.4.3, 1.5.3, 1.5.4 |
| Amazon Aurora (pgvector) | Vectors next to relational data, SQL filters and joins | 1.5.3 |
| Amazon RDS | Relational store paired with S3 document repositories | 1.4.1 |
| Amazon DynamoDB and DynamoDB Streams | Conversation history, metadata, session state, circuit-breaker state; change capture | 1.4.1, 1.6.2 |
| Amazon S3 | Documents, object metadata, prompt template repository, logs, batch input and output | 1.4.1, 1.4.2, 1.6.3, 3.2.2 |
| Amazon Neptune | Graph relationships for GraphRAG | List only |
| Amazon DocumentDB, Amazon ElastiCache | Document store with vector search; response or semantic cache | List only |
| Amazon Athena, Amazon EMR, Amazon Kinesis, Amazon MSK, Amazon AppFlow | Query, large-scale processing, streaming ingestion, SaaS data ingestion | List only |

**Integration and compute**

| Service | Role | Guide |
|---|---|---|
| AWS Step Functions | Circuit breaker, agent loops, human approval, query transformation, document processing, routing | 1.2.3, 1.5.5, 1.6.2, 1.6.4, 2.1.2, 2.1.3, 2.1.5, 2.4.4, 2.5.3, 2.5.5, 3.1.1 |
| AWS Lambda | Glue code: validation, chunking, tool execution, routing, compliance checks | Named in 16 skills |
| Amazon API Gateway | Front door: validation, throttling, streaming, routing, response filtering | 1.2.2, 2.1.5, 2.3.2, 2.4.x, 2.5.1, 3.1.4 |
| AWS AppConfig | Switch model or prompt settings without redeploying; gradual rollout with rollback | 1.2.2 |
| Amazon EventBridge, SQS, SNS | Event-driven integration, buffering, notification | 2.3.2 |
| AWS Amplify, AWS AppSync | Front-end components and GraphQL or real-time APIs | 2.5.2 (Amplify) |
| Amazon ECS, EKS, Fargate, ECR, EC2, App Runner | Host stateful MCP servers, agents and self-managed models | List only |
| AWS Outposts, AWS Wavelength | Data residency on premises; ultra-low latency at the edge | 2.3.4 |

**Security and governance**

| Service | Role | Guide |
|---|---|---|
| IAM, IAM Identity Center, Amazon Cognito | Least privilege on model and tool access; workforce and customer identity | 2.1.3, 2.3.3, 3.2.1 |
| Amazon VPC and AWS PrivateLink | Private connectivity to Bedrock and SageMaker through interface endpoints | 3.2.1 |
| AWS Lake Formation | Column, row and tag-based access to data | 3.2.1 |
| Amazon Macie | Find sensitive data in S3 | 3.2.2 |
| AWS KMS, AWS Secrets Manager, AWS Encryption SDK | Encryption keys, secrets for tool credentials, client-side encryption | List only |
| AWS WAF | Rate limits and request filtering in front of the API | List only |
| AWS CloudTrail | API audit trail | 1.6.3, 3.3.2 |

**Operations and delivery**

| Service | Role | Guide |
|---|---|---|
| Amazon CloudWatch, CloudWatch Logs, CloudWatch Synthetics | Metrics, alarms, Logs Insights on prompts and responses, canaries | 1.3.1, 1.6.3, 1.6.4, 2.5.6, 3.2.1, 3.3.1, 3.4.1, 3.4.2, 4.3.2, 5.2.5 |
| AWS X-Ray | Trace FM API calls across service boundaries | 2.4.3, 2.5.6, 5.2.5 |
| AWS Cost Explorer, AWS Cost Anomaly Detection | Cost by tag; alert on unusual spend | List only |
| Amazon Managed Grafana | Dashboards over CloudWatch and other sources | List only |
| AWS Well-Architected Tool | Generative AI Lens reviews | 1.1.3 |
| CodePipeline, CodeBuild, CodeDeploy, CloudFormation, CDK | CI/CD with evaluation gates; infrastructure as code | List only |
| AWS Service Catalog, AWS Systems Manager, AWS Chatbot, AWS Auto Scaling | Approved templates, parameters and operations, chat notifications, scaling | List only |

---

## 13. Look-alike services and how to split them

| Pair | The deciding question | Answer |
|---|---|---|
| Bedrock vs SageMaker AI | Is a managed model behind an API enough? | Yes → Bedrock. Need your own container, weights or GPU control → SageMaker AI |
| Knowledge Bases vs Kendra vs Q Business | Who builds the experience? | You build a custom app → Knowledge Bases. You need enterprise search with ACL-aware results → Kendra. You want a finished employee assistant → Q Business |
| Bedrock Agents vs AgentCore | Managed configuration or your own code? | Configuration, least code → Bedrock Agents. Any framework, isolation, identity, gateway → AgentCore |
| Flows vs Step Functions vs agents | Is the path known in advance? | Known prompt chain → Flows. Known workflow with retries, waits, approvals → Step Functions. Model decides the path → agent |
| Lex vs Bedrock Agents | Guided task dialogue or open reasoning? | Slots and intents, voice channel → Lex. Reasoning and tool selection → agent |
| Textract vs Data Automation | Raw OCR or business fields? | Text, forms, tables → Textract. Named fields across document types, least effort → Data Automation |
| Transcribe vs Data Automation | Stream or file; text or insights? | Live audio or plain transcript → Transcribe. Summaries, scenes, structured output from media files → Data Automation |
| Rekognition vs multimodal FM vs Guardrails | Label, reason or filter? | Fixed labels and moderation at scale → Rekognition. Describe or answer about the image → multimodal FM. Block harmful images on a Bedrock call → Guardrails |
| Comprehend vs Guardrails vs Macie | Where is the sensitive text? | In your pipeline → Comprehend. In the prompt or response of a model call → Guardrails. At rest in S3 → Macie |
| Clarify vs Bedrock Model Evaluations | Classic ML bias or generated-text quality? | Bias and explainability on a model you host → Clarify. Quality, faithfulness, toxicity of FM output → Model Evaluations |
| Model Monitor vs CloudWatch | SageMaker endpoint or Bedrock model? | Drift on a hosted endpoint with data capture → Model Monitor. Bedrock tokens, latency, errors → CloudWatch and invocation logs |
| Ground Truth vs Augmented AI | Build a dataset or review live output? | Label data in bulk → Ground Truth. Review individual predictions in production → Augmented AI |
| Glue Data Quality vs Data Wrangler | Automated rules or interactive exploration? | Rules in a pipeline → Glue Data Quality. Visual profiling by an analyst → Data Wrangler |
| Glue Data Catalog vs Lake Formation | Describe the data or control access to it? | Register and track sources → Data Catalog. Column, row, tag permissions → Lake Formation |
| Model Registry vs Prompt Management | What is being versioned? | Model packages → Model Registry. Prompt templates → Prompt Management |
| Q Developer vs Kiro | Assistant or spec-driven agentic IDE? | Suggestions, refactoring, troubleshooting → Q Developer. Specs, steering, hooks → Kiro |
| CloudTrail vs model invocation logging | Who called, or what was said? | API identity and time → CloudTrail. Prompt and response content → invocation logging |
| Provisioned Throughput vs cross-Region inference vs batch | What is the traffic shape? | Steady and high → Provisioned Throughput. Bursty and throttled → cross-Region inference. Not urgent → batch |

---

## Sources

- [In-scope AWS services](https://docs.aws.amazon.com/aws-certification/latest/ai-professional-01/aip-01-in-scope-services.html)
- [Out-of-scope AWS services](https://docs.aws.amazon.com/aws-certification/latest/ai-professional-01/aip-01-out-of-scope-services.html)
- [Exam guide overview and domains](https://docs.aws.amazon.com/aws-certification/latest/ai-professional-01/ai-professional-01.html)
- [Amazon Lex V2 generative AI features](https://docs.aws.amazon.com/lexv2/latest/dg/latest-features.html)
- [Amazon Q Business admin controls and guardrails](https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/guardrails-concepts.html)
