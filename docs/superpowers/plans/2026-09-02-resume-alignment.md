# Resume Alignment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Align the public homepage dates and selected projects with Daniel's latest approved resume details.

**Architecture:** Make targeted Markdown content edits in the existing Jekyll pages. Preserve unpublished research, the downloadable PDF, site structure, and all unrelated entries.

**Tech Stack:** Jekyll, Markdown, HTML, Bundler

---

### Task 1: Align public homepage dates

**Files:**
- Modify: `_pages/about.md`

- [ ] **Step 1: Confirm the old public values exist**

Run: `rg -n "December 2027|Sep\. 2026 - Dec\. 2027|Jan\. 2026 - Mar\. 2026|Mar\. 2025 - Sep\. 2025|Sep\. 2022 - Dec\. 2025" _pages/about.md`

Expected: all five old values are present before editing.

- [ ] **Step 2: Apply the approved replacements**

Replace the five values with `January 2028`, `Aug. 2026 - Jan. 2028`, `Dec. 2025 - Mar. 2026`, `Jun. 2025 - Sep. 2025`, and `Sep. 2022 - Jan. 2026` respectively. Do not edit the Georgia Tech or NYU research entries.

- [ ] **Step 3: Verify homepage values**

Run: `rg -n "January 2028|Aug\. 2026 - Jan\. 2028|Dec\. 2025 - Mar\. 2026|Jun\. 2025 - Sep\. 2025|Sep\. 2022 - Jan\. 2026" _pages/about.md`

Expected: all five approved values are present.

### Task 2: Align selected projects

**Files:**
- Modify: `_pages/projects.md`

- [ ] **Step 1: Replace Qwopus and add the two latest projects**

Use these three entries at the top of `Selected Projects`:

```markdown
- **Qwopus-Reasoning**: Post-trained Qwen through SFT on reasoning and math data, followed by GRPO with custom rewards and MTP integration; improved GPQA-Diamond accuracy by 6% and inference throughput by 11% over the base model.
- **AgenticRL-Long Horizon Coding Agent**: Post-trained Qwen3.5-4B with GSPO on multi-turn coding trajectories using execution-verified terminal rewards, improving single-rollout task success by 7% and pass@5 by 9% over the base model.
- **Tiny OneRec**: Built an LLM-based generative recommender by quantizing item text embeddings into Semantic IDs with an RQ-VAE and post-training Qwen through multi-task SFT for alignment and GRPO with constrained decoding and rank-aware rewards; achieved an HR@10 of 14.3% and an NDCG@10 of 10.4% on the Amazon Reviews Office Products dataset.
```

- [ ] **Step 2: Verify project metrics and retention**

Run: `rg -n "GPQA-Diamond accuracy by 6%|throughput by 11%|AgenticRL-Long Horizon Coding Agent|single-rollout task success by 7%|Tiny OneRec|HR@10 of 14\.3%|LLaMA 3\.2-Vision" _pages/projects.md`

Expected: all new metrics and one retained existing project are present.

### Task 3: Validate the completed change

**Files:**
- Verify: `_pages/about.md`
- Verify: `_pages/projects.md`

- [ ] **Step 1: Run the site build**

Run: `bundle exec jekyll build`

Expected: Jekyll exits successfully without content or Liquid errors.

- [ ] **Step 2: Inspect the final diff**

Run: `git diff --check` and `git diff -- _pages/about.md _pages/projects.md`

Expected: no whitespace errors; only the approved public content changes appear.

- [ ] **Step 3: Commit the implementation**

```bash
git add _pages/about.md _pages/projects.md docs/superpowers/plans/2026-09-02-resume-alignment.md
git commit -m "Align website content with latest resume"
```
