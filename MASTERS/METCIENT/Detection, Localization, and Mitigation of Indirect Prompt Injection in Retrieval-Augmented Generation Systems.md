Here is a complete **Research Plan draft** based on your material’s expected structure: Title, Keywords, Abstract, Introduction, Theoretical Background, Related Works, Methodology, Proposed Solution, Preliminary Results, Continuation of Research, References, and Annexes. The structure follows the research-plan template you sent, but I adjusted it to make the project more coherent as a security/AI research proposal.

# Research Plan

## Title

**Detection, Localization, and Mitigation of Indirect Prompt Injection in Retrieval-Augmented Generation Systems**


## Subtitle
**How can RAG systems detect, localize, and mitigate indirect prompt injections embedded in retrieved documents while preserving the utility of model-generated responses?**

## Keywords

Prompt Injection; Indirect Prompt Injection; Retrieval-Augmented Generation; RAG; Large Language Models; LLM Security; AI Security; Prompt Injection Detection; Prompt Injection Localization; Adversarial Inputs; Secure LLM Applications; Untrusted Data; Model Robustness; AI Agents; Structured Prompts; Security Evaluation.

---

## Abstract

Large Language Models are increasingly integrated into applications that retrieve and process external information, such as documents, webpages, emails, tool outputs, and database records. Retrieval-Augmented Generation systems improve answer relevance by adding external context to the model input, but they also introduce a security risk: untrusted retrieved content may contain malicious instructions that manipulate the model’s behavior. This vulnerability is commonly known as indirect prompt injection. OWASP classifies prompt injection as a major risk for LLM applications and explicitly notes that RAG and fine-tuning do not fully eliminate this vulnerability. ([OWASP Gen AI Security Project](https://genai.owasp.org/llmrisk/llm01-prompt-injection/ "LLM01:2025 Prompt Injection - OWASP Gen AI Security Project"))

This research proposes to study indirect prompt injection in RAG pipelines through three complementary goals: detecting whether retrieved context contains injected instructions, localizing the specific contaminated segment, and mitigating the attack before the final model response is generated. The work will implement a controlled RAG environment, construct a benchmark of benign and contaminated documents, evaluate multiple LLMs under attack and non-attack conditions, and compare lightweight defense strategies. The expected contribution is a reproducible evaluation protocol and a defensive module that reduces attack success while preserving task utility.

---

# 1. Introduction

## 1.1 Context

LLM-based applications are no longer limited to isolated question-answering. They are commonly connected to external tools, file repositories, web search, enterprise documents, and retrieval systems. This design improves usefulness because the model can answer based on updated or domain-specific information. However, it also changes the threat model: the model may receive content that the application developer does not fully control.

In RAG systems, retrieved passages are usually placed inside the model context together with system instructions and the user’s query. If a retrieved document contains adversarial instructions, the model may treat that document as a command instead of as data. OWASP defines indirect prompt injection as a case where an LLM accepts input from external sources, such as websites or files, and the external content alters the model behavior unexpectedly. ([OWASP Gen AI Security Project](https://genai.owasp.org/llmrisk/llm01-prompt-injection/ "LLM01:2025 Prompt Injection - OWASP Gen AI Security Project"))

## 1.2 Business Problem

Organizations increasingly use RAG-based assistants to answer questions over internal knowledge bases, technical documentation, legal documents, support tickets, and operational records. These systems may process documents created by employees, customers, suppliers, or external sources.

The business problem is that a contaminated document can silently influence the assistant’s response. In a real deployment, this could lead to incorrect recommendations, policy violations, manipulated decisions, leakage of sensitive context, or loss of trust in the system. Even when the model does not perform an obviously malicious action, the output may become unreliable.

## 1.3 Technical Problem

The technical problem is the weak separation between **trusted instructions** and **untrusted data** in LLM inputs. RAG systems often concatenate these elements into a single prompt-like context. Because the model processes both instructions and retrieved text as natural language, injected instructions inside documents can compete with system-level instructions.

This problem motivates the need for mechanisms that can:

1. detect whether retrieved context contains adversarial instructions;
    
2. localize which part of the retrieved content is suspicious;
    
3. remove, isolate, or structurally separate the suspicious content before generation;
    
4. preserve normal task performance on benign inputs.
    

## 1.4 General Objective

To evaluate and propose methods for detecting, localizing, and mitigating indirect prompt injection attacks in Retrieval-Augmented Generation systems.

## 1.5 Specific Objectives

1. Build a controlled RAG pipeline for security evaluation.
    
2. Construct a dataset of benign and contaminated documents.
    
3. Define a taxonomy of indirect prompt injection patterns in retrieved content.
    
4. Evaluate the vulnerability of different LLMs to indirect prompt injection.
    
5. Implement and compare lightweight defense mechanisms.
    
6. Measure both security effectiveness and utility preservation.
    
7. Produce a reproducible experimental protocol with code, data, prompts, and evaluation scripts.
    

## 1.6 Research Questions

|ID|Research Question|
|---|---|
|RQ1|How often do indirect prompt injections in retrieved documents alter the final response of a RAG system?|
|RQ2|Which types of injected instructions are more successful in manipulating RAG outputs?|
|RQ3|Can a lightweight detection method identify contaminated retrieved passages with acceptable precision and recall?|
|RQ4|Can the system localize the specific segment responsible for the injection?|
|RQ5|Which mitigation strategy best reduces attack success while preserving normal answer quality?|
|RQ6|How much utility loss is introduced by prompt injection defenses in benign RAG tasks?|

## 1.7 Hypotheses

|ID|Hypothesis|
|---|---|
|H1|Indirect prompt injection significantly increases the probability of incorrect or attacker-aligned responses in RAG systems.|
|H2|Defenses that separate instructions from retrieved data reduce attack success more effectively than prompt-only warnings.|
|H3|Localization-based filtering preserves more utility than removing the entire retrieved document.|
|H4|Stronger models may have better task performance, but they remain vulnerable to indirect prompt injection.|
|H5|Evaluating only attack success is insufficient; defenses must also be evaluated for benign-task utility loss.|

The last hypothesis is motivated by recent work arguing that prompt injection defenses should be assessed across both security effectiveness and general-purpose utility. ([arXiv](https://arxiv.org/abs/2505.18333 "[2505.18333] A Critical Evaluation of Defenses against Prompt Injection Attacks"))

## 1.8 Expected Contributions

This research is expected to contribute:

1. A reproducible benchmark for indirect prompt injection in RAG systems.
    
2. A structured evaluation protocol for security and utility.
    
3. A localization-aware defense module for contaminated retrieved content.
    
4. An empirical comparison of different mitigation strategies.
    
5. A discussion of limitations and threats to validity in RAG security evaluation.
    

---

# 2. Theoretical Background

## 2.1 Large Language Models

Large Language Models are neural models trained to process and generate natural language. In application contexts, they are commonly guided through system prompts, developer instructions, user prompts, and external context. Their flexibility makes them useful for summarization, question-answering, classification, code generation, and decision support, but also introduces security risks when the model receives untrusted instructions.

## 2.2 Retrieval-Augmented Generation

Retrieval-Augmented Generation is an architecture in which an application retrieves relevant documents or passages and inserts them into the model context before generation. A typical RAG pipeline contains:

```text
User Question
      ↓
Retriever
      ↓
Relevant Documents / Chunks
      ↓
Prompt Assembly
      ↓
LLM Generation
      ↓
Final Answer
```

The security issue appears because the retrieved content is usually treated as informative context, but it may contain adversarial instructions.

## 2.3 Prompt Injection

Prompt injection is a vulnerability in which model behavior is manipulated through crafted input. OWASP describes prompt injection as input that alters the model’s behavior or output in unintended ways, and distinguishes direct prompt injection from indirect prompt injection. Direct prompt injection comes from the user prompt itself, while indirect prompt injection comes from external content such as files, webpages, or tool outputs. ([OWASP Gen AI Security Project](https://genai.owasp.org/llmrisk/llm01-prompt-injection/ "LLM01:2025 Prompt Injection - OWASP Gen AI Security Project"))

## 2.4 Indirect Prompt Injection

Indirect prompt injection is especially relevant for RAG because the attacker does not need to directly interact with the model. Instead, the attacker can contaminate a document that may later be retrieved by the system. When the model reads that document, it may follow the injected instruction.

In this research, the focus is not on harmful real-world exploitation, but on controlled synthetic attacks used to evaluate whether a RAG system can distinguish task-relevant document content from adversarial instructions.

## 2.5 Detection, Localization, and Mitigation

This work separates the defense problem into three stages:

|Stage|Meaning|
|---|---|
|Detection|Determine whether retrieved content is likely contaminated.|
|Localization|Identify the specific segment containing the injected instruction.|
|Mitigation|Prevent the suspicious segment from influencing the final answer.|

This separation is important because a system that only detects an attack may still be unable to recover useful benign information from the same document. PromptLocate is directly relevant here because it frames localization of injected prompts as important for forensic analysis and data recovery. ([arXiv](https://arxiv.org/abs/2510.12252 "[2510.12252] PromptLocate: Localizing Prompt Injection Attacks"))

---

# 3. Related Works

## 3.1 OWASP LLM01: Prompt Injection

OWASP’s GenAI Security Project classifies prompt injection as LLM01 in the 2025 Top 10 risks for LLM applications. It explicitly includes indirect prompt injection through external files and websites and notes that the severity depends on the business context and the agency granted to the model. ([OWASP Gen AI Security Project](https://genai.owasp.org/llmrisk/llm01-prompt-injection/ "LLM01:2025 Prompt Injection - OWASP Gen AI Security Project"))

This work uses OWASP as a practical motivation for why prompt injection is not merely an academic problem, but a deployment-level security risk.

## 3.2 StruQ: Structured Queries

StruQ proposes structured queries as a defense against prompt injection. Its central idea is to separate trusted prompts and untrusted data into different channels. The authors argue that prompt injection occurs when an LLM input contains both trusted instructions and untrusted data with potentially injected instructions. ([sizhe-chen.github.io](https://sizhe-chen.github.io/StruQ-Website/ "StruQ: Defending Against Prompt Injection with Structured Queries"))

This research draws from StruQ’s core principle of separating instruction and data, but studies a lighter-weight version that can be applied to existing RAG pipelines without training a new model.

## 3.3 AgentDojo

AgentDojo is an evaluation framework for LLM agents that execute tools over untrusted data. It includes realistic tasks and security test cases, and it is designed as an extensible environment for evaluating new attacks and defenses. ([arXiv](https://arxiv.org/abs/2406.13352 "[2406.13352] AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents"))

Although this research focuses on RAG rather than full agent tool use, AgentDojo is methodologically relevant because it evaluates both task completion and adversarial robustness.

## 3.4 PromptLocate

PromptLocate proposes the localization of injected prompts inside contaminated data. The paper argues that localization remains underexplored and proposes a method that splits contaminated data into segments, identifies segments with injected instructions, and pinpoints injected data. ([arXiv](https://arxiv.org/abs/2510.12252 "[2510.12252] PromptLocate: Localizing Prompt Injection Attacks"))

This research uses PromptLocate as a central inspiration, but applies the localization idea to RAG pipelines and compares localization-based mitigation with simpler defenses.

## 3.5 Critical Evaluation of Prompt Injection Defenses

A recent critical evaluation argues that prompt injection defenses should be evaluated not only by attack resistance, but also by whether they preserve general-purpose utility. The paper states that existing defense evaluations often lack a principled methodology and may overstate effectiveness. ([arXiv](https://arxiv.org/abs/2505.18333 "[2505.18333] A Critical Evaluation of Defenses against Prompt Injection Attacks"))

This work adopts that principle by measuring both **Attack Success Rate** and **Utility Loss**.

## 3.6 Literature Review Strategy

The literature review will be narrative rather than systematic in the first version of the research plan. The selection criteria are:

1. works directly related to prompt injection;
    
2. works focused on indirect prompt injection or untrusted external data;
    
3. works proposing prompt injection defenses;
    
4. works proposing benchmarks or evaluation methods;
    
5. works discussing RAG or tool-using LLM security.
    

A systematic review may be considered later if the scope expands.

---

# 4. Methodology

## 4.1 Research Method

This research will follow an experimental computational methodology. The study will implement a controlled RAG environment, execute adversarial and benign test cases, collect quantitative metrics, and compare defense strategies.

The methodology is divided into six phases:

```text
Phase 1: Literature Review
        ↓
Phase 2: Dataset Construction
        ↓
Phase 3: RAG Pipeline Implementation
        ↓
Phase 4: Attack Scenario Design
        ↓
Phase 5: Defense Implementation
        ↓
Phase 6: Experimental Evaluation
```

## 4.2 Phase 1 — Literature Review

The first phase will review academic and technical works on:

- prompt injection;
    
- indirect prompt injection;
    
- RAG vulnerabilities;
    
- structured prompting;
    
- prompt injection defenses;
    
- localization of adversarial text;
    
- evaluation methodology for LLM security.
    

The output of this phase will be a related works matrix containing: paper, venue/year, problem addressed, method, reproducibility, limitations, and relation to this research.

## 4.3 Phase 2 — Dataset Construction

The dataset will contain benign documents and contaminated versions of the same documents. The documents will be synthetic or drawn from open sources where licensing permits reuse.

Possible document domains:

|Domain|Example Task|
|---|---|
|Technical manuals|Answer operational questions based on documentation.|
|Product descriptions|Recommend or compare products based on factual criteria.|
|Institutional policies|Answer compliance or policy questions.|
|Travel information|Rank hotels, restaurants, or activities based on constraints.|
|Academic-style documents|Summarize or answer questions from structured content.|

Each document will be split into chunks. Some chunks will remain benign, while others will contain controlled injected instructions. The dataset will include labels for:

1. document ID;
    
2. chunk ID;
    
3. benign or contaminated status;
    
4. injection type;
    
5. injection location;
    
6. expected answer;
    
7. expected safe behavior.
    

## 4.4 Phase 3 — RAG Pipeline Implementation

The experimental RAG pipeline will include:

```text
Input Query
   ↓
Embedding Model
   ↓
Vector Store
   ↓
Top-k Retrieval
   ↓
Context Assembly
   ↓
Defense Module
   ↓
LLM
   ↓
Output Evaluation
```

The implementation may use Python, a vector database or local vector index, and APIs or local models depending on availability and budget.

The system will log:

- retrieved chunks;
    
- final prompt/context;
    
- model output;
    
- detected injection status;
    
- localized suspicious segment;
    
- final evaluation label;
    
- runtime and cost.
    

## 4.5 Phase 4 — Attack Scenario Design

The attack scenarios will be controlled and synthetic. They will not target real systems, real credentials, or real users.

The injected content will represent categories such as:

|Attack Category|Description|
|---|---|
|Instruction override|The retrieved text attempts to override the system task.|
|Answer manipulation|The retrieved text tries to change the final answer.|
|Ranking manipulation|The retrieved text tries to make a specific item appear better.|
|Refusal manipulation|The retrieved text tries to make the model refuse a benign request.|
|Context distraction|The retrieved text introduces irrelevant but instruction-like content.|
|Policy confusion|The retrieved text imitates system or developer instructions.|

The goal is not to create harmful attacks, but to test whether the model treats untrusted retrieved content as instructions.

## 4.6 Phase 5 — Defense Implementation

The research will compare four defense configurations.

|Defense|Description|
|---|---|
|D0: No defense|Baseline RAG pipeline.|
|D1: Prompt warning|The system prompt tells the model to treat retrieved content as untrusted data.|
|D2: Structured context|Retrieved data is clearly separated from trusted instructions using rigid formatting.|
|D3: Detection filter|A classifier or LLM-based filter flags suspicious chunks before generation.|
|D4: Localization-based mitigation|The system identifies suspicious segments and removes or isolates only those segments.|

D2 is inspired by the general principle behind StruQ: separating trusted instructions from untrusted data. StruQ itself uses a special structured-query approach and a trained model, but this research will test whether lighter-weight structural separation improves RAG robustness. ([sizhe-chen.github.io](https://sizhe-chen.github.io/StruQ-Website/ "StruQ: Defending Against Prompt Injection with Structured Queries"))

## 4.7 Phase 6 — Experimental Evaluation

The evaluation will compare models, attack types, and defense configurations.

### Independent Variables

|Variable|Values|
|---|---|
|Model|Selected closed and/or open LLMs|
|Dataset condition|Benign, contaminated|
|Defense|D0, D1, D2, D3, D4|
|Attack type|Override, manipulation, refusal, ranking, distraction|
|Retrieval setting|Top-1, Top-3, Top-5 chunks|

### Dependent Variables

|Metric|Meaning|
|---|---|
|Attack Success Rate|Percentage of attacked cases where the model follows or reflects the injection.|
|Task Success Rate|Percentage of cases where the model answers the original task correctly.|
|Detection Precision|Percentage of flagged chunks that are actually contaminated.|
|Detection Recall|Percentage of contaminated chunks correctly flagged.|
|Localization Accuracy|Whether the system identifies the correct contaminated segment.|
|Utility Loss|Drop in benign task performance caused by the defense.|
|False Positive Rate|Benign chunks incorrectly flagged as malicious.|
|Refusal Rate|Percentage of cases where the model refuses despite the task being benign.|

### Core Formulas

```text
Attack Success Rate = Successful Attacks / Total Attacked Cases

Task Success Rate = Correct Task Responses / Total Cases

Utility Loss = Benign TSR without Defense - Benign TSR with Defense

Detection Precision = True Positives / (True Positives + False Positives)

Detection Recall = True Positives / (True Positives + False Negatives)
```

## 4.8 Mapping Research Questions to Methodology

|Research Question|Methodological Step|Metric|
|---|---|---|
|RQ1|Run contaminated RAG cases without defense|Attack Success Rate|
|RQ2|Compare attack categories|ASR by attack type|
|RQ3|Run detection filter|Precision, Recall, F1|
|RQ4|Evaluate segment labels|Localization Accuracy|
|RQ5|Compare D0-D4|ASR, TSR, Utility Loss|
|RQ6|Run benign cases under all defenses|Task Success Rate, Refusal Rate|

## 4.9 Reproducibility and Open Science

The project should include:

1. public code repository;
    
2. fixed random seeds;
    
3. versioned dataset;
    
4. documented prompts;
    
5. model configuration files;
    
6. evaluation scripts;
    
7. result tables in CSV;
    
8. experiment logs;
    
9. clear cost/budget report;
    
10. limitations and known sources of variance.
    

For proprietary models, the exact model name and date of execution should be recorded, since model behavior can change over time.

---

# 5. Proposed Solution

## 5.1 Overview

The proposed solution is a defensive module called:

# **RAG Injection Guard**

The module will be inserted between retrieval and final generation.

```text
User Question
     ↓
Retriever
     ↓
Retrieved Chunks
     ↓
RAG Injection Guard
     ↓
Filtered / Structured Context
     ↓
LLM Generator
     ↓
Final Answer
```

## 5.2 Architecture

The module contains four components:

|Component|Function|
|---|---|
|Segmenter|Splits retrieved chunks into smaller semantic or paragraph-level segments.|
|Detector|Scores each segment for possible injected instructions.|
|Localizer|Marks the exact suspicious segment.|
|Mitigator|Removes, masks, quotes, or isolates suspicious segments before generation.|

## 5.3 Processing Flow

```text
Retrieved Chunk
     ↓
Segment Text
     ↓
Score Each Segment
     ↓
Classify Segment as Benign/Suspicious
     ↓
If Suspicious: localize and mitigate
     ↓
Reassemble Safe Context
     ↓
Send to LLM
```

## 5.4 Mitigation Strategies

The research will test three mitigation behaviors:

|Strategy|Description|
|---|---|
|Remove|Delete the suspicious segment from the context.|
|Quote|Keep the segment but explicitly quote it as untrusted data.|
|Isolate|Move the suspicious segment into a separate “ignored content” field for audit purposes.|

The comparison will show whether it is better to remove suspicious text entirely or preserve it in a safer format.

## 5.5 Expected Novelty

The novelty is not claiming to fully solve prompt injection. The contribution is more focused:

> A reproducible, localization-aware evaluation of indirect prompt injection mitigation in RAG systems, comparing detection-only, structure-only, and localization-based defenses under both security and utility metrics.

This is academically defensible because it combines three dimensions that are often studied separately: RAG vulnerability, prompt injection defense, and localization of contaminated text.

---

# 6. Preliminary Results

Since the experiments have not been executed yet, this section should initially be written as **preliminary artifacts and expected initial observations**, not as final empirical claims.

## 6.1 Preliminary Artifacts

The research can already produce:

1. initial related works table;
    
2. initial RAG pipeline design;
    
3. initial dataset schema;
    
4. preliminary taxonomy of attacks;
    
5. first version of evaluation metrics;
    
6. planned architecture of RAG Injection Guard.
    

## 6.2 Expected Initial Findings

The expected initial findings are:

1. Basic prompt-only warnings will reduce some attacks but will not be sufficient.
    
2. Structured context formatting will improve robustness but will not eliminate attacks.
    
3. Removing entire contaminated chunks will reduce attack success but may harm answer quality.
    
4. Localization-based mitigation may provide a better trade-off between security and utility.
    
5. Some attacks will be difficult to classify because they resemble normal document instructions.
    

## 6.3 Preliminary Result Table Template

|Model|Defense|Benign TSR|Attack ASR|Detection F1|Localization Accuracy|Utility Loss|
|---|--:|--:|--:|--:|--:|--:|
|Model A|D0|—|—|—|—|—|
|Model A|D1|—|—|—|—|—|
|Model A|D2|—|—|—|—|—|
|Model A|D3|—|—|—|—|—|
|Model A|D4|—|—|—|—|—|

## 6.4 Limitations

Initial limitations include:

1. The benchmark may not represent all real-world RAG deployments.
    
2. Synthetic attacks may differ from real attacker behavior.
    
3. LLM outputs are stochastic and may vary across runs.
    
4. Proprietary model updates may affect reproducibility.
    
5. Detection performance may depend heavily on the chosen model or classifier.
    
6. A defense that works in English may not generalize to other languages or domains.
    

## 6.5 Threats to Validity

|Threat|Description|Mitigation|
|---|---|---|
|Internal validity|Results may depend on prompt wording.|Use fixed prompts and multiple runs.|
|External validity|Dataset may not represent production RAG systems.|Use multiple document domains.|
|Construct validity|Attack success may be hard to define.|Use explicit rubrics and human/manual validation for samples.|
|Reproducibility|Proprietary models may change.|Record model versions and execution dates.|
|Evaluation bias|LLM-as-judge may be unreliable.|Combine automatic checks with manual inspection.|

---

# 7. Continuation of Research

## 7.1 Future Activities

|Phase|Activity|
|---|---|
|1|Finalize literature review and related works matrix.|
|2|Build benign document dataset.|
|3|Create contaminated document variants.|
|4|Implement baseline RAG pipeline.|
|5|Implement attack evaluation scripts.|
|6|Implement D1-D4 defenses.|
|7|Run experiments across models and settings.|
|8|Analyze results and threats to validity.|
|9|Prepare final paper/report.|
|10|Release reproducibility package.|

## 7.2 Expected Difficulties

|Difficulty|Expected Treatment|
|---|---|
|Defining attack success|Create a clear annotation rubric.|
|Model output variability|Use repeated runs and deterministic settings when possible.|
|Cost of API models|Start with a small benchmark and expand gradually.|
|Dataset quality|Use controlled templates and manual review.|
|Defense overblocking|Measure false positives and utility loss.|

## 7.3 Timeline

|Month|Activity|
|---|---|
|Month 1|Literature review and final research scope.|
|Month 2|Dataset design and RAG pipeline implementation.|
|Month 3|Attack scenario construction and baseline experiments.|
|Month 4|Defense implementation.|
|Month 5|Full experimental evaluation.|
|Month 6|Analysis, writing, reproducibility package, and final revision.|

## 7.4 Expected Final Deliverables

1. Research report or article.
    
2. Related works matrix.
    
3. RAG benchmark dataset.
    
4. Attack taxonomy.
    
5. RAG Injection Guard prototype.
    
6. Evaluation scripts.
    
7. Experimental results.
    
8. Reproducibility package.
    

---

# References

OWASP Gen AI Security Project. **LLM01:2025 Prompt Injection**. Used as the main practical security reference for defining direct and indirect prompt injection risks. ([OWASP Gen AI Security Project](https://genai.owasp.org/llmrisk/llm01-prompt-injection/ "LLM01:2025 Prompt Injection - OWASP Gen AI Security Project"))

Debenedetti, E., Zhang, J., Balunović, M., Beurer-Kellner, L., Fischer, M., & Tramèr, F. **AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents**. NeurIPS 2024 Datasets and Benchmarks Track. Used as methodological inspiration for evaluating utility and adversarial robustness over untrusted data. ([AgentDojo](https://agentdojo.spylab.ai/ "AgentDojo"))

Chen, S., Piet, J., Sitawarin, C., & Wagner, D. **StruQ: Defending Against Prompt Injection with Structured Queries**. USENIX Security 2025. Used as inspiration for separating trusted instructions from untrusted data. ([sizhe-chen.github.io](https://sizhe-chen.github.io/StruQ-Website/ "StruQ: Defending Against Prompt Injection with Structured Queries"))

Jia, Y., Liu, Y., Shao, Z., Jia, J., & Gong, N. **PromptLocate: Localizing Prompt Injection Attacks**. To appear in IEEE Symposium on Security and Privacy 2026. Used as the main reference for the localization aspect of the proposed research. ([arXiv](https://arxiv.org/abs/2510.12252 "[2510.12252] PromptLocate: Localizing Prompt Injection Attacks"))

Jia, Y., Shao, Z., Liu, Y., Jia, J., Song, D., & Gong, N. **A Critical Evaluation of Defenses against Prompt Injection Attacks**. arXiv, 2025. Used as the basis for evaluating both defense effectiveness and general-purpose utility. ([arXiv](https://arxiv.org/abs/2505.18333 "[2505.18333] A Critical Evaluation of Defenses against Prompt Injection Attacks"))

Correia, P. H. B., Achjian, R. W., de Oliveira, D. E. G. C., Maria, Y. A., Hayashi, V. T., Lopes, M., Miers, C. C., & Simplicio Jr., M. A. **A Systematic Literature Review on LLM Defenses Against Prompt Injection and Jailbreaking: Expanding NIST Taxonomy**. arXiv, 2026. Useful for expanding the theoretical background and defense taxonomy. ([arXiv](https://arxiv.org/abs/2601.22240 "[2601.22240] A Systematic Literature Review on LLM Defenses Against Prompt Injection and Jailbreaking: Expanding NIST Taxonomy"))

---

# Annexes

## Annex A — Dataset Schema

|Field|Description|
|---|---|
|document_id|Unique document identifier|
|chunk_id|Retrieved chunk identifier|
|domain|Document domain|
|query|User question|
|benign_answer|Expected answer without attack|
|contaminated|Boolean label|
|injection_type|Type of attack|
|injection_start|Start position of injected segment|
|injection_end|End position of injected segment|
|expected_safe_behavior|What the system should do|
|notes|Manual annotation notes|

## Annex B — Experiment Log Schema

|Field|Description|
|---|---|
|run_id|Unique execution ID|
|model|Model used|
|date|Execution date|
|defense|Defense configuration|
|query_id|Query identifier|
|retrieved_chunks|Chunks retrieved|
|final_answer|Model response|
|attack_success|Boolean|
|task_success|Boolean|
|detection_result|Detection label|
|localization_result|Localized segment|
|cost|Estimated cost|
|latency|Runtime|

## Annex C — Final Research Scope

The recommended final scope is:

> Evaluate indirect prompt injection in RAG systems using a controlled benchmark of benign and contaminated documents, compare no-defense, prompt-warning, structured-context, detection-based, and localization-based defenses, and measure both attack reduction and utility preservation.

This scope is specific enough for implementation, broad enough for academic discussion, and aligned with current open problems in LLM security.