---
name: research-paper-writing
description: Use when drafting, reviewing, rewriting, or polishing a research paper for a conference, workshop, journal, or internal academic review; preparing manuscript sections, claims, citations, methodology, results, figures, tables, threats to validity, anonymized review versions, or camera-ready academic papers.
---

# Research Paper Writing Skill

Use this skill when drafting, reviewing, rewriting, or polishing a research paper for a conference, workshop, journal, or internal academic review.

## Goal

Write a standalone scientific manuscript that reads like work produced by domain researchers for peer review. The paper must communicate:

- the problem,
- the gap in prior work,
- the method,
- the evidence,
- the results,
- the limitations,
- and the contribution.

The manuscript must not read like a report about how the document was generated.

## Core principles

- State claims precisely.
- Separate measured results from interpretation.
- Keep scope boundaries clear without sounding defensive.
- Cite external ideas, facts, definitions, methods, and prior results.
- Do not cite sources that do not support the sentence.
- Do not mention the target venue, writing tools, AI tools, build process, plotting packages, or internal files in the paper body.
- Keep source/build details in comments, scripts, README files, or artifact documentation, not in the manuscript text.

## Source hierarchy

Prefer sources in this order:

1. The authors' own data, experiments, code, and measurements for reported results.
2. Official documentation, standards, specifications, white papers, technical reports, audits, or repositories for system-specific claims.
3. Peer-reviewed papers and strong preprints for theory, related work, risks, methods, and empirical findings.
4. Official tool or infrastructure documentation only when the tool is part of the research method.
5. Blogs, dashboards, tutorials, and news only for context, never for core scientific claims unless no stronger source exists.

## What belongs in the paper body

Include:

- problem motivation and research gap,
- research questions or hypotheses,
- related work and how this paper differs,
- background needed to understand the contribution,
- experimental setup or method,
- variables, assumptions, scenarios, and metrics,
- tools that are part of the scientific method,
- tables and figures showing research content,
- results scoped to the evaluated setting,
- interpretation supported by data and citations,
- threats to validity,
- conclusion and future work.

## What must not appear in the paper body

Do not include document-production details, such as:

- target conference or venue name,
- statements justifying the topic through a venue call for papers,
- typesetting system or template decisions,
- plotting, drawing, or diagramming package names,
- "generated from CSV", "rendered by native figures", or similar build-process wording,
- internal file names or paths,
- build commands,
- script names used only to produce the manuscript,
- AI assistants, prompts, generated-text workflow, or editing process,
- citations to document-production tools.

Exception: mention a tool only when it is part of the scientific experiment itself, not merely part of writing, formatting, plotting, or packaging the paper.

## Venue and review mode

Use venue requirements only to configure the manuscript.

Allowed outside the paper body:

- template choice,
- page limits,
- anonymization settings,
- citation style,
- source comments,
- README notes,
- submission checklist.

Forbidden in the paper body:

- "This paper is written for ..."
- "This fits the conference because ..."
- citations to the venue call for papers,
- explanations of the template or formatting tools.

For double-blind review:

- anonymize authors and affiliations,
- remove acknowledgements that identify the authors,
- remove identifying artifact links if required,
- cite the authors' prior work neutrally in the third person,
- restore names, affiliations, acknowledgements, and links only in the camera-ready version when allowed.

## Recommended paper structure

1. **Abstract**
   - One paragraph.
   - Problem, method, setup, key results, contribution.
   - No build details, venue name, or tool details.

2. **Introduction**
   - Motivate the problem.
   - Identify the gap.
   - State the contribution and research questions.
   - Scope the work without over-apologizing.

3. **Related Work**
   - Group sources by theme.
   - Explain how the paper differs from prior work.
   - Cite each borrowed idea or result.

4. **Background**
   - Explain only what is needed to understand the method and results.
   - Use authoritative sources for factual system or theory claims.
   - Avoid marketing language.

5. **Methodology**
   - Describe setup, assumptions, variables, metrics, procedures, and exclusions.
   - Include enough detail for reproducibility.
   - Mention tools only if they are part of the research method.

6. **Results**
   - Report measurements directly.
   - Use tables and figures.
   - Separate observation from explanation.
   - Scope conclusions to the tested setting.

