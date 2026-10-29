# Roofing Warranty Claim Assistant: Comprehensive AI Project Proposal

## 1. Project Overview

### Core Idea

The **Roofing Warranty Claim Assistant** is an AI-powered web application designed to streamline, accelerate, and standardize the creation of warranty claim response letters for a roofing material manufacturer.

### Real-World Problem & Opportunity

Currently, processing customer warranty claims is a manual, time-consuming bottleneck. Field technicians submit unstructured site visit reports (descriptions and photographs) via Salesforce. The warranty manager must manually review these reports, cross-reference them against complex, version- and location-specific warranty documents, determine warrantability, and draft a response letter following a strict corporate template.

This manual workflow leads to inconsistent response times, potential human oversight in policy interpretation, and administrative fatigue. By introducing a structured AI pipeline, we can automate the heavy lifting—summarizing field data, retrieving exact policy terms via RAG, and drafting compliant response letters—while retaining essential human oversight before anything is sent to a customer.

## 2. Target Users & Workflow

### Target Users

* **Primary Users:** Warranty Managers and Customer Service Representatives who oversee claim approvals and final communications.

* **Secondary Users / Stakeholders:** Executive leadership monitoring response times and accuracy metrics.

### End-to-End User Workflow

1. **Trigger:** A field technician submits a new site visit report and associated photographs into Salesforce.

2. **Ingestion & Retrieval:** The application detects the new record, chunks and embeds the site visit report, and queries the vector database for the specific customer's warranty agreement.

3. **AI Analysis & Chaining:** The system executes a multi-step workflow:

   * *Step 1:* Evaluates technical findings against warranty terms (Chain-of-Thought).

   * *Step 2:* Determines warrantability (Approved, Denied, or Courtesy Fix).

   * *Step 3:* Populates the standard corporate response letter template.

4. **Human-in-the-Loop Review:** The warranty manager reviews the generated letter and supporting evidence in a clean web dashboard. They can approve, edit, or reject the draft.

5. **Output & Dispatch:** Once approved, the letter is pushed back to Salesforce and queued for delivery to the customer.

## 3. AI Capabilities & Course Concepts Implemented

This application leverages concepts from across the entire curriculum to ensure reliability, accuracy, and safety:

