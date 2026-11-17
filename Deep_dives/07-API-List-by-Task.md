# AIP-C01 — API List: Which API Does Which Job

The API operations behind each exam task, grouped by service and then by exam task. Service selection rules are in the [service list](06-Service-List-When-to-Use.md).

Back to the [index](../INDEX.md).

**How the exam uses API names**

- It does not ask for request syntax. It uses API names to make options differ: `Retrieve` against `RetrieveAndGenerate`, `Converse` against `InvokeModel`, `ApplyGuardrail` against a guardrail attached to the call.
- Know three things per API: **what job it does**, **which client it belongs to** (build time or run time), and **the one parameter that changes behavior**.
- The **Client** column is the SDK client and CLI namespace (for example `bedrock-runtime`). A wrong-client option ("call `InvokeAgent` on the `bedrock` client") is a quiet distractor.
- Every operation name and parameter in this file was checked against the AWS SDK service models (botocore 1.43.90) on 2026-11-17.

## Contents

- [1. Client map: build time and run time](#1-client-map-build-time-and-run-time)
- [2. Bedrock model inference](#2-bedrock-model-inference)
- [3. Bedrock models, capacity, guardrails and evaluation](#3-bedrock-models-capacity-guardrails-and-evaluation)
- [4. Knowledge Bases, prompts, flows and agents at build time](#4-knowledge-bases-prompts-flows-and-agents-at-build-time)
- [5. Retrieval, agents and flows at run time](#5-retrieval-agents-and-flows-at-run-time)
- [6. Amazon Bedrock AgentCore](#6-amazon-bedrock-agentcore)
- [7. Amazon Bedrock Data Automation](#7-amazon-bedrock-data-automation)
- [8. Amazon SageMaker AI and named services](#8-amazon-sagemaker-ai-and-named-services)
- [9. AWS Glue and Lake Formation](#9-aws-glue-and-lake-formation)
- [10. Amazon Comprehend](#10-amazon-comprehend)
- [11. Amazon Lex](#11-amazon-lex)
- [12. Amazon Q](#12-amazon-q)
- [13. Kendra, Textract, Transcribe, Rekognition, Augmented AI](#13-kendra-textract-transcribe-rekognition-augmented-ai)
- [14. Vector stores](#14-vector-stores)
- [15. Supporting services](#15-supporting-services)
- [16. API chains by exam task](#16-api-chains-by-exam-task)
- [17. API pairs the exam contrasts](#17-api-pairs-the-exam-contrasts)
- [18. IAM actions and condition keys](#18-iam-actions-and-condition-keys)
- [Sources](#sources)

---

## 1. Client map: build time and run time

| Client | Plane | What lives there |
|---|---|---|
| `bedrock` | Control | Model catalog, customization jobs, Provisioned Throughput, guardrails, evaluation jobs, batch jobs, inference profiles, prompt routers, invocation logging |
| `bedrock-runtime` | Data | Model inference, `ApplyGuardrail`, token counting |
| `bedrock-agent` | Control | Knowledge bases, data sources, ingestion, agents, action groups, aliases, prompts, flows |
| `bedrock-agent-runtime` | Data | `Retrieve`, `RetrieveAndGenerate`, `InvokeAgent`, `InvokeFlow`, `Rerank`, sessions |
| `bedrock-data-automation` | Control | Projects and blueprints |
| `bedrock-data-automation-runtime` | Data | Invoke and track Data Automation jobs |
| `bedrock-agentcore-control` | Control | Agent runtimes, gateways, memory stores, credential providers, policies |
| `bedrock-agentcore` | Data | Invoke agent runtime, memory events and records, tokens, code interpreter, browser |
| `sagemaker` | Control | Models, endpoints, jobs, registry, cards, pipelines, monitoring |
| `sagemaker-runtime` | Data | Invoke endpoints |
| `sagemaker-a2i-runtime` | Data | Human review loops |

---

## 2. Bedrock model inference

| Task | API | Client | Exam note |
|---|---|---|---|
| Chat or text generation with one request shape for all models | `Converse` | `bedrock-runtime` | Preferred default. Carries `system`, `messages`, `inferenceConfig`, `toolConfig`, `guardrailConfig`. Switching models means changing `modelId` only |
| Same, streamed token by token | `ConverseStream` | `bedrock-runtime` | Fix for "users wait, then see the whole answer". Needs a streaming path to the client |
| Model-native request body | `InvokeModel` | `bedrock-runtime` | Embeddings, image generation, provider-specific fields. Body format differs per model |
| Model-native, streamed | `InvokeModelWithResponseStream` | `bedrock-runtime` | Streaming for models or features outside the unified API |
| Speech-to-speech, two-way stream | `InvokeModelWithBidirectionalStream` | `bedrock-runtime` | Real-time voice models |
| Long-running generation with output in S3 | `StartAsyncInvoke`, `GetAsyncInvoke` | `bedrock-runtime` | Video generation and other long jobs; not the same as batch inference |
| Check a text against a guardrail with no model call | `ApplyGuardrail` | `bedrock-runtime` | `source` is `INPUT` or `OUTPUT`. Use for SageMaker or third-party models, or to check input before retrieval |
| Count tokens before sending | `CountTokens` | `bedrock-runtime` | Cost estimate and context-window check without paying for inference |

**Converse details that decide questions**

| Field | Values and meaning |
|---|---|
| `modelId` | Model ID, **inference profile** ID or ARN (cross-Region or application), Provisioned Throughput ARN, **prompt ARN** from Prompt Management, or **prompt router** ARN |
| `inferenceConfig` | `maxTokens`, `temperature`, `topP`, `stopSequences`. Model-specific extras go in `additionalModelRequestFields` |
| `toolConfig` | Tool definitions with a JSON schema, plus `toolChoice`. The model returns a `toolUse` block; your code runs the tool and sends back a `toolResult` |
| `guardrailConfig` | `guardrailIdentifier`, `guardrailVersion`, `trace` |
| `promptVariables` | Fills the variables of a managed prompt when `modelId` is a prompt ARN |
| `performanceConfig`, `serviceTier` | Latency-optimized inference; Priority, Standard or Flex tier |
| `stopReason` (response) | `end_turn`, `tool_use`, `max_tokens`, `stop_sequence`, `guardrail_intervened`, `content_filtered`, `model_context_window_exceeded` |
| `usage` (response) | `inputTokens`, `outputTokens`, `totalTokens`, `cacheReadInputTokens`, `cacheWriteInputTokens` |

- **Truncated answer** means `stopReason` is `max_tokens`: raise `maxTokens` or shorten the task. It is not a throttling problem.
- **Prompt caching** is a `cachePoint` block placed after the stable prefix in `system`, `messages` or `tools`. Proof that it works is `cacheReadInputTokens` above zero.
- **Structured output** is enforced with a JSON schema: `outputConfig` with a `json_schema` text format on models that support it, or a tool with a JSON schema forced through `toolChoice`.

---

## 3. Bedrock models, capacity, guardrails and evaluation

| Task | API | Client | Exam note |
|---|---|---|---|
| List models and their capabilities | `ListFoundationModels`, `GetFoundationModel` | `bedrock` | Filter by provider, modality, customization and inference type |
| Check that a model is usable in this account and Region | `GetFoundationModelAvailability` | `bedrock` | First check for an access error on a new model |
| List cross-Region inference profiles | `ListInferenceProfiles`, `GetInferenceProfile` | `bedrock` | The profile ID is what you pass as `modelId` |
| Create an application inference profile | `CreateInferenceProfile` | `bedrock` | Tag it per team or application for **cost allocation** and usage metrics |
| Fine-tune, continue pre-training or distill | `CreateModelCustomizationJob`, `GetModelCustomizationJob` | `bedrock` | `customizationType`: `FINE_TUNING`, `CONTINUED_PRE_TRAINING`, `DISTILLATION`. Training data is JSONL in S3; needs a service role |
| Serve a custom model on demand | `CreateCustomModelDeployment` | `bedrock` | On-demand deployment for supported custom models |
| Buy dedicated capacity | `CreateProvisionedModelThroughput` | `bedrock` | `modelUnits` and optional `commitmentDuration` (`OneMonth`, `SixMonths`; omit for no commitment). Invoke with the provisioned model ARN |
| Import your own model weights | `CreateModelImportJob` | `bedrock` | Custom Model Import; serverless serving of your weights |
| Copy a custom model to another Region | `CreateModelCopyJob` | `bedrock` | Part of a multi-Region plan for custom models |
| Run batch inference | `CreateModelInvocationJob`, `GetModelInvocationJob` | `bedrock` | JSONL input and output in S3. Cheapest path for non-urgent bulk work |
| Create a prompt router | `CreatePromptRouter`, `GetPromptRouter` | `bedrock` | `models`, `fallbackModel`, `routingCriteria` (response quality difference). Invoke by passing the router ARN as `modelId` |
| Create and version a guardrail | `CreateGuardrail`, `CreateGuardrailVersion`, `UpdateGuardrail` | `bedrock` | Policies: content, topic, word, sensitive information, contextual grounding, automated reasoning. `DRAFT` is editable; numbered versions are immutable |
| Enforce a guardrail account-wide | `PutEnforcedGuardrailConfiguration` | `bedrock` | Central enforcement instead of trusting each caller |
| Create an Automated Reasoning policy | `CreateAutomatedReasoningPolicy` | `bedrock` | Formal rules extracted from a policy document; attached through a guardrail |
| Run an evaluation job | `CreateEvaluationJob`, `GetEvaluationJob` | `bedrock` | `applicationType`: `ModelEvaluation` or `RagEvaluation`. Automatic metrics, LLM-as-a-judge, human workers, or your own inference responses |
| Turn on prompt and response logging | `PutModelInvocationLoggingConfiguration` | `bedrock` | Destinations: CloudWatch Logs, S3, or both. Separate switches for text, image, embedding data. Account and Region level |

---

## 4. Knowledge Bases, prompts, flows and agents at build time

| Task | API | Client | Exam note |
|---|---|---|---|
| Create a knowledge base | `CreateKnowledgeBase` | `bedrock-agent` | Type `VECTOR`, `KENDRA` or `SQL` (structured data). Vector stores include OpenSearch Serverless, OpenSearch managed cluster, Aurora (`RDS`), Neptune Analytics, S3 Vectors |
| Connect a data source | `CreateDataSource` | `bedrock-agent` | Source types: S3, web crawler, Confluence, SharePoint, Salesforce, custom. Sets **chunking** (`FIXED_SIZE`, `HIERARCHICAL`, `SEMANTIC`, `NONE`), **parsing** (Data Automation or foundation model) and custom transformation Lambda |
| Sync content | `StartIngestionJob`, `GetIngestionJob` | `bedrock-agent` | Sync is incremental: only added, changed and deleted objects. `GetIngestionJob` returns statistics and failure reasons |
| Add or remove documents without a full sync | `IngestKnowledgeBaseDocuments`, `DeleteKnowledgeBaseDocuments` | `bedrock-agent` | Direct ingestion for near-real-time updates and custom sources |
| Inspect what was ingested | `ListKnowledgeBaseDocuments`, `GetKnowledgeBaseDocuments` | `bedrock-agent` | Document-level status when "some documents never appear" |
| Create and version a prompt | `CreatePrompt`, `CreatePromptVersion`, `GetPrompt` | `bedrock-agent` | Variables, variants, model and inference settings stored with the prompt. A version is an immutable snapshot |
| Build a flow | `CreateFlow`, `ValidateFlowDefinition`, `PrepareFlow` | `bedrock-agent` | Prepare before testing the working draft |
| Publish a flow | `CreateFlowVersion`, `CreateFlowAlias` | `bedrock-agent` | The alias points at a version; moving the alias is the rollback |
| Create an agent | `CreateAgent` | `bedrock-agent` | `instruction`, `foundationModel`, `guardrailConfiguration`, `memoryConfiguration`, `promptOverrideConfiguration`, `agentCollaboration` (`SUPERVISOR`, `SUPERVISOR_ROUTER`) |
| Add tools | `CreateAgentActionGroup` | `bedrock-agent` | Schema: `apiSchema` (OpenAPI) or `functionSchema`. Executor: `lambda` or `customControl` (**return of control**). Built-in signatures: `AMAZON.UserInput`, `AMAZON.CodeInterpreter` |
| Attach a knowledge base | `AssociateAgentKnowledgeBase` | `bedrock-agent` | The description tells the agent when to search it |
| Add a collaborator agent | `AssociateAgentCollaborator` | `bedrock-agent` | Multi-agent collaboration under a supervisor |
| Make the draft testable | `PrepareAgent` | `bedrock-agent` | Required after every change to the draft. Forgetting it explains "my change has no effect" |
| Publish an agent | `CreateAgentAlias`, `UpdateAgentAlias` | `bedrock-agent` | Creating an alias snapshots a version. Point the alias back to the previous version to roll back |

---

## 5. Retrieval, agents and flows at run time

| Task | API | Client | Exam note |
|---|---|---|---|
| Retrieve chunks only | `Retrieve` | `bedrock-agent-runtime` | You build the prompt and call the model. Use for custom prompts, your own post-processing, or a non-Bedrock generator |
| Retrieve and generate with citations | `RetrieveAndGenerate` | `bedrock-agent-runtime` | Least code. Manages the session with `sessionId`. Can also answer over a document passed in the request with no knowledge base |
| Same, streamed | `RetrieveAndGenerateStream` | `bedrock-agent-runtime` | Streaming RAG answers |
| Rerank any list of passages | `Rerank` | `bedrock-agent-runtime` | Reranker model for results that did not come from a knowledge base |
| Natural language to SQL | `GenerateQuery` | `bedrock-agent-runtime` | Structured-data knowledge bases |
| Invoke an agent | `InvokeAgent` | `bedrock-agent-runtime` | `agentId`, `agentAliasId`, `sessionId`, `enableTrace`, `sessionState`, `memoryId`. Response is a stream |
| Invoke an agent defined in the request | `InvokeInlineAgent` | `bedrock-agent-runtime` | Agent configured per call; nothing to create or prepare |
| Read or clear agent memory | `GetAgentMemory`, `DeleteAgentMemory` | `bedrock-agent-runtime` | Session summaries kept across sessions per `memoryId` |
| Run a flow | `InvokeFlow` | `bedrock-agent-runtime` | Takes the flow ID and alias ID |
| Rewrite a prompt for a target model | `OptimizePrompt` | `bedrock-agent-runtime` | Prompt optimization |
| Store conversation state for a framework agent | `CreateSession`, `CreateInvocation`, `PutInvocationStep` | `bedrock-agent-runtime` | Session management for agents built with open-source frameworks |

**Retrieval settings to recognize** (inside `retrievalConfiguration.vectorSearchConfiguration`)

| Setting | Effect |
|---|---|
| `numberOfResults` | How many chunks come back. More recall, more tokens |
| `overrideSearchType` | `HYBRID` or `SEMANTIC`. Hybrid helps with product codes, names and exact terms |
| `filter` | Metadata filter applied before ranking: precision and tenant isolation |
| `implicitFilterConfiguration` | Model derives the filter from the query and your metadata schema |
| `rerankingConfiguration` | Second-stage reranker model over the retrieved set |
| `orchestrationConfiguration.queryTransformationConfiguration` | Query decomposition for compound questions (`RetrieveAndGenerate`) |
| `generationConfiguration` | Custom prompt template, guardrail, inference settings for the answer step |

**Agent session state to recognize** (inside `sessionState` on `InvokeAgent`)

| Field | Effect |
|---|---|
| `sessionAttributes` | Persist for the session and reach the action group Lambda: user ID, tenant, authorization context |
| `promptSessionAttributes` | Injected into the prompt for one turn: date, locale |
| `returnControlInvocationResults` | Your application's result for an action the agent handed back through return of control |
| `knowledgeBaseConfigurations` | Per-call retrieval settings and metadata filters for an attached knowledge base |

---

## 6. Amazon Bedrock AgentCore

| Task | API | Client | Exam note |
|---|---|---|---|
| Host an agent built with any framework | `CreateAgentRuntime`, `CreateAgentRuntimeEndpoint` | `bedrock-agentcore-control` | Artifact, role, network mode, inbound authorizer, protocol |
| Call the hosted agent | `InvokeAgentRuntime` | `bedrock-agentcore` | `runtimeSessionId` gives each session its own isolated environment; `qualifier` selects the endpoint |
| End a session | `StopRuntimeSession` | `bedrock-agentcore` | Releases the session's compute |
| Expose APIs as MCP tools | `CreateGateway`, `CreateGatewayTarget` | `bedrock-agentcore-control` | Gateway speaks MCP; inbound auth is `CUSTOM_JWT` or `AWS_IAM`. Targets: Lambda, OpenAPI schema, Smithy model, existing MCP server, API Gateway |
| Create a memory store | `CreateMemory` | `bedrock-agentcore-control` | Event expiry for short-term memory; strategies extract long-term records |
| Write conversation turns | `CreateEvent`, `ListEvents` | `bedrock-agentcore` | Short-term memory per `actorId` and `sessionId` |
| Search long-term memory | `RetrieveMemoryRecords` | `bedrock-agentcore` | Semantic search within a namespace |
| Give the agent an identity | `CreateWorkloadIdentity` | `bedrock-agentcore-control` | Identity for the agent itself |
| Store outbound credentials | `CreateOauth2CredentialProvider`, `CreateApiKeyCredentialProvider` | `bedrock-agentcore-control` | Token vault for calling third-party tools |
| Get tokens at run time | `GetWorkloadAccessToken`, `GetResourceOauth2Token`, `GetResourceApiKey` | `bedrock-agentcore` | Act on behalf of a user without putting secrets in the prompt |
| Run generated code safely | `StartCodeInterpreterSession`, `InvokeCodeInterpreter` | `bedrock-agentcore` | Sandboxed code execution |
| Drive a web browser | `StartBrowserSession` | `bedrock-agentcore` | Managed browser tool |
| Restrict what tools may do | `CreatePolicyEngine`, `CreatePolicy` | `bedrock-agentcore-control` | Deterministic policy on tool calls at the gateway, outside the model |
| Evaluate agent quality | `CreateEvaluator`, `CreateOnlineEvaluationConfig`, `Evaluate` | `bedrock-agentcore-control`, `bedrock-agentcore` | Built-in and custom evaluators on live traces |

---

## 7. Amazon Bedrock Data Automation

| Task | API | Client | Exam note |
|---|---|---|---|
| Define what to extract | `CreateBlueprint`, `CreateBlueprintVersion` | `bedrock-data-automation` | Fields with instructions; versions are immutable |
| Group settings for all modalities | `CreateDataAutomationProject` | `bedrock-data-automation` | Standard output settings plus blueprints; `LIVE` and `DEVELOPMENT` stages |
| Process a file from S3 | `InvokeDataAutomationAsync` | `bedrock-data-automation-runtime` | Needs the project ARN and a **data automation profile ARN** (cross-Region inference). Output goes to S3 |
| Track the job | `GetDataAutomationStatus` | `bedrock-data-automation-runtime` | Or receive the EventBridge job event instead of polling |
| Process a small input inline | `InvokeDataAutomation` | `bedrock-data-automation-runtime` | Synchronous; images and short documents |
| Improve a blueprint from examples | `InvokeBlueprintOptimizationAsync` | `bedrock-data-automation` | Uses sample documents with ground truth |

---

## 8. Amazon SageMaker AI and named services

| Task | API | Client | Exam note |
|---|---|---|---|
| Register a model for hosting | `CreateModel` | `sagemaker` | Container image plus model artifacts and role |
| Define how to host it | `CreateEndpointConfig` | `sagemaker` | `ProductionVariants` (A/B weights), `ShadowProductionVariants`, `AsyncInferenceConfig`, serverless config, `DataCaptureConfig` |
| Create or change an endpoint | `CreateEndpoint`, `UpdateEndpoint` | `sagemaker` | `DeploymentConfig`: blue/green with canary or linear shifting, rolling update, **auto-rollback** on a CloudWatch alarm |
| Shift traffic between variants | `UpdateEndpointWeightsAndCapacities` | `sagemaker` | A/B test without a new deployment |
| Host a base model and LoRA adapters on one endpoint | `CreateInferenceComponent` | `sagemaker` | One component for the base model, one per adapter; each scales independently |
| Call an endpoint | `InvokeEndpoint` | `sagemaker-runtime` | `InferenceComponentName` picks the adapter; `TargetVariant` picks the variant |
| Call an endpoint, streamed | `InvokeEndpointWithResponseStream` | `sagemaker-runtime` | Token streaming from a self-hosted model |
| Queue a long or large request | `InvokeEndpointAsync` | `sagemaker-runtime` | Input from S3 (`InputLocation`), output to S3, SNS notification, scale to zero |
| Score a dataset offline | `CreateTransformJob` | `sagemaker` | Batch transform; no persistent endpoint |
| Run custom data processing | `CreateProcessingJob` | `sagemaker` | **SageMaker Processing**; also how Clarify and Data Wrangler flows run |
| Fine-tune or train | `CreateTrainingJob` | `sagemaker` | Out of scope in depth; know that it produces the model artifact |
| Version models | `CreateModelPackageGroup`, `CreateModelPackage` | `sagemaker` | **Model Registry** |
| Approve a version | `UpdateModelPackage` | `sagemaker` | `ModelApprovalStatus`: `Approved`, `Rejected`, `PendingManualApproval`. The status change emits an event that starts deployment |
| Document a model | `CreateModelCard`, `CreateModelCardExportJob` | `sagemaker` | "Programmatic model cards" (Skill 3.3.1) |
| Automate the lifecycle | `CreatePipeline`, `StartPipelineExecution` | `sagemaker` | Evaluate, register, deploy with a condition step as the gate |
| Monitor a hosted model | `CreateMonitoringSchedule` | `sagemaker` | **Model Monitor**; needs data capture on the endpoint |
| Define monitor jobs | `CreateDataQualityJobDefinition`, `CreateModelQualityJobDefinition`, `CreateModelBiasJobDefinition`, `CreateModelExplainabilityJobDefinition` | `sagemaker` | Bias and explainability definitions are **Clarify** |
| Label data | `CreateLabelingJob`, `CreateWorkteam` | `sagemaker` | **Ground Truth** |
| Set up human review | `CreateHumanTaskUi`, `CreateFlowDefinition` | `sagemaker` | **Augmented AI** review template and workflow |
| Send an item to reviewers | `StartHumanLoop` | `sagemaker-a2i-runtime` | Called by your code when confidence is below a threshold |
| Browse or curate the model catalog | `ListHubContents`, `DescribeHubContent` | `sagemaker` | **JumpStart**; deployment is usually done through the SDK, which calls the model and endpoint APIs above |
| Optimize a model for inference | `CreateOptimizationJob` | `sagemaker` | Quantization and compilation for large models |
| Compile for a target device | `CreateCompilationJob` | `sagemaker` | **Neo** |
| Find the right instance type | `CreateInferenceRecommendationsJob` | `sagemaker` | Inference Recommender load tests |
| Scale an endpoint | `RegisterScalableTarget`, `PutScalingPolicy` | `application-autoscaling` | Target tracking on concurrent requests or invocations per instance |

---

## 9. AWS Glue and Lake Formation

| Task | API | Client | Exam note |
|---|---|---|---|
| Define data quality rules | `CreateDataQualityRuleset` | `glue` | Rules are written in DQDL: completeness, uniqueness, freshness, custom SQL |
| Let Glue suggest rules | `StartDataQualityRuleRecommendationRun` | `glue` | Profiles the data and proposes a ruleset |
| Evaluate a dataset | `StartDataQualityRulesetEvaluationRun`, `GetDataQualityResult` | `glue` | Gate ingestion on the result; results also go to CloudWatch and EventBridge |
| Discover schema and register sources | `CreateCrawler`, `StartCrawler` | `glue` | Populates the Data Catalog (Skill 3.3.2) |
| Register or read a source by hand | `CreateDatabase`, `CreateTable`, `GetTable` | `glue` | Data Catalog entries with custom parameters for source attribution |
| Run ETL | `CreateJob`, `StartJobRun`, `GetJobRun` | `glue` | Clean and reshape before embedding or fine-tuning |
| Chain steps | `CreateWorkflow`, `CreateTrigger`, `StartWorkflowRun` | `glue` | Scheduled or event-driven refresh |
| Grant column or row access | `GrantPermissions`, `CreateDataCellsFilter` | `lakeformation` | Fine-grained access on catalog tables (Skill 3.2.1) |
| Tag-based access | `CreateLFTag`, `AddLFTagsToResource` | `lakeformation` | Scale permissions by classification |

---

## 10. Amazon Comprehend

| Task | API | Client | Exam note |
|---|---|---|---|
| Extract entities | `DetectEntities` | `comprehend` | Metadata for filtering (Skill 1.3.4) |
| Extract key phrases | `DetectKeyPhrases` | `comprehend` | Tags and metadata |
| Detect the language | `DetectDominantLanguage` | `comprehend` | Route to a language-specific prompt or index |
| Sentiment | `DetectSentiment`, `DetectTargetedSentiment` | `comprehend` | Escalation and business metrics |
| Find PII with offsets | `DetectPiiEntities` | `comprehend` | Real time; returns type, score and character offsets. Your code masks |
| Check whether PII is present | `ContainsPiiEntities` | `comprehend` | Labels only; cheap yes-or-no gate |
| Redact PII in bulk | `StartPiiEntitiesDetectionJob` | `comprehend` | Asynchronous over S3. `Mode`: `ONLY_REDACTION` or `ONLY_OFFSETS` |
| Score toxicity | `DetectToxicContent` | `comprehend` | Per-category scores; English only |
| Classify with your own labels | `CreateDocumentClassifier`, `CreateEndpoint`, `ClassifyDocument` | `comprehend` | Intent recognition (Skill 1.6.2). Real-time use needs an endpoint |
| Detect your own entity types | `CreateEntityRecognizer` | `comprehend` | Domain terms such as part numbers |
| Analyze many documents | `BatchDetectEntities`, `StartEntitiesDetectionJob`, `StartDocumentClassificationJob` | `comprehend` | Batch call for small sets; asynchronous job for S3 scale |
| Keep a custom model current | `CreateFlywheel`, `StartFlywheelIteration` | `comprehend` | Managed retraining loop |

---

## 11. Amazon Lex

| Task | API | Client | Exam note |
|---|---|---|---|
| Send user text to a bot | `RecognizeText` | `lexv2-runtime` | Returns intent, slots, messages and session state |
| Send audio | `RecognizeUtterance` | `lexv2-runtime` | Speech in, speech or text out |
| Stream a voice conversation | `StartConversation` | `lexv2-runtime` | Bidirectional streaming |
| Read or set conversation context | `GetSession`, `PutSession`, `DeleteSession` | `lexv2-runtime` | Session attributes carry context across turns |
| Define the bot | `CreateBot`, `CreateBotLocale`, `CreateIntent`, `CreateSlotType`, `CreateSlot` | `lexv2-models` | The built-in `AMAZON.QnAIntent` is added as an intent and points at a knowledge store and a Bedrock model |
| Build and publish | `BuildBotLocale`, `CreateBotVersion`, `CreateBotAlias` | `lexv2-models` | The alias is what applications and Amazon Connect call |
| Generate bot elements with a model | `GenerateBotElement`, `StartBotResourceGeneration` | `lexv2-models` | Utterance generation and descriptive bot builder |

---

## 12. Amazon Q

| Task | API | Client | Exam note |
|---|---|---|---|
| Create the assistant | `CreateApplication` | `qbusiness` | Tied to IAM Identity Center for user identity |
| Create its index and retriever | `CreateIndex`, `CreateRetriever` | `qbusiness` | Retriever type: native index or an existing **Kendra** index |
| Connect and sync enterprise data | `CreateDataSource`, `StartDataSourceSyncJob` | `qbusiness` | Connectors carry document permissions |
| Push documents directly | `BatchPutDocument` | `qbusiness` | Custom sources without a connector |
| Ask a question | `ChatSync`, `Chat` | `qbusiness` | `ChatSync` returns the whole answer; `Chat` streams. Answers respect the user's document access |
| Let the assistant act in other systems | `CreatePlugin` | `qbusiness` | Built-in and custom (OpenAPI) plugins |
| Set admin controls | `UpdateChatControlsConfiguration` | `qbusiness` | Blocked phrases, topic rules, enterprise-data-only responses |
| Collect feedback | `PutFeedback` | `qbusiness` | Thumbs up or down on a message |
| Publish the web experience | `CreateWebExperience` | `qbusiness` | Managed chat UI |
| Create a Q App | `CreateQApp`, `PredictQApp` | `qapps` | Generate an app definition from a description |

Amazon Q Developer and Kiro are used through the IDE, the CLI and the console. The exam tests **what they are for** (Skills 2.5.4 and 2.5.6), not an API.

---

## 13. Kendra, Textract, Transcribe, Rekognition, Augmented AI

| Task | API | Client | Exam note |
|---|---|---|---|
| Create and fill a search index | `CreateIndex`, `CreateDataSource`, `StartDataSourceSyncJob`, `BatchPutDocument` | `kendra` | Connectors bring access control lists with the documents |
| Search for end users | `Query` | `kendra` | Ranked results, extracted answers, FAQ matches |
| Fetch passages for RAG | `Retrieve` | `kendra` | Longer semantically relevant passages meant for an FM prompt. Same name as the Bedrock API, different client |
| Extract plain text | `DetectDocumentText` | `textract` | Lines and words only |
| Extract structure | `AnalyzeDocument` | `textract` | `FeatureTypes`: `TABLES`, `FORMS`, `QUERIES`, `SIGNATURES`, `LAYOUT` |
| Invoices, receipts, identity documents | `AnalyzeExpense`, `AnalyzeID` | `textract` | Purpose-built extractors |
| Multi-page PDFs | `StartDocumentAnalysis`, `GetDocumentAnalysis`, `StartDocumentTextDetection` | `textract` | Asynchronous with an SNS completion notice |
| Transcribe a recording | `StartTranscriptionJob`, `GetTranscriptionJob` | `transcribe` | Options: speaker labels, custom vocabulary, `ContentRedaction` for PII, language identification, toxicity detection |
| Transcribe live audio | `StartStreamTranscription` | Transcribe streaming API | HTTP/2 or WebSocket; live captions and voice front ends |
| Analyze contact center calls | `StartCallAnalyticsJob` | `transcribe` | Sentiment, categories, redaction |
| Improve domain terms | `CreateVocabulary`, `CreateVocabularyFilter` | `transcribe` | Custom vocabulary; word filtering |
| Label an image | `DetectLabels` | `rekognition` | Objects, scenes, concepts |
| Moderate an image | `DetectModerationLabels` | `rekognition` | Screen uploads before they reach a model or a user |
| Read text in an image | `DetectText` | `rekognition` | Short text in photos; documents go to Textract |
| Analyze stored video | `StartLabelDetection`, `StartContentModeration`, `GetLabelDetection` | `rekognition` | Asynchronous video analysis |
| Human review of a prediction | `StartHumanLoop` | `sagemaker-a2i-runtime` | Flow and UI are created on the `sagemaker` client ([section 8](#8-amazon-sagemaker-ai-and-named-services)) |

Titan and other embedding models have no API of their own: call `InvokeModel` on `bedrock-runtime` with the embedding model ID. Titan Text Embeddings V2 takes `inputText`, `dimensions` (1,024, 512 or 256) and `normalize`.

---

## 14. Vector stores

| Task | API | Client | Exam note |
|---|---|---|---|
| Create a serverless vector collection | `CreateCollection` | `opensearchserverless` | Type `VECTORSEARCH` |
| Secure it | `CreateSecurityPolicy`, `CreateAccessPolicy`, `CreateVpcEndpoint` | `opensearchserverless` | Encryption and network policies; a **data access policy** grants index permissions. A missing data access policy is the usual 403 |
| Create a vector index | `PUT /index` with a `knn_vector` field | OpenSearch REST | Engine, HNSW parameters, dimension must match the embedding model |
| Vector, neural and hybrid search | `_search` with a `knn`, `neural` or `hybrid` query | OpenSearch REST | Hybrid needs a search pipeline with a normalization processor |
| Embed during ingestion | Ingest pipeline with a text embedding processor | OpenSearch REST | Neural plugin with a connector to a Bedrock embedding model |
| Low-cost vector storage | `CreateVectorBucket`, `CreateIndex`, `PutVectors`, `QueryVectors` | `s3vectors` | S3 Vectors: large, infrequently queried vector sets |
| Vectors in PostgreSQL | `CREATE EXTENSION vector`; HNSW or IVFFlat index; `ORDER BY embedding <=> query` | SQL (Aurora pgvector) | Combine similarity with SQL filters and joins |
| Graph-based retrieval | `CreateGraph`, `ExecuteQuery` | `neptune-graph` | Neptune Analytics behind GraphRAG |
| Conversation history and metadata | `PutItem`, `Query`, `UpdateTimeToLive` | `dynamodb` | Partition by session or user; time to live expires old turns |

---

## 15. Supporting services

| Task | API | Client | Exam note |
|---|---|---|---|
| Store validated model and prompt settings | `CreateConfigurationProfile`, `CreateHostedConfigurationVersion` | `appconfig` | Validators: JSON Schema or Lambda |
| Roll a change out gradually | `CreateDeploymentStrategy`, `StartDeployment`, `StopDeployment` | `appconfig` | Bake time and **automatic rollback** on a CloudWatch alarm |
| Read configuration at run time | `StartConfigurationSession`, `GetLatestConfiguration` | `appconfigdata` | In Lambda, the AppConfig extension caches this locally |
| Start a workflow | `StartExecution`, `StartSyncExecution` | `stepfunctions` | Standard for long-running; synchronous Express for short request-response |
| Resume after a human decision | `SendTaskSuccess`, `SendTaskFailure`, `SendTaskHeartbeat` | `stepfunctions` | Pairs with the `.waitForTaskToken` integration pattern |
| Re-run from the failed step | `RedriveExecution` | `stepfunctions` | No need to repeat completed model calls |
| Stream a Lambda response | `InvokeWithResponseStream`, `CreateFunctionUrlConfig` | `lambda` | Function URL `InvokeMode` is `RESPONSE_STREAM` or `BUFFERED` |
| Cap parallel model calls from a queue | `CreateEventSourceMapping` | `lambda` | Maximum concurrency on the SQS event source protects Bedrock quotas |
| Remove cold starts; shift traffic | `PutProvisionedConcurrencyConfig`, `CreateAlias`, `UpdateAlias` | `lambda` | Alias routing weights give a canary |
| Push chunks to a WebSocket client | `PostToConnection` | `apigatewaymanagementapi` | The streaming path for WebSocket APIs |
| Buffer and retry work | `SendMessage`, `ReceiveMessage`, `ChangeMessageVisibility` | `sqs` | Visibility timeout longer than the slowest model call; dead-letter queue |
| Publish and route events | `PutEvents`, `PutRule`, `PutTargets`, `CreateApiDestination` | `events` | EventBridge; API destinations call external webhooks |
| Publish custom metrics and alarms | `PutMetricData`, `PutMetricAlarm`, `PutCompositeAlarm`, `PutAnomalyDetector` | `cloudwatch` | Token usage, quality scores, anomaly detection on token bursts |
| Query logs | `StartQuery`, `GetQueryResults` | `logs` | **Logs Insights** over invocation logs (Skill 2.5.6) |
| Turn logs into metrics or streams | `PutMetricFilter`, `PutSubscriptionFilter` | `logs` | Count refusals or errors; forward to analysis |
| Mask sensitive data in logs | `PutDataProtectionPolicy` | `logs` | Protects PII that reached the log group |
| Trace requests | `PutTraceSegments`, `GetTraceSummaries` | `xray` | Normally through the SDK or OpenTelemetry, not direct calls |
| Audit API activity | `LookupEvents`, `PutEventSelectors`, `CreateEventDataStore` | `cloudtrail` | Management events by default; data events must be enabled |
| Raise a quota | `GetServiceQuota`, `RequestServiceQuotaIncrease` | `service-quotas` | Tokens per minute and requests per minute are per model and Region |
| Cost by tag; anomalies | `GetCostAndUsage`, `CreateAnomalyMonitor`, `GetAnomalies` | `ce` | Pair with application inference profile tags |
| Encrypt and decrypt | `GenerateDataKey`, `Decrypt`, `CreateGrant` | `kms` | Customer managed keys on knowledge bases, agents, custom models, logs |
| Fetch tool credentials | `GetSecretValue` | `secretsmanager` | Never place secrets in prompts or environment variables |
| Scan S3 for sensitive data | `CreateClassificationJob`, `CreateCustomDataIdentifier`, `GetFindings` | `macie2` | Before ingestion into a knowledge base (Skill 3.2.2) |
| Review an architecture | `CreateWorkload`, `AssociateLenses` | `wellarchitected` | Generative AI Lens (Skill 1.1.3) |

---

## 16. API chains by exam task

Read each arrow as "then". These are the sequences an option describes in words.

### Domain 1: FM integration, data and compliance

| Exam task | API chain |
|---|---|
| Switch model or provider without redeploying (1.2.2) | `StartConfigurationSession` → `GetLatestConfiguration` → `Converse` with the configured `modelId` |
| Survive a Regional capacity problem (1.2.3) | `ListInferenceProfiles` → `Converse` with the inference profile ID |
| Deploy a fine-tuned model on Bedrock (1.2.4) | `CreateModelCustomizationJob` → `CreateProvisionedModelThroughput` or `CreateCustomModelDeployment` → `Converse` |
| Deploy a fine-tuned model on SageMaker AI with approval (1.2.4) | `CreateTrainingJob` → `CreateModelPackage` → `UpdateModelPackage` (approve) → `CreateModel` → `CreateEndpointConfig` → `CreateEndpoint` |
| Serve many LoRA adapters on one endpoint (1.2.4) | `CreateInferenceComponent` (base) → `CreateInferenceComponent` (adapter) → `InvokeEndpoint` with the component name |
| Roll back a bad model deployment (1.2.4) | `UpdateEndpoint` with auto-rollback alarms, or `UpdateEndpoint` back to the previous configuration |
| Validate data before ingestion (1.3.1) | `CreateDataQualityRuleset` → `StartDataQualityRulesetEvaluationRun` → `GetDataQualityResult` |
| Process audio, documents and video (1.3.2) | `StartTranscriptionJob`; `InvokeDataAutomationAsync` → `GetDataAutomationStatus`; `CreateProcessingJob` for custom transforms |
| Enrich text with metadata (1.3.4) | `DetectEntities`, `DetectKeyPhrases`, `DetectDominantLanguage` → write a metadata file next to the document |
| Build managed RAG (1.4.1, 1.5.3) | `CreateKnowledgeBase` → `CreateDataSource` → `StartIngestionJob` → `GetIngestionJob` |
| Keep the vector store current (1.4.5) | S3 event → `StartIngestionJob`; or `IngestKnowledgeBaseDocuments` and `DeleteKnowledgeBaseDocuments` for single documents |
| Generate embeddings (1.5.2) | `InvokeModel` with an embedding model; in bulk `CreateModelInvocationJob` |
| Retrieve with hybrid search and reranking (1.5.4) | `Retrieve` with `overrideSearchType` and `rerankingConfiguration`; or `Rerank` on your own results |
| Decompose a compound query (1.5.5) | `RetrieveAndGenerate` with query transformation; or Lambda splits, several `Retrieve` calls, merge |
| Query structured data in natural language (1.5.5, 3.1.2) | `GenerateQuery` → run the SQL → `Converse` to phrase the answer |
| Manage prompt templates (1.6.1, 1.6.3) | `CreatePrompt` → `CreatePromptVersion` → `Converse` with the prompt ARN and `promptVariables` |
| Chain prompts with branches (1.6.6) | `CreateFlow` → `PrepareFlow` → `CreateFlowVersion` → `CreateFlowAlias` → `InvokeFlow` |
| Recognize intent (1.6.2) | `ClassifyDocument` on a Comprehend endpoint, or `RecognizeText` on a Lex bot |
| Keep conversation history (1.6.2) | `PutItem` and `Query` on DynamoDB; AgentCore `CreateEvent`; or `sessionId` on `InvokeAgent` |

### Domain 2: Implementation and integration

| Exam task | API chain |
|---|---|
| Build a managed agent (2.1.1) | `CreateAgent` → `CreateAgentActionGroup` → `AssociateAgentKnowledgeBase` → `PrepareAgent` → `CreateAgentAlias` → `InvokeAgent` |
| Execute an action in the caller's environment (2.1.x) | `InvokeAgent` returns a return-of-control event → application runs the action → `InvokeAgent` with `returnControlInvocationResults` |
| Deploy a code-first agent (2.1.1) | `CreateAgentRuntime` → `CreateAgentRuntimeEndpoint` → `InvokeAgentRuntime` with a `runtimeSessionId` |
| Expose existing APIs as agent tools (2.1.6, 2.1.7) | `CreateGateway` → `CreateGatewayTarget` |
| Give an agent memory (2.1.1) | `CreateMemory` → `CreateEvent` each turn → `RetrieveMemoryRecords` |
| Pause for human approval (2.1.5) | Step Functions task with `.waitForTaskToken` → `SendTaskSuccess` or `SendTaskFailure`; for review queues `StartHumanLoop` |
| Guarantee capacity (2.2.1) | `CreateProvisionedModelThroughput` → `Converse` with the provisioned model ARN |
| Call a self-hosted model (2.2.1) | `InvokeEndpoint`, `InvokeEndpointWithResponseStream`, or `InvokeEndpointAsync` |
| Stream to the user (2.4.2, 2.5.1) | `ConverseStream` → `InvokeWithResponseStream` (Lambda) or `PostToConnection` (WebSocket) |
| Decouple long requests (2.4.1) | `SendMessage` → Lambda consumer → `Converse` → store the result → notify |
| Route simple prompts to a cheaper model (2.4.4) | `CreatePromptRouter` → `Converse` with the router ARN |
| Automate document processing (2.5.3) | `CreateBlueprint` → `CreateDataAutomationProject` → `InvokeDataAutomationAsync` → EventBridge event → next step |

### Domain 3: Safety, security and governance

| Exam task | API chain |
|---|---|
| Filter input and output on a Bedrock call (3.1.1, 3.1.2) | `CreateGuardrail` → `CreateGuardrailVersion` → `Converse` with `guardrailConfig` |
| Apply the same guardrail to a non-Bedrock model (3.1.x) | `ApplyGuardrail` (input) → `InvokeEndpoint` → `ApplyGuardrail` (output) |
| Pre-process for PII and toxicity (3.1.4, 3.2.3) | `DetectPiiEntities`, `DetectToxicContent` → Lambda masks or blocks → `Converse` |
| Redact PII in a document set (3.2.2) | `StartPiiEntitiesDetectionJob` in redaction mode → ingest the redacted copies |
| Find sensitive data in S3 (3.2.2) | `CreateClassificationJob` → `GetFindings` → quarantine before `StartIngestionJob` |
| Restrict data access (3.2.1) | `GrantPermissions`, `CreateDataCellsFilter`; metadata `filter` on `Retrieve` |
| Record prompts and responses (3.3.x) | `PutModelInvocationLoggingConfiguration`; `PutDataProtectionPolicy` to mask PII in the log group |
| Audit who did what (3.3.2) | `LookupEvents` or a CloudTrail Lake query |
| Register and trace data sources (3.3.2) | `CreateCrawler` → `StartCrawler` → `GetTable` |
| Document a model (3.3.1, 3.4.3) | `CreateModelCard` → `CreateModelCardExportJob` |
| Check fairness (3.4.2) | `CreateEvaluationJob` with an LLM judge; `CreateModelBiasJobDefinition` for a hosted classic model |

### Domain 4: Operational efficiency and optimization

| Exam task | API chain |
|---|---|
| Estimate tokens before a call (4.1.1) | `CountTokens` |
| Cut cost of a repeated prefix (4.1.4) | `Converse` with a `cachePoint`; confirm with `cacheReadInputTokens` |
| Cut cost of bulk work (4.2.3) | `CreateModelInvocationJob` |
| Replace a large model with a distilled one (4.1.2) | `CreateModelCustomizationJob` with type `DISTILLATION` |
| Attribute cost per team (4.3.x) | `CreateInferenceProfile` with tags → `GetCostAndUsage` grouped by tag |
| Remove throttling (4.2.3) | Retry with backoff → inference profile as `modelId` → `RequestServiceQuotaIncrease` |
| Scale a self-hosted model (4.1.3) | `RegisterScalableTarget` → `PutScalingPolicy` |
| Monitor tokens and quality (4.3.2) | `PutMetricData` → `PutMetricAlarm` or `PutAnomalyDetector` |

### Domain 5: Testing, validation and troubleshooting

| Exam task | API chain |
|---|---|
| Compare models or prompts (5.1.2) | `CreateEvaluationJob` (model evaluation) → `GetEvaluationJob` |
| Evaluate a RAG system (5.1.5) | `CreateEvaluationJob` (RAG evaluation), retrieve-only or retrieve-and-generate |
| A/B test a hosted model (5.1.2) | `UpdateEndpointWeightsAndCapacities`; for Lambda `UpdateAlias` with routing weights; for settings `StartDeployment` in AppConfig |
| See why an agent chose a tool (5.2.2) | `InvokeAgent` with `enableTrace` |
| Find why documents are missing (5.2.4) | `GetIngestionJob` → `ListKnowledgeBaseDocuments` |
| Analyze prompts and responses (5.2.5, 2.5.6) | `StartQuery` → `GetQueryResults` on the invocation log group |
| Check model access (5.2.2) | `GetFoundationModelAvailability`; then the caller's IAM policy |

---

## 17. API pairs the exam contrasts

| Pair | Choose the first when | Choose the second when |
|---|---|---|
| `Converse` / `InvokeModel` | Chat, tools, guardrails, model portability | Embeddings, images, model-native fields |
| `Converse` / `ConverseStream` | The caller needs the whole answer (batch step, tool loop) | A person is watching; perceived latency matters |
| `Retrieve` / `RetrieveAndGenerate` | You control the prompt, the model or the post-processing | Least code, built-in citations and session |
| `Retrieve` (Bedrock) / `Retrieve` (Kendra) | The source is a Bedrock knowledge base | The source is a Kendra index |
| `StartIngestionJob` / `IngestKnowledgeBaseDocuments` | Sync a whole data source after changes | Add or change a few documents immediately |
| Guardrail in `Converse` / `ApplyGuardrail` | The model is on Bedrock | The model is elsewhere, or you check text with no model call |
| `InvokeAgent` / `InvokeInlineAgent` | A prepared, versioned agent behind an alias | The agent definition changes per request |
| `InvokeAgent` / `InvokeAgentRuntime` | Bedrock Agents | Your own agent code hosted on AgentCore Runtime |
| `InvokeFlow` / `StartExecution` | Bedrock Flows prompt chain | Step Functions workflow |
| `CreateModelInvocationJob` / `StartAsyncInvoke` | Many prompts, results later, lower cost | One long-running generation |
| `CreateProvisionedModelThroughput` / inference profile | Steady load, guaranteed capacity | Bursts and Regional availability |
| `InvokeEndpoint` / `InvokeEndpointAsync` | Low-latency online inference | Large payloads, long processing, scale to zero |
| `InvokeEndpointAsync` / `CreateTransformJob` | Requests arrive over time | A whole dataset is ready in S3 |
| `DetectPiiEntities` / `StartPiiEntitiesDetectionJob` | One text now; you mask with offsets | Many documents; you want redacted copies |
| `DetectDocumentText` / `AnalyzeDocument` | Text only | Forms, tables, queries, signatures |
| `InvokeDataAutomationAsync` / `InvokeDataAutomation` | Files in S3, any modality | One small image or short document, inline |
| `Query` / `Retrieve` (Kendra) | Search results for people | Passages for an FM prompt |
| `ChatSync` / `Converse` | Finished enterprise assistant with permissions | Your own application logic |
| `StartHumanLoop` / `SendTaskSuccess` | Managed reviewer workforce and UI | Your own approval step in a workflow |
| `LookupEvents` / `StartQuery` (Logs) | Who made which API call | What the prompts and responses contained |

---

## 18. IAM actions and condition keys

| Need | What to know |
|---|---|
| Permission for `Converse` | The IAM action is `bedrock:InvokeModel`. `ConverseStream` needs `bedrock:InvokeModelWithResponseStream`. There is no separate Converse action |
| Limit which models a role can call | Resource is the foundation model ARN, custom model ARN, provisioned model ARN or inference profile ARN |
| Cross-Region inference | Allow the **inference profile ARN and the model ARNs in every destination Region**. A Region-restricting SCP must allow those Regions or calls fail |
| Force a guardrail on every call | Condition key `bedrock:GuardrailIdentifier` on the invoke actions; deny when it is absent or different |
| Guardrail without a model call | Action `bedrock:ApplyGuardrail` on the guardrail ARN |
| Knowledge base queries | `bedrock:Retrieve` and `bedrock:RetrieveAndGenerate` on the knowledge base ARN; generation also needs invoke permission on the model |
| Agents | `bedrock:InvokeAgent` on the **agent alias** ARN. The agent's own service role needs model, knowledge base and Lambda access; the Lambda function needs a resource policy that trusts Bedrock |
| Jobs that read and write S3 | Customization, batch, evaluation and ingestion jobs take a **service role**; the caller needs `iam:PassRole` |
| Private access | Interface VPC endpoints exist per client: `bedrock`, `bedrock-runtime`, `bedrock-agent`, `bedrock-agent-runtime`. A missing runtime endpoint breaks inference even if the control endpoint exists |
| Customer managed keys | The service role needs `kms:GenerateDataKey` and `kms:Decrypt`; a missing key permission shows up as an access error on ingestion or invocation |

Error codes and their fixes are in [Domain 5: troubleshooting](../05-Domain5-Testing-Validation-Troubleshooting.md).

---

## Sources

- [Amazon Bedrock API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/Welcome.html)
- [Converse API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)
- [InvokeInlineAgent](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_InvokeInlineAgent.html)
- [AgentCore control plane: CreateMemory](https://docs.aws.amazon.com/bedrock-agentcore-control/latest/APIReference/API_CreateMemory.html)
- [Using the Bedrock Data Automation API](https://docs.aws.amazon.com/bedrock/latest/userguide/bda-using-api.html)
- [Step Functions integration with Bedrock](https://docs.aws.amazon.com/step-functions/latest/dg/connect-bedrock.html)
- [Exam guide: in-scope services](https://docs.aws.amazon.com/aws-certification/latest/ai-professional-01/aip-01-in-scope-services.html)