7. **Discussion**
   - Interpret results.
   - Connect evidence to mechanisms, design trade-offs, or theory.
   - Avoid global rankings unless the study supports them.

8. **Threats to Validity**
   - State technical boundaries.
   - Cover internal validity, external validity, construct validity, and reproducibility.

9. **Conclusion**
   - Summarize what was shown.
   - Restate the contribution.
   - Mention future work only when it follows from limitations.

## Claim and citation rules

Each factual claim must be one of:

- directly measured in this work,
- supported by a citation,
- clearly stated as an assumption,
- common background knowledge.

Cite close to the claim. Avoid placing one citation at the end of a long paragraph if it supports only part of the paragraph.

Use citations for:

- definitions,
- prior findings,
- system behavior,
- theoretical models,
- risk claims,
- historical claims,
- external datasets,
- official mechanisms,
- related-work summaries.

Do not cite:

- writing tools,
- plotting tools,
- templates,
- the target venue,
- sources that only partially or indirectly support the sentence.

Maintain bibliography hygiene:

- every citation key in the manuscript must exist in the bibliography,
- every bibliography entry should be cited unless intentionally included as suggested reading,
- prefer authoritative sources over weak contextual sources,
- remove unused references before submission.

## Plagiarism and paraphrasing

Avoid plagiarism of both words and ideas.

- Do not copy paragraphs from sources.
- Do not merely replace a few words from a source sentence.
- Do not reuse a source's structure without attribution.
- Cite external ideas even when paraphrased.
- Quote only when the exact wording matters.
- Use quotation marks and citations for exact wording.
- Write summaries from understanding, not by editing source sentences.

## Style rules

Prefer concise, direct academic prose.

Use:

- short sentences,
- one main idea per sentence,
- active voice for methods and results,
- precise nouns and verbs,
- scoped claims when needed,
- concrete metric definitions,
- direct statements of measured findings.

Avoid:

- defensive phrasing,
- ungrounded hedging,
- blog-like advice,
- marketing adjectives,
- overuse of "important", "useful", "mature", "clearly", "significant",
- repeated contrast formulas,
- meta-commentary about how the reader should interpret the text.

Do not remove necessary scientific caution. Hedges are allowed when they state uncertainty, scope, or conditions. Tie cautious language to a condition, dataset, metric, workload, threat, or limitation.

Prefer:

- "The measured ordering may change across workloads."
- "This estimate applies to the evaluated datasets."
- "The conclusion is limited by the sample size and metric definition."

Avoid:

- "The result might be different in some cases."
- "The method may not always work."
- "The finding is somewhat uncertain."

## Avoid template and metadiscourse formulations

Avoid patterns that announce importance, novelty, interpretation, or the writer's intent without adding technical content.

- "should not be read as"
- "should not be interpreted as"
- "this does not imply that"
- "not X, but rather Y"
- "it is important to note that"
- "in conclusion, it can be said that"
- "the result instead shows that"
- "still, the controlled test shows"
- "the main conclusion is not that ... rather ..."
- "this makes X useful"
- "in recent years, X has become increasingly important"
- "X plays a crucial role in ..."
- "this paper aims to ..."
- "this study provides valuable insights into ..."
- "the results clearly demonstrate ..."
- "it can be observed that ..."
- "this highlights the importance of ..."
- "this underscores the need for ..."
- "this paper fills an important gap ..."
- "to the best of our knowledge ..."
- "a comprehensive framework"
- "a novel approach"
- "future work could explore ..."
- "sheds light on ..."
- "utilize" when "use" is sufficient
- "leverage" when "use", "apply", or "build on" is more precise

Replace them with direct technical wording.

Replacement rules:

- Replace claims about importance with the concrete mechanism, trend, dependency, or consequence.
- Replace claims about novelty with the specific difference from prior work.
- Replace "aims to" with the research action: "measures", "compares", "evaluates", "analyzes", or "models".
- Replace "insights" with the specific result.
- Replace "clearly demonstrates" with measured values and scope.
- Replace "it can be observed that" with references to the evidence, such as "Table 2 reports ..." or "Figure 3 shows ...".
- Replace broad "gap" claims with the exact missing measurement, method, dataset, or comparison.
- Replace generic future-work sentences with extensions motivated by a stated limitation.

