---
title: LLM Gateways
description: |
  An environmental scan for a higher-education AI workgroup
categories: [ai-generated, research, llm-gateways]
date: 2026-09-17 07:01:30
---


Audience: a mixed-technical workgroup considering agentic AI in teaching, research, and administration.

Scope: a broad landscape, not a procurement recommendation or implementation manual.

## 1. The central idea

An **LLM gateway is a shared entry point between an application and the AI models it uses**. Instead of every application arranging its own connections, credentials, spending controls, and error handling, a gateway can provide those services in one place.

Think of a campus service desk that knows which services are available, checks who may use them, directs requests, and records usage. The service desk does not do the underlying work. Similarly, a gateway usually does not run the AI model or determine whether its answer is correct.

**Discussion takeaway:** gateways can make AI access easier to manage, but they are not inherently necessary, private, cheaper, or safe. Their value depends on the problem being solved and how they are configured.

### Start with six terms

| Term | Plain-language meaning |
| --- | --- |
| Large language model, or LLM | A model that generates or interprets language; many current models also process images, audio, or other inputs. |
| Model provider | An organization or service that makes a model available. The model's creator and the company running it may be different. |
| API | A way for software to request a service from other software. An API key is a credential, not a model. |
| Token | A unit of model input or output, often a word fragment. Many services charge separately for input and output tokens. |
| Inference | Running a model to produce a result. Someone's computer must perform this work, whether on campus, on a laptop, or in a provider's cloud. |
| Agent | An application that uses a model in a repeated workflow and may call tools, such as document search or a calendar. The application determines what tools are available and executes authorized actions. |

A paid chatbot subscription and paid API access are usually separate products. A gateway does not automatically turn a personal ChatGPT or Claude subscription into institution-wide API access. Check the relevant service terms and supported integrations.

### Where the gateway fits

```text
Person
  |
  v
Chat interface or agent application
  |
  v
LLM gateway: access rules, routing, budgets, monitoring
  |
  +--> Cloud model provider A
  +--> Cloud model provider B
  +--> Campus or laptop model server
```

Responses return through the gateway to the application. Some gateways retain copies of requests and responses in logs or caches.

An agent also has a separate action path:

```text
Agent application --> tool or MCP gateway, if used --> email, files, databases
```

**Controlling model calls does not automatically control tool actions.** A gateway that filters prompts cannot by itself stop an agent from deleting a file through a separate tool connection. Some products now combine LLM, tool, and agent traffic management, but each path still needs appropriate permissions. OWASP's [Excessive Agency guidance](https://genai.owasp.org/llmrisk/llm06-sensitive-information-disclosure/) explains why limited permissions and human approval matter.

### What a gateway is not

- **Not a chat interface:** users still need an application. A gateway's dashboard is often for administrators, not students.
- **Not necessarily a model host:** forwarding requests to another company's model is different from running that model.
- **Not an agent framework:** planning, memory, tool execution, and multi-step coordination usually live in the application or an additional platform component.
- **Not a knowledge base:** searching institutional documents, often called retrieval-augmented generation or RAG, requires a separate retrieval system and document permissions.
- **Not a truth or compliance engine:** filters and logs support governance; they do not establish factual accuracy, lawful data use, or accessibility.

