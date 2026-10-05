# claude-sa-portfolio

A step-by-step learning path toward becoming an Anthropic Solutions Architect, together with the portfolio projects built along the way.

**Duration:** about 16 weeks at 8–10 hours a week.

Each phase has three parts: what to **learn**, what to **build**, and a **checkpoint** to pass before moving on.

> Course names and product features change often, so check the current [Anthropic Academy](https://anthropic.skilljar.com) catalog and [docs](https://docs.claude.com) as you go.

---

## Table of contents

- [Phase 0: Setup](#phase-0-setup-week-0-about-2-days)
- [Phase 1: Claude basics and the API](#phase-1-claude-basics-and-the-api-weeks-12)
- [Phase 2: Prompt engineering](#phase-2-prompt-engineering-weeks-34)
- [Phase 3: Evaluations](#phase-3-evaluations-week-5)
- [Phase 4: Tool use and structured actions](#phase-4-tool-use-and-structured-actions-week-6)
- [Phase 5: Retrieval (RAG) and context engineering](#phase-5-retrieval-rag-and-context-engineering-weeks-78)
- [Phase 6: Agents](#phase-6-agents-weeks-910)
- [Phase 7: MCP, the Model Context Protocol](#phase-7-mcp-the-model-context-protocol-week-11)
- [Phase 8: Enterprise deployment](#phase-8-enterprise-deployment-weeks-1213)
- [Phase 9: Solutions architect skills](#phase-9-solutions-architect-skills-weeks-1415)
- [Phase 10: Capstone and job preparation](#phase-10-capstone-and-job-preparation-week-16-onward)
- [Weekly routine](#weekly-routine)
- [My advantage](#my-advantage)

---

## Phase 0: Setup (Week 0, about 2 days)

- [ ] Create an account on the Claude Console (console.anthropic.com), get an API key and set a low spend limit.
- [ ] Install the Python SDK (`pip install anthropic`) and Claude Code.
- [ ] Bookmark these:
  - **[docs.claude.com](https://docs.claude.com)**: the main documentation.
  - **[Anthropic Academy](https://anthropic.skilljar.com)**: free courses.
  - **[anthropics/claude-cookbooks](https://github.com/anthropics/claude-cookbooks)**: runnable notebooks.
  - **[anthropics/courses](https://github.com/anthropics/courses)**: includes the prompt engineering tutorial.
  - **[anthropic.com/engineering](https://www.anthropic.com/engineering)**: the engineering blog.
- [ ] Make this repo public. Every project below goes into it.

---

## Phase 1: Claude basics and the API (Weeks 1–2)

📂 Lessons: [phase-1-claude-basics-and-api](phase-1-claude-basics-and-api/)

### Learn
- [ ] The model family (Opus, Sonnet, Haiku) and the tradeoff between quality, speed and cost. Learn how to pick a model for a given use case.
- [ ] The Messages API: roles, system prompts, `max_tokens`, `temperature`, stop reasons and streaming.
- [ ] Tokens, context windows, pricing, rate limits and the different usage tiers.
- [ ] Academy course: **"Building with the Claude API"**.

### Build
- [ ] A command-line chatbot with streaming and conversation history.
- [ ] A cost calculator: given a workload in requests per day and tokens per request, compare the monthly cost of each model.

### Checkpoint
- [ ] Explain to a non-technical person why you'd choose Haiku over Opus for a given task, with numbers.

---

## Phase 2: Prompt engineering (Weeks 3–4)

### Learn
- [ ] Clear and direct instructions, examples (few-shot), XML tags, step-by-step thinking and extended thinking, prefilling, and chaining prompts together.
- [ ] Structured outputs: getting reliable JSON back.
- [ ] Reducing hallucinations: grounding the answer in quotes from the source and allowing "I don't know".
- [ ] Work through the **interactive prompt engineering tutorial** in the courses repo, every exercise.

### Build
- [ ] A document extractor that turns invoices or energy contracts into validated JSON.
- [ ] A classifier for support tickets, written in three prompt versions, with the accuracy of each one compared.

### Checkpoint
- [ ] You can take a prompt that fails and fix it in a methodical way, without guessing.

---

## Phase 3: Evaluations (Week 5)

SAs who can measure quality win deals, so don't skip this phase.

### Learn
- [ ] How to define success criteria.
- [ ] Building test sets.
- [ ] Grading by exact code checks, by a model acting as the judge, or by humans.
- [ ] Running regression tests on prompts.
- [ ] Read the eval notebooks in the cookbooks.

### Build
- [ ] An eval harness of 50 or more test cases for the Phase 2 classifier. It should report accuracy, latency and cost per model.

### Checkpoint
- [ ] You can prove with data that prompt version 3 beats version 1.

---

## Phase 4: Tool use and structured actions (Week 6)

### Learn
- [ ] How to define tools and write their JSON schemas.
- [ ] The tool-use loop.
- [ ] Forcing a specific tool with `tool_choice`, and running tools in parallel.
- [ ] Handling errors.
- [ ] Server-side tools: web search, code execution and the Files API.
- [ ] Read the blog post **"Writing effective tools for agents"**.

### Build
- [ ] An "energy assistant" that calls three of your own tools: get meter data, get tariff and calculate savings.

### Checkpoint
- [ ] Draw the request and response sequence for one tool call from memory.

---

## Phase 5: Retrieval (RAG) and context engineering (Weeks 7–8)

### Learn
- [ ] How retrieval works: embeddings (through Voyage AI), how to split documents into chunks, hybrid search and reranking.
- [ ] **Contextual Retrieval**, Anthropic's technique (it has its own blog post and cookbook).
- [ ] **Prompt caching**: how it works and what it saves in cost and latency.
- [ ] **Batch API**: a 50% discount for jobs that can run asynchronously.
- [ ] Citations.
- [ ] Read **"Effective context engineering for AI agents"**.

### Build
- [ ] A question-answering system over 100 or more technical documents, for example EMS documentation. Measure retrieval quality and show the cost before and after caching.

### Checkpoint
- [ ] You can explain when to use retrieval, when to put everything in a long context window, and when to fine-tune instead. (The answer is rarely fine-tuning.)

---

## Phase 6: Agents (Weeks 9–10)

### Learn
- [ ] Read **"Building effective agents"** and study the difference between workflows and agents. The workflow patterns are prompt chaining, routing, parallelization, orchestrator-workers and evaluator-optimizer.
- [ ] **Claude Agent SDK**: the agent loop, subagents, permissions and hooks.
- [ ] **Claude Code**: CLAUDE.md files, skills, hooks, slash commands, headless mode and use in CI. Take the Academy course **"Claude Code in Action"**.
- [ ] Computer use: just the concepts.

### Build
- [ ] A multi-agent research assistant built with the Agent SDK.
- [ ] A Claude Code automation for a real repo, such as a PR review hook or a skill.

### Checkpoint
- [ ] Given a customer's problem, you can choose the simplest architecture that works.

---

## Phase 7: MCP, the Model Context Protocol (Week 11)

### Learn
- [ ] How the protocol is structured: hosts, clients and servers.
- [ ] What a server can expose: tools, resources and prompts.
- [ ] Transports: stdio and streamable HTTP.
- [ ] Authentication with OAuth, and the security risks.
- [ ] Academy courses: **"Introduction to MCP"** and the advanced MCP course.

### Build
- [ ] An MCP server that exposes an internal system you know, such as AMQP queues or EMS data. Connect it to Claude Code and Claude Desktop.

### Checkpoint
- [ ] You can explain to a CTO when MCP makes sense and when a plain API integration is the better choice.

---

## Phase 8: Enterprise deployment (Weeks 12–13)

### Learn
- [ ] Running **Claude on Amazon Bedrock** (Academy course) and on Google Vertex AI. Compare them with the Anthropic API directly.
- [ ] Security and compliance:
  - Data retention and zero data retention.
  - Single sign-on.
  - Private network connectivity (VPC and PrivateLink).
  - Running models in a specific region.
- [ ] Guardrails: prompt injection, jailbreak defenses and filtering what the model outputs.
- [ ] Scaling: rate limits, retries with backoff, fallback models, observability and logging.
- [ ] Products beyond the API: Claude.ai Team and Enterprise plans, Projects, connectors and the Claude Code enterprise rollout.
- [ ] Responsible AI: Anthropic's Usage Policy and Responsible Scaling Policy, and how Anthropic approaches safety. Customers will ask about these.

### Build
- [ ] Deploy the Phase 5 retrieval app on AWS. Use Bedrock with Lambda or ECS, store secrets in Secrets Manager, add CloudWatch metrics, and define it all with Terraform or CDK.
- [ ] Write an architecture diagram and a threat model for it.

### Checkpoint
- [ ] You can answer a security questionnaire from an enterprise customer.

---

## Phase 9: Solutions architect skills (Weeks 14–15)

At Anthropic the SA role is customer-facing, so this phase matters as much as the technical ones.

### Learn and practice
- [ ] **Discovery:** understand the business problem, the success metrics, the constraints and the stakeholders before you talk about technology.
- [ ] **Solution design documents:** problem, architecture, options with their tradeoffs, cost model, risks and rollout plan.
- [ ] **Demos:** a 15-minute live demo that tells a story. Practice it until it's smooth.
- [ ] **Objections:** "hallucinations", "data privacy", "too expensive", "why not GPT or Gemini?" and "how do we measure ROI?"
- [ ] **Proof-of-concept scoping:** set success criteria and a timeline of 2–4 weeks.

### Build
Two solution design documents for realistic scenarios:
- [ ] An energy utility that wants a field-technician assistant.
- [ ] A bank that wants to process documents with strict compliance requirements.

### Checkpoint
- [ ] Record yourself giving the demo, watch it back, and do it again.

---

## Phase 10: Capstone and job preparation (Week 16 onward)

- [ ] **Capstone:** an end-to-end project with retrieval, tools, MCP, evals, a cloud deployment and a cost model. Write a README for it as a case study.
- [ ] **Portfolio:** all the projects above, each with a short write-up and a 2–3 minute video.
- [ ] **Visibility:** blog posts on LinkedIn or Medium, contributions to the cookbooks or to MCP servers, local meetups.
- [ ] **Interview preparation:**
  - Live system design, for example "design a support agent for 10 million users".
  - Coding with the API.
  - A role-play with a customer.
  - "Why Anthropic?" Know the company's mission and safety work well.
- [ ] Check [anthropic.com/careers](https://www.anthropic.com/careers) for the current SA job descriptions and requirements, and map each requirement to a project in this portfolio.

---

## Weekly routine

| Day | Activity |
|---|---|
| 2 evenings | Learn: docs, courses, blog posts |
| 2 evenings | Build: projects |
| Weekend (2 hours) | Write up what you built and explain it out loud as if to a customer |
| Ongoing | Read Anthropic release notes and the engineering blog |

---

## My advantage

I already build AWS orchestration and messaging systems. Many candidates know prompting but not production cloud architecture, and that's what enterprise customers need. Make Phases 4, 7 and 8 my strengths, and use energy and industrial examples throughout this portfolio.