Examples:

Bad:

> This metric should not be interpreted as final profit.

Good:

> This metric measures pre-cost value and excludes execution costs, fees, and market-friction effects.

Bad:

> This does not imply that the method always performs best.

Good:

> Performance may change across datasets, parameter settings, and deployment conditions.

Bad:

> The main conclusion is not that one design dominates. Rather, each design has trade-offs.

Good:

> The results identify distinct trade-offs across accuracy, cost, complexity, and robustness.

## Results writing

Write results as scoped measurements, not universal truths.

Use:

- "In the tested setting, method A has the lowest measured cost."
- "Under this configuration, metric B increases by ..."
- "The measured ordering may change across datasets, environments, parameters, or workloads."

Avoid:

- "Method A is the best."
- "This proves that ..."
- "This is always cheaper/faster/safer."
- "Clearly, ..."

Recommended pattern:

1. Report the measured value.
2. Compare it to the relevant baseline.
3. Explain the likely mechanism, with citation if external.
4. State the scope boundary if needed.

## Methodology writing

The methodology should allow a reviewer to reproduce or audit the study.

Include:

- study setup,
- data or input selection,
- scenarios or treatments,
- control variables,
- measured metrics,
- computation method,
- exclusion criteria,
- assumptions,
- known limitations of the setup.

Do not include:

- how the manuscript figures were rendered,
- which package drew the diagram,
- internal data filenames,
- build scripts or PDF-generation details.

## Figures and tables

Figures and tables should present research content.

Captions should state:

- what is measured,
- under what setting,
- the main takeaway.

Captions must not mention:

- plotting packages,
- typesetting systems,
- internal filenames,
- image-generation scripts,
- "generated from file X",
- "rendered by tool Y".

Table rules:

- keep protocol, method, or model order consistent,
- include units in labels,
- use consistent number formatting,
- align comparable values,
- avoid tables that exceed page width,
- avoid duplicating all table values in prose.

Figure rules:

- ensure labels are readable,
- avoid overlapping text,
- keep figures within page width,
- use scientific labels, not implementation labels,
- inspect the compiled PDF visually.

## Source-file rules

The source may contain build details, but the compiled paper should not.

Allowed in source files:

- package imports,
- data-loading code,
- plotting code,
- comments for maintainers,
- build instructions,
- anonymization switches.

Forbidden in compiled manuscript text:

- document-production tool names,
- package names,
- file paths,
- build process descriptions,
- AI-assistance disclosures unless required by policy.

## Threats to validity

Write limitations as technical boundaries, not apologies.

Bad:

> The experiment is intentionally controlled. This makes the comparison clear, but it limits external validity.

Good:

> The controlled design improves internal comparability but limits external validity.

Bad:

> The paper therefore does not claim that the ordering is stable.

Good:

> The measured ordering may change across datasets, environments, parameter values, and workloads.

Consider:

- input selection,
- sample size,
- evaluation setting,
- implementation choices,
- measurement precision,
- construct validity of metrics,
- generalization beyond the tested setting,
- excluded real-world factors,
- reproducibility limits.

## Final audit checklist

Before submission or delivery, verify:

- The paper body does not mention the target venue.
- The paper body does not mention writing, plotting, formatting, AI, or build tools.
- Captions describe research content, not figure-generation process.
- Every factual claim is measured, cited, assumed, or common background.
- Every citation supports the exact sentence where it appears.
- No source text is copied or lightly paraphrased.
- All citation keys exist in the bibliography.
- No irrelevant or unused references remain.
- Results are scoped to the tested setting.
- Tables and figures use consistent labels, units, and ordering.
- The compiled PDF shows all figures and tables correctly.
- The review version is anonymized if required.
- The camera-ready version restores identity and acknowledgements only when allowed.

## Delivery checklist

When delivering final paper files:

- provide the main manuscript source,
- provide the bibliography,
- provide required data or artifact files,
- provide a buildable ZIP package if requested,
- exclude stale PDFs unless requested,
- state whether compilation was tested,
- if compilation was not tested, say exactly what was checked instead.