Official introductions illustrating this common pattern: [Cloudflare overview](https://developers.cloudflare.com/ai-gateway/), [Microsoft overview](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities), and [LiteLLM overview](https://docs.litellm.ai/docs/simple_proxy).

## 2. Why gateways can be useful

| Need | What a gateway can contribute | Higher-education example | Qualification |
| --- | --- | --- | --- |
| One connection to several providers | A common interface and centrally maintained integrations | A research group compares models without rebuilding every application | Common interface does not mean identical capabilities or answers |
| Shared access without shared provider secrets | Separate gateway credentials for projects, users, or applications | Each course receives its own revocable access credential | Identity and advanced administration may require paid features |
| Spending visibility and limits | Usage attribution, budgets, rate limits, and model restrictions | A department allocates a pilot budget across courses | Some limits are approximate, delayed, or exclude certain billing paths |
| Continuity during outages | Retries, alternate providers, and fallback models | A public-information assistant remains available during a provider interruption | A fallback can change quality, privacy terms, or location of processing |
| Matching resources to tasks | Rules that select models by task, cost, or other criteria | A lower-cost model labels public documents; a stronger model handles a difficult synthesis | The routing rule must be evaluated, not assumed to be intelligent |
| Troubleshooting | Records of errors, latency, model selection, and cost | Staff identify which step makes a research assistant slow | Complete prompt logs may expose sensitive material |
| Consistent policy | Approved-model lists, content checks, and restrictions on destinations | Student-facing apps use only institutionally reviewed providers | Applications that bypass the gateway remain outside its control |
| Shared local and cloud access | One endpoint for campus models and approved commercial models | A lab uses campus inference for restricted data and cloud inference for public data | Classification errors or permissive fallbacks can defeat that separation |

These are available capabilities across the landscape, not a promise that every product or free edition includes them. Examples are documented by [Portkey](https://portkey.ai/docs/product/ai-gateway), [LiteLLM](https://www.litellm.ai/pricing), and [Kong](https://developer.konghq.com/ai-gateway/).

### Why agentic AI makes these questions more important

A single user request can trigger many model calls: planning, searching, summarizing, checking, and revising. Repeated calls can increase costs and make failures difficult to locate. Gateway records can connect model usage to a project or workflow, while the agent application needs its own maximum-step limits, tool permissions, and stopping conditions.

Distinguish four often-confused behaviors:

1. **Routing:** choose where a request goes.
2. **Fallback:** use another destination after a problem.
3. **Task-based model selection:** decide which model is suitable for this kind of work.
4. **Ensembling or critique:** ask multiple models to produce or review answers and combine the results.

The first two are common gateway capabilities. The third may use explicit rules or an additional learned router. The fourth normally requires workflow logic beyond basic forwarding. Asking a second model to critique an answer is not independent verification and can add cost.

[RouteLLM](https://github.com/lm-sys/RouteLLM) is a useful example of learned model selection rather than a complete institutional gateway. Its documentation emphasizes calibrating the cost-quality tradeoff. Its benchmark savings should not be treated as a forecast for a university's workloads.

## 3. When a gateway may not help, or may be inadvisable

| Situation | Why to hesitate | Simpler or safer direction |
| --- | --- | --- |
| One small application uses one approved provider | Another dependency may add more work than value | Direct API access with sensible provider-side limits |
| The need is simply a staff chatbot | A gateway does not supply a finished, supported user experience | Evaluate an approved chat product or front end first |
| No one can maintain a self-hosted service | Updates, security incidents, backups, and outages still need owners | Use a managed service or keep the architecture simpler |
| The selected gateway is not approved for the data | It becomes another organization or system handling sensitive content | Do not route that data through it pending review |
| The application needs a new provider-specific feature | Translation can lag or lose functionality | Native API access or a carefully tested passthrough route |
| Strictly reproducible research | Automatic routing, model aliases, and fallbacks may change conditions | Pin versions where possible; record actual providers and disable substitutions |
| Strict offline processing | A local gateway can still call cloud models, filters, or telemetry services | Verify every component and disable external routes |
| Very low latency is essential | An extra network hop, filter, or logging step may be material | Measure end-to-end performance against direct access |
| Existing campus API infrastructure already meets the need | A second control system duplicates cost and policy | Extend the existing service if its AI support is sufficient |

Additional failure modes deserve explicit discussion:

- **Centralized failure and compromise:** a shared gateway concentrates availability risk and valuable credentials. Redundancy, isolation, and recovery plans are not optional at institutional scale.
- **Privacy-changing fallbacks:** never silently send restricted data to an unapproved provider merely to avoid an error. In such cases, failing closed means refusing the request rather than relaxing the policy.
- **Retries and duplicate work:** retries can incur additional inference costs. Replaying a whole agent workflow can also repeat consequential actions; the application must prevent duplicate execution.
- **Unsafe caching:** exact caching reuses an identical request's answer; semantic caching reuses an answer to a similar request. Neither should mix users' permissions or reuse individualized student information across accounts. Similar questions may need different answers.
- **False assurance from guardrails:** automated checks can miss harmful content or reject legitimate material. This is especially relevant for teaching sensitive topics and working across languages and dialects.
- **Overstated portability:** “OpenAI-compatible” means some shared request conventions, not full compatibility with every endpoint, tool call, structured response, or streaming behavior. [Ollama explicitly describes subset compatibility](https://docs.ollama.com/api/openai-compatibility).

## 4. Deployment choices: two independent decisions

Ask separately: **Who operates the gateway? Where does inference happen?** The answers need not be the same.

| Pattern | Gateway operator and location | Possible model locations | Main advantage | Main burden or risk |
| --- | --- | --- | --- | --- |
| Managed cloud service | Vendor operates it | Usually supported cloud providers; private endpoints depend on product/networking | Quick start; little gateway maintenance | Additional vendor, fees, data handling, and dependency |
| Vendor software in a private environment | Vendor and/or campus IT, under contract | Cloud, campus, or private cloud | Enterprise support with more deployment control | Licensing and infrastructure complexity |
| Self-hosted open-source gateway | Campus IT or research team | Cloud and/or campus | Control over routing, configuration, and gateway records | Staffing, upgrades, security, databases, and availability |
| Local gateway | Individual's computer | Cloud APIs and/or local models | Convenient experimentation and one local connection | Usually not a dependable multi-user service |
| Entirely local stack | Individual's computer | Local model runtime | Can operate without sending prompts off-device | Hardware limits, model suitability, and local security |
| No separate gateway | Application connects directly | Cloud or local | Fewer components | Shared controls must live elsewhere if needed |

**Local gateway does not mean local inference. Self-hosted does not mean no data leaves campus. Open-source software does not mean free operation.**

A useful technical distinction for procurement: the **data plane** carries requests and responses; the **control plane** manages configuration and administration. A vendor-hosted control plane with a campus-hosted data plane is a hybrid deployment, not automatically a fully isolated system. Ask what configuration, credentials, logs, and telemetry each part receives.

### Three accessible implementation sketches

**A. Managed comparison pilot**

An approved chat or agent application connects to a service such as OpenRouter or Vercel AI Gateway. The project selects approved models, establishes billing, limits access, and uses public or synthetic material. Suitable for testing the value of multiple models without operating a server.

**B. Shared campus service**

An application connects to a university-operated gateway, such as LiteLLM or an extension of existing API management. IT connects identity, provider contracts, budgets, approved routes, monitoring, and support. This can serve many projects, but it is a service-management commitment, not merely a software installation.

**C. Laptop-only demonstration**

A local interface or agent connects to a local model server such as Ollama or LM Studio. Add a local gateway only if model routing or common controls are part of the lesson. A model must be downloaded beforehand and fit available memory. Disable cloud models, cloud fallbacks, external tools, and remote logging if the demonstration is intended to be offline.

For local experiments, bind services to the local machine unless sharing is deliberate; do not expose unauthenticated model or gateway endpoints to a public network. Store provider credentials outside shared documents and notebooks.

## 5. Commercial and managed landscape

### How to read this survey

There is no reliable, independently audited market-share ranking in the sources reviewed. This is a **representative shortlist of prominent options**, selected for public documentation, developer visibility, and diversity of approach, not a claim about revenue or installed-base leadership.

Open-source visibility offers a limited signal: GitHub pages displayed approximately 59,000 stars for LiteLLM, 13,000 for Portkey's gateway, and 8,100 for Bifrost when checked. Stars measure attention, not production adoption, security, or commercial market share. Enterprise cloud products are not comparable using that measure.

Features below describe documented offerings, sometimes spanning paid editions. Verify the exact plan, supported endpoints, deployment region, and contract. A blank or missing capability in this survey means **not assessed**, not necessarily unavailable.

### Managed and AI-focused services

| Offering | Documented strengths | Deployment and payment approach | Fit and caution |
| --- | --- | --- | --- |
| **[OpenRouter](https://openrouter.ai/docs/quickstart)** | One API for many models; provider selection and fallbacks; aggregated billing and usage visibility | Managed model-access service; prepaid inference credits; BYOK supported with separate fee rules | Good starting point for comparing model access. Gateway and downstream provider data policies both matter; not all providers have identical retention or training terms |
| **[Portkey](https://portkey.ai/docs/product/ai-gateway)** | Conditional routing, retries, fallbacks, load balancing, caching, budgets, guardrails, multimodal and MCP support | Hosted offering plus open-source gateway and commercial private deployments | Broad AI operations approach. Do not equate the standalone open-source gateway with the complete commercial platform; current docs display “PRISMA AIRS AI Gateway” branding |
| **[Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/)** | Analytics, logging, caching, rate limiting, retries and fallbacks; DLP and guardrail options | Managed; free core capabilities; BYOK and Unified Billing paths; some services incur charges | Attractive for an inexpensive managed control layer. AI Gateway and Workers AI are different products: the former manages traffic, the latter runs models |
| **[Vercel AI Gateway](https://vercel.com/docs/ai-gateway)** | Multi-provider access, routing, fallbacks, request records, credentials, spending controls, multiple API formats and modalities | Managed; provider-priced inference and BYOK; documentation advertises no token-price markup | Not limited to applications hosted on Vercel. Budget scope is important: documented system-credential budgets do not include BYOK spend, and BYOK failures can fall back to system credentials |
| **[Helicone](https://github.com/Helicone/helicone)** | Gateway plus observability: cost/latency tracking, sessions and traces, prompt management, routing and fallbacks | Hosted plans and self-hostable components; gateway and observability repositories have different licenses | Useful when debugging and understanding AI behavior are central. Pricing page announces joining Mintlify; reconfirm roadmap, support, and availability before adoption |
| **[TrueFoundry](https://www.truefoundry.com/ai-gateway)** | Unified model access, budgets, routing, governance, observability, guardrails, and related MCP/tool controls | SaaS plans and enterprise private deployment options | Broad institutional platform candidate. Verify what is included in gateway versus wider platform services |
| **[LiteLLM Enterprise](https://www.litellm.ai/pricing)** | Adds SSO, identity provisioning, audit logs, secret management, administration, support, and deployment options to its open-source gateway | Commercial self-hosted offering; annual capacity/deployment/support-based quote | Relevant when a campus wants open-source foundations plus paid operational support. Commercial support is not the same as outsourcing all operations |

BYOK means “bring your own key”: the gateway uses your existing model-provider account. It does not mean the gateway cannot see the prompt, or that provider usage is free. DLP means data-loss prevention, such as detecting or redacting selected sensitive information. SSO means single sign-on through an institutional identity system.

### Existing API-management and cloud ecosystems

| Offering | What it adds or represents | Why include it |
| --- | --- | --- |
| **[Kong AI Gateway / Konnect](https://developer.konghq.com/ai-gateway/)** | Unified controls for model, MCP, and agent traffic; provider routing, token budgets, semantic caching, prompt controls, sensitive-data redaction, analytics | Important for institutions already managing APIs with Kong. Supports hybrid control/data-plane patterns. An open-source Kong base does not imply every AI capability is free |
| **[Azure API Management AI gateway](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities)** | Extends API Management with AI endpoint governance, token quotas, routing, monitoring, and model/tool/agent integrations | Particularly relevant to Microsoft-oriented campuses. It is not a separate standalone gateway product; features vary by tier. The reviewed page labels its unified model API as preview |
| **[Google Cloud Apigee](https://docs.cloud.google.com/apigee/docs/api-platform/get-started/ai-capabilities)** | Policy-based model routing, token limits, semantic caching, Model Armor integration, and MCP security | Relevant where Apigee or Apigee hybrid already supports institutional APIs. Include costs and operation of integrated services, not only the gateway |
| **[Amazon Bedrock Converse](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)** | A consistent conversation API across supported Bedrock models, including model-dependent tool use and guardrails | An adjacent alternative, not a general neutral gateway across arbitrary external providers. If all required models and controls are already available inside Bedrock, another gateway may be unnecessary |

Microsoft Foundry and Google Vertex AI also belong in the wider model-platform conversation. Model platforms provide access to models and related services; API gateways govern traffic to those services. These categories increasingly overlap, but their scopes and contracts remain different.

### Pricing: compare the structure before the headline

Illustrative published information checked on the research date, not a quote:

| Offering | Verified pricing structure or example | What remains to check |
| --- | --- | --- |
| OpenRouter | [FAQ](https://openrouter.ai/docs/faq) describes provider inference prices, credit-purchase fees, and plan-dependent BYOK allowances/fees | Exact numerical fees did not render in the retrieved FAQ/pricing content; verify on the live [pricing page](https://openrouter.ai/pricing) before calculating costs |
| Cloudflare | [Pricing](https://developers.cloudflare.com/ai-gateway/reference/pricing/) lists free core analytics, caching, and rate limiting; a 5% fee on Unified Billing credit purchases; guardrails billed through Workers AI | Log storage/export limits, paid integrations, and actual inference usage |
| Vercel | [Overview](https://vercel.com/docs/ai-gateway) states zero markup on provider token prices, including BYOK | Credits, payment rules, applicable limits, and which spending controls cover which credentials |
| Helicone | [Pricing](https://www.helicone.ai/pricing) lists a free tier, Pro at $79/month, Team at $799/month, and enterprise quotes; usage-based charges also apply | These platform prices are not a complete model-inference bill; reconfirm terms following the announced organizational change |
| TrueFoundry | [Pricing](https://www.truefoundry.com/pricing) lists a free developer tier and Pro at $25/user/month, including 20,000 requests/tool calls per user, then $15 per additional 100,000 | Model-provider and external guardrail charges are excluded; enterprise/private deployment terms require review |
| LiteLLM | [Pricing](https://www.litellm.ai/pricing) lists $0 open-source license cost and quoted enterprise plans | Hosting, databases, staff time, provider charges, and commercial features |
| Portkey, Kong, Azure, Apigee | Commercial offerings with edition, deployment, or usage considerations | Obtain a workload-specific estimate; this scan did not verify comparable total prices |

A more useful cost model is:

```text
Total cost = model inference + gateway fees + logs/storage + guardrails
           + hosting/networking + staff time + support
```

Caching or cheaper-model routing can reduce part of the bill, but extra evaluations, retries, and agent steps can increase it. Compare **cost per successfully completed task**, not only price per token.

## 6. Open-source and self-hostable alternatives

Open source, self-hostable, and free of charge are different attributes. “Open core” means an open-source foundation with additional commercially licensed features. Check the license of the exact release and component, particularly before modifying or redistributing software.

| Project and repository | Approach and documented capabilities | Deployment and tradeoff | License/status notes |
| --- | --- | --- | --- |
| **[LiteLLM](https://github.com/BerriAI/litellm)** | Broad provider translation; gateway credentials, users/teams, budgets, spend tracking, fallbacks, logging; also available as a programming library | Local process through shared institutional deployment; persistent governance features introduce database/operations needs | [MIT outside separately licensed enterprise components](https://raw.githubusercontent.com/BerriAI/litellm/main/LICENSE); paid identity/audit/support capabilities |
| **[Portkey Gateway](https://github.com/Portkey-AI/gateway)** | Lightweight multi-provider gateway; retries, fallback strategies, load balancing, conditional routing, multimodal support, guardrail integrations | Local Node.js/container deployment and larger deployments; commercial platform adds management capabilities | MIT repository; README describes a 2.0 pre-release transition, so compare stable-release features rather than assuming parity |
| **[Bifrost](https://github.com/maximhq/bifrost)** | Go-based gateway with an OpenAI-compatible interface, failover, load balancing, semantic caching, and built-in configuration/monitoring UI | Laptop/container start and enterprise deployment paths | Apache-2.0 core; README places advanced clustering, adaptive balancing, guardrails, and other capabilities in enterprise offerings; vendor speed claims not independently tested here |
| **[Helicone AI Gateway](https://github.com/Helicone/ai-gateway)** | Rust-based gateway with provider selection, cost/rate controls, caching, and telemetry integration | Standalone/self-hosted gateway or hosted path; full observability stack is a separate deployment consideration | Gateway repository shows GPL-3.0; [observability repository](https://github.com/Helicone/helicone) shows Apache-2.0. Do not assume one license covers everything |
| **[Agent Router, formerly Envoy AI Gateway](https://github.com/theagentrouter/agent-router)** | Envoy-based control layer for models and tools; credentials, routing, quotas, failover, and usage attribution | Standalone local CLI and Kubernetes/Envoy deployment; especially relevant to platform teams | Apache-2.0. [Former repository README](https://raw.githubusercontent.com/envoyproxy/ai-gateway/main/README.md) documents the rename and unchanged deployment resource names |
| **[Apache APISIX](https://github.com/apache/apisix)** | General API gateway extended with AI proxying, multi-model balancing/fallbacks, token limiting, and MCP bridging | Good option when conventional API traffic and AI traffic share infrastructure; more general-purpose configuration to learn | Apache-2.0 project; AI functionality depends on plugins and version |
| **[Higress](https://github.com/higress-group/higress)** | Istio/Envoy-based gateway; unified model access, caching, token limits, balancing, observability, and MCP hosting through plugins | Local Docker and Kubernetes paths; broader infrastructure orientation | Apache-2.0; useful perspective beyond the predominantly US commercial shortlist |
| **[TensorZero](https://github.com/tensorzero/tensorzero)** | Combines a gateway with observability, evaluation, optimization, and experimentation | Illustrates an evaluation-centered approach rather than only access control | Apache-2.0; repository explicitly marked archived June 12, 2026. Historical/reference inclusion, not a default recommendation for a new maintained service |
| **[RouteLLM](https://github.com/lm-sys/RouteLLM)** | Framework for learned routing between stronger and cheaper models, with evaluation tools | Research-oriented complement to a gateway; not a complete campus identity/budget/governance platform | Review current maintenance, dependencies, and model compatibility; example providers/models may be dated |

A library embedded in an application can reduce integration complexity without deploying a shared network service. LiteLLM's library and proxy modes illustrate that distinction. A small custom proxy is another option, but authentication, streaming, secret storage, retries, and compatibility maintenance quickly become substantial work. Prefer an established component unless the requirement is genuinely narrow.

## 7. Local models and user interfaces: related, but different layers

These tools matter because they can make a gateway unnecessary for a small local setup, or supply the models and interface behind one.

| Tool | What it provides | Relationship to gateways |
| --- | --- | --- |
| **[Ollama](https://github.com/ollama/ollama)** | Model runtime/server with an [OpenAI-compatible API subset](https://docs.ollama.com/api/openai-compatibility); MIT runtime license | A local backend, not a complete campus gateway. Ollama also supports cloud access: confirm the selected model actually runs locally |
| **[LM Studio](https://lmstudio.ai/docs/developer/openai-compat)** | Desktop-oriented local model experience and API server | Accessible local demonstration option. Treat the application and its current terms separately from open-source inference engines and model licenses |
| **[llama.cpp](https://github.com/ggml-org/llama.cpp)** | MIT-licensed inference software with an API server and broad hardware support | Lower-level local or server backend; useful when hardware and deployment control matter |
| **[vLLM](https://github.com/vllm-project/vllm)** | High-throughput model-serving software with compatible APIs and batching | More relevant to shared lab/campus inference infrastructure than a nontechnical desktop demo; not itself a full organizational governance gateway |
| **[Open WebUI](https://github.com/open-webui/open-webui)** | Self-hostable user interface connecting to [cloud and local providers](https://docs.openwebui.com/getting-started/quick-start/connect-a-provider/) | Can supply the human-facing interface and connect directly or through a gateway. Its [current license](https://raw.githubusercontent.com/open-webui/open-webui/main/LICENSE) contains branding restrictions; do not describe it as an unrestricted permissive license |

Software licenses and **model licenses** are separate. A freely available runtime does not establish permission to use every downloadable model for every purpose. Nor does an open-weights model necessarily meet every definition of open-source AI.

A useful first experiment is to compare the same public prompt through a local model and an approved cloud model. Only add a gateway after identifying a control or convenience that the direct setup lacks.

## 8. Higher-education governance checklist

This is a review framework, not legal advice or a claim that a product meets institutional obligations. In the US, [FERPA protects education records and rights concerning disclosure](https://studentprivacy.ed.gov/faq/what-ferpa); a gateway does not remove those obligations. Research agreements, consent, intellectual property, health data rules, and jurisdiction-specific privacy requirements may also apply.

Before using real institutional data, ask:

- [ ] **Purpose:** What specific problem requires a gateway instead of a direct API or approved chat service?
- [ ] **Data classification:** Are prompts likely to contain grades, accommodations, advising notes, unpublished research, participant data, or confidential documents?
- [ ] **Full data path:** Which application, gateway, model host, guardrail service, logging system, and backup system can receive content?
- [ ] **Contracts and location:** Are all relevant providers and subprocessors approved? What processing regions, retention terms, training policies, and deletion commitments apply?
- [ ] **Logging:** Do we need full text, or only cost/error metadata? Who may view records, and how long are they retained?
- [ ] **Identity:** Can access be tied to institutional identities and revoked promptly? Avoid sharing an instructor's master credential with a class.
- [ ] **Authorization:** Does document retrieval preserve each user's permissions? Do tool services independently enforce access?
- [ ] **Routing:** Can restricted workloads use only approved destinations? Do fallbacks preserve data and capability requirements?
- [ ] **Budgets:** Are limits per person, course, grant, or application? Are they hard or soft, and what happens to in-flight requests?
- [ ] **Agent boundaries:** Which tools are read-only? Which actions need human approval? Are loops, retries, and concurrency bounded?
- [ ] **Research reproducibility:** Can we export the actual model/provider, configuration, version information, routing decisions, and relevant records without unnecessarily retaining personal data?
- [ ] **Equity and accessibility:** Can students participate without personal payment credentials? Is the front end accessible? Have quality and filtering been tested across relevant languages and learner populations?
- [ ] **Operations:** Who handles upgrades, vulnerabilities, outages, complaints, and policy changes after the pilot ends?
- [ ] **Exit:** Can we export configuration and appropriate records, revoke keys, delete retained content, and migrate applications?

“No training on your data,” “zero retention,” and “regional processing” answer different questions. Encryption in transit does not mean the gateway or provider cannot process the plaintext. OpenRouter's [provider-policy discussion](https://openrouter.ai/docs/features/privacy-and-logging) is a concrete example of why downstream providers need separate review.

## 9. Suggested discussion and low-risk next steps

### A 45-minute workgroup agenda

| Time | Activity | Question |
| --- | --- | --- |
| 0–8 minutes | Walk through the two diagrams | Where does a prompt go, and where does an agent take action? |
| 8–18 minutes | Compare three campus scenarios | What control is missing today? |
| 18–28 minutes | Contrast managed, self-hosted, and local arrangements | Who operates each component, and who sees the data? |
| 28–38 minutes | Review the landscape and governance checklist | Which approaches deserve a pilot, and which can be ruled out? |
| 38–45 minutes | Agree on one experiment and an owner | What evidence would make us decide not to adopt a gateway? |

### Three scenarios to keep the conversation concrete

1. **Teaching:** An instructor wants equivalent access for a class using synthetic writing samples. Consider course budgets, accessibility, consent, and who can inspect logs. Compare an approved finished chat service against a gateway-backed interface.
2. **Research:** A lab wants to compare models on public texts. A gateway may reduce integration work, but pinning conditions and recording actual routes matter more than automatic fallback. Use restricted participant data only after separate review.
3. **Administration:** An assistant answers public policy questions and proposes calendar changes. The gateway can manage model access, but document freshness, calendar permissions, and approval before changes remain application/tool responsibilities.

### A sensible pilot shortlist

- **Managed model-access comparison:** OpenRouter and Vercel AI Gateway.
- **Managed controls and observability:** Cloudflare, Portkey, Helicone, or TrueFoundry, depending on priorities and lifecycle review.
- **Shared self-hosted evaluation:** LiteLLM and Bifrost; consider Portkey's standalone gateway where its feature set fits.
- **Existing enterprise infrastructure:** start with the campus's current Azure API Management, Kong, or Apigee capabilities before adding a separate platform.
- **Infrastructure-focused open-source paths:** Agent Router, APISIX, or Higress when platform staff already have relevant expertise.
- **Local baseline:** direct Ollama or LM Studio access, then add a gateway only to demonstrate a specific benefit.

Use public or synthetic data. Compare direct access with one gateway on the same small set of representative tasks. Measure task success, actual cost including retries, latency, routing behavior, and time spent administering the system. Test budget exhaustion, provider failure, denied routes, and log deletion. A successful pilot can conclude that no gateway is needed.

## 10. Reading path and research limitations

### Short reading path for nontechnical participants

1. [Cloudflare: AI Gateway overview](https://developers.cloudflare.com/ai-gateway/) – a short introduction to observation and control.
2. [Microsoft: AI gateway capabilities](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities) – how the need changes as applications and teams multiply.
3. [OpenRouter: provider data policies](https://openrouter.ai/docs/features/privacy-and-logging) – why one API does not imply one data policy.
4. [OWASP: Excessive Agency](https://genai.owasp.org/llmrisk/llm06-sensitive-information-disclosure/) – why tool permissions and human approval remain necessary.
5. [Open WebUI: connecting providers](https://docs.openwebui.com/getting-started/quick-start/connect-a-provider/) – a concrete explanation of interface, API, and model server.
6. [US Department of Education: What is FERPA?](https://studentprivacy.ed.gov/faq/what-ferpa) – a starting point for the student-records discussion, not a substitute for institutional review.

### Technical follow-up

- [LiteLLM proxy documentation](https://docs.litellm.ai/docs/simple_proxy) and [repository](https://github.com/BerriAI/litellm).
- [Portkey gateway documentation](https://portkey.ai/docs/product/ai-gateway) and [repository](https://github.com/Portkey-AI/gateway).
- [Bifrost repository](https://github.com/maximhq/bifrost).
- [Kong AI Gateway documentation](https://developer.konghq.com/ai-gateway/).
- [Apigee AI capabilities](https://docs.cloud.google.com/apigee/docs/api-platform/get-started/ai-capabilities).
- [Agent Router repository](https://github.com/theagentrouter/agent-router), [APISIX repository](https://github.com/apache/apisix), and [Higress repository](https://github.com/higress-group/higress).
- [RouteLLM repository](https://github.com/lm-sys/RouteLLM) for task-sensitive routing experiments.

### Method and limits

This scan used targeted web searches for discovery, followed by bounded reads of official documentation, pricing pages, project READMEs, and selected licenses. Vendor comparison articles were discovery leads, not independent evidence of superiority. Product claims above are documented capabilities, not results of hands-on testing or security audits. Architectural recommendations and campus scenarios are synthesis.

Important limits:

- **Popularity:** no commercial market-share ranking was established. GitHub attention is only a rough open-source visibility signal.
- **Fast-changing products:** the reviewed sources show Portkey branding changes, Helicone joining Mintlify, Envoy AI Gateway becoming Agent Router, and TensorZero being archived. Recheck lifecycle and support before procurement.
- **Pricing extraction:** OpenRouter's numerical fees and Portkey's usable plan details were not available in the retrieved content. They are deliberately not guessed. Enterprise comparisons require quotes.
- **Retrieval quality:** an initially attempted Apigee URL did not return usable content; the current official capabilities page was located and read instead. Some long repository pages were truncated, with targeted README/license reads used for relevant claims. This is not a full audit of every feature or license obligation.
- **Evidence quality:** vendor performance and compliance slogans were not treated as independently verified results. No software was installed, no model benchmarks were run, and no institutional data was sent to a gateway.
- **Further work:** a shortlist should trigger version-specific capability tests, security and accessibility review, institutional contract review, and a realistic total-cost estimate.

**Bottom line:** choose a gateway to solve a defined access, cost, reliability, or governance problem. Do not adopt one merely because the application is called an agent.