* **The Big Three & Context:** Combines dynamic contextual inputs (Salesforce site reports and customer-specific warranty documents) with expert human input (the warranty manager's final review) and rigorous evaluation datasets.

* **LLMs & Prompting:** Utilizes advanced prompting strategies, including **Few-Shot Prompting** (providing historical approved letters as examples) and **Chain-of-Thought (CoT)** reasoning to ensure the model explains *why* a claim is warrantable before drafting the letter. Distinct **Message Roles** (System, User, Assistant) strictly bound the LLM's persona and output constraints.

* **Structured Outputs:** Enforces strict Pydantic schemas and JSON parsing so that LLM outputs return predictable, machine-readable data objects (containing fields like `warrantability_status`, `policy_clause_referenced`, `reasoning_summary`, and `letter_body`) rather than unpredictable freeform text.

* **RAG & Vector Databases:** Implements a Retrieval-Augmented Generation pipeline. Warranty agreements and historical site visit reports are chunked, embedded using vector embeddings, and stored in a vector database (e.g., Pinecone or Chroma) for precise semantic similarity search during claim review.

* **Chaining:** Breaks the complex claim processing workflow into sequential logical steps (Ingest $\rightarrow$ RAG Lookup $\rightarrow$ Warrantability Analysis $\rightarrow$ Template Population $\rightarrow$ Human Gate).

* **Guardrails & Safety:** Employs input validation (checking for missing photographs or blank report fields), output filtering to prevent hallucinations or policy violations, and adversarial testing against edge-case or fraudulent claims.

* **Humans in the Loop (HITL):** Acknowledges that AI confidence is not 100%. Implements a mandatory review dashboard where human experts validate, correct, and provide feedback on drafts, turning real-world usage into a continuous improvement loop.

* **Observability & Evaluation:** Integrates tracing tools (such as LangSmith or Arize Phoenix) to monitor latency, token usage, and prompt drift, backed by a ground-truth dataset of historical claims to continuously calculate accuracy metrics.

* **AI with Git & Advanced Cursor:** Developed using modern AI-assisted engineering practices, leveraging IDE indexing, shared workspace rules (`.cursorrules`), and automated code/security reviews via Git workflows.

## 4. Technical Approach & Architecture

### High-Level Architecture

* **Backend Framework:** Python with FastAPI to handle API requests and orchestrate the AI workflow chains.

* **AI Orchestration & LLM:** OpenAI API (or equivalent enterprise LLM) accessed via structured completion libraries (e.g., LangChain or instructor with Pydantic).

* **Vector Database & Embeddings:** ChromaDB / Pinecone storing chunked warranty documents and historical Salesforce records.

* **Database & Integrations:** PostgreSQL for application state, integrated with Salesforce REST APIs for webhook triggers and record updates.

* **Frontend:** A lightweight React dashboard (single-file or modular Vite setup) providing the Human-in-the-Loop review interface.

* **Development Environment:** Built and maintained using **Cursor** with custom workspace rules and automated Git hooks for linting and security scanning.

## 5. Example Prompts & Expected Outputs

Below are three examples demonstrating the inputs and outputs at the core analysis stage. Each example uses the exact same static `system_role` prompt to demonstrate how runtime variables in the `user` payload yield predictable, structured responses.

### Example 1: Courtesy Fix / Non-Warrantable Claim

#### Input Prompt

```json
{
  "system_role": "You are an expert technical warranty analyst for a roofing material manufacturer. Analyze the provided field site visit report against the attached customer warranty text. Use Chain-of-Thought reasoning to determine if the reported issue is covered under the warranty terms. Draft a response letter following corporate guidelines. If information is ambiguous, flag the claim for human inspection. Output must adhere strictly to the required Pydantic JSON schema.",
  "user": {
    "site_report": "Field Tech Notes: Observed minor granule loss and flashing separation on the north slope of the roof installed in 2018. Flashing was improperly installed by contractor. Fixed flashing on-site as a courtesy.",
    "warranty_excerpt": "Section 4.2: Manufacturer warrants against manufacturing defects in shingles for 30 years. Excludes damage caused by improper installation by third-party contractors."
  }
}
```

#### Expected Output (JSON Schema Compliance)

```json
{
  "claim_id": "CLM-2026-8942",
  "warrantability_status": "DENIED_WITH_COURTESY_FIX",
  "confidence_score": 0.95,
  "policy_clause_referenced": "Section 4.2 (Improper Installation Exclusion)",
  "reasoning_steps": [
    "1. Analyzed technician notes: Granule loss is minor, but flashing separation was caused by contractor installation error.",
    "2. Cross-referenced Section 4.2: Installation errors by third-party contractors are explicitly excluded from warranty coverage.",
    "3. Noted action taken: Technician performed a courtesy fix on-site."
  ],
  "letter_draft": "Dear Valued Customer,\n\nThank you for submitting your warranty claim regarding your roofing installation from 2018. Our field technician visited your site and conducted a thorough evaluation. While our technician performed an immediate courtesy fix on your flashing during the visit, our review of your warranty agreement (Section 4.2) indicates that flashing separation resulting from third-party installation practices falls outside formal warranty coverage. We remain committed to supporting you and hope the on-site adjustment resolves your immediate concern.\n\nSincerely,\nWarranty Management Team"
}
```

### Example 2: Fully Warrantable Claim (Approved)

#### Input Prompt

```json
{
  "system_role": "You are an expert technical warranty analyst for a roofing material manufacturer. Analyze the provided field site visit report against the attached customer warranty text. Use Chain-of-Thought reasoning to determine if the reported issue is covered under the warranty terms. Draft a response letter following corporate guidelines. If information is ambiguous, flag the claim for human inspection. Output must adhere strictly to the required Pydantic JSON schema.",
  "user": {
    "site_report": "Field Tech Notes: Severe premature thermal cracking and blistering across 40% of southern exposure shingles. Roof installed 2021. Installation meets all regional code guidelines and manufacturer guidelines.",
    "warranty_excerpt": "Section 2.1: Full replacement coverage for material blistering or cracking resulting from manufacturing defects within 10 years of installation date."
  }
}
```

#### Expected Output (JSON Schema Compliance)

```json
{
  "claim_id": "CLM-2026-9104",
  "warrantability_status": "APPROVED",
  "confidence_score": 0.98,
  "policy_clause_referenced": "Section 2.1 (Thermal Cracking & Blistering Manufacturing Defect)",
  "reasoning_steps": [
    "1. Field report notes thermal cracking and blistering within 5 years of 2021 installation.",
    "2. Installation verified as correct and compliant with guidelines.",
    "3. Section 2.1 guarantees full replacement coverage for manufacturing defects within 10 years."
  ],
  "letter_draft": "Dear Valued Customer,\n\nWe have reviewed the site inspection report for your claim regarding the roofing materials installed in 2021. Based on the technician's findings and in accordance with Section 2.1 of your warranty agreement, we have approved your claim for material replacement due to thermal cracking.\n\nOur customer support team will reach out within two business days to coordinate contractor dispatch and material shipment.\n\nSincerely,\nWarranty Management Team"
}
```

### Example 3: Non-Happy Path / Edge Case (Ambiguous Report Flagged for Human Review)

#### Input Prompt

```json
{
  "system_role": "You are an expert technical warranty analyst for a roofing material manufacturer. Analyze the provided field site visit report against the attached customer warranty text. Use Chain-of-Thought reasoning to determine if the reported issue is covered under the warranty terms. Draft a response letter following corporate guidelines. If information is ambiguous, flag the claim for human inspection. Output must adhere strictly to the required Pydantic JSON schema.",
  "user": {
    "site_report": "Field Tech Notes: Customer complains of water leaks in master bedroom ceiling. Inspected roof surface; found storm debris and missing shingles on south slope. Unclear if damage was caused by recent hail event or normal wear.",
    "warranty_excerpt": "Section 5.0: Excludes coverage for acts of God, extreme weather events (including hail storms), and wind damage exceeding 60mph."
  }
}
```

#### Expected Output (JSON Schema Compliance)

```json
{
  "claim_id": "CLM-2026-9331",
  "warrantability_status": "REQUIRES_HUMAN_INSPECTION",
  "confidence_score": 0.42,
  "policy_clause_referenced": "Section 5.0 (Acts of God / Weather Exclusion)",
  "reasoning_steps": [
    "1. Field tech noted leaks and missing shingles, but explicitly stated origin (wear vs. recent weather event) is undetermined.",
    "2. Section 5.0 excludes hail damage, but covers material failure under normal weather conditions.",
    "3. Missing critical weather data to make a definitive policy determination. Automated determination stopped."
  ],
  "letter_draft": "[AUTOMATED DRAFT PAUSED]: Claim flagged for manual review due to ambiguous site report regarding storm damage vs. material failure. Please inspect technician photos or request secondary field assessment."
}
```

## 6. Evaluation Strategy

To ensure the system maintains high accuracy without drifting over time, we will implement:

* **Ground-Truth Dataset:** A curated test suite of 100 historical claims (with verified human-approved letters and correct warrantability determinations).

* **Automated Evaluation Metrics:** Running automated test suites using LLM-as-a-judge frameworks to measure:

  * *Factual Alignment:* Did the model correctly cite the applicable warranty section?

  * *Tone and Compliance:* Does the generated letter match corporate communication standards and tone?

* **Continuous Iteration:** Reviewing failed test cases weekly to refine prompt templates and adjust RAG chunking parameters.

## 7. Observability Plan

* **Tracing:** Use observability platforms (e.g., LangSmith) to trace every incoming claim request, tracking token consumption, latency, and step-by-step chain execution.

* **Prompt Versioning:** Maintain strict version control on prompt templates to roll back changes immediately if error rates spike.

* **Error Tracking & Logging:** Log validation failures (e.g., when Pydantic schema validation catches an invalid output format) and human-in-the-loop rejection reasons to identify systematic edge cases.

## 8. Risks, Limitations, and Mitigation Strategies

* **Hallucination of Warranty Terms:**

  * *Risk:* The LLM might misinterpret complex legal wording in older warranty documents.

  * *Mitigation:* Strict RAG grounding with source citations required in every output, paired with mandatory human review before customer dispatch.

* **Edge-Case Complexity:**

  * *Risk:* Highly unusual weather or structural damage claims may confuse automated logic.

  * *Mitigation:* Confidence scoring thresholds; low-confidence claims are automatically flagged for senior warranty manager review rather than auto-drafting.

* **Data Privacy & Salesforce Security:**

  * *Risk:* Sensitive customer PII sent to LLM APIs.

  * *Mitigation:* Anonymize customer identifiers in prompts or deploy secure, enterprise-compliant LLM API endpoints with strict data retention agreements.

## 9. Future Improvements & Expansion Ideas

* **Multimodal Vision Analysis:** Integrate vision-capable LLMs (e.g., GPT-4o or Claude 3.5 Sonnet) directly into the processing pipeline to analyze field technician photographs alongside text reports, automatically detecting roof damage patterns (e.g., hail impact vs. installation tear).

* **Automated High-Confidence Dispatch:** Implement a zero-touch dispatch path for simple, high-confidence courtesy-fix or standard denial scenarios (e.g., confidence score $>0.95$), bypassing human review to send responses within minutes.

* **Multilingual Letter Generation:** Automatically translate warranty response letters into Spanish or French based on the customer's preferred language in Salesforce, expanding accessibility across North American markets.

* **Predictive Defect Analytics:** Aggregate structured reasoning outputs across thousands of claims to provide R&D and manufacturing teams with early warning signals on specific product lines or regional contractor installation issues.