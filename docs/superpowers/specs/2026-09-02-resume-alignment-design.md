# Resume Alignment Design

## Goal

Align the public homepage and projects page with the latest resume details explicitly approved by Daniel, without changing unpublished research content, the downloadable resume PDF, site layout, Georgia Tech research, or NYU research.

## Considered Approaches

1. **Targeted public-page update (selected):** Edit only the affected entries in `_pages/about.md` and `_pages/projects.md`. This minimizes unrelated changes and preserves the existing site structure.
2. **Resume-driven content generation:** Parse the resume and generate site entries automatically. This would reduce future duplication but is unnecessary for this small update and could overwrite site-only research content.
3. **Full resume/site synchronization:** Replace the downloadable PDF and reconcile every difference between the resume and website. This exceeds the approved scope because the PDF, unpublished research page, Georgia Tech RA, and NYU RA must remain unchanged.

## Approved Content Changes

### `_pages/about.md`

- Change the CMU graduation statement from December 2027 to January 2028.
- Change the CMU education dates from `Sep. 2026 - Dec. 2027` to `Aug. 2026 - Jan. 2028`.
- Change the Z.ai start date from January 2026 to December 2025.
- Change the UC San Diego dates from `Mar. 2025 - Sep. 2025` to `Jun. 2025 - Sep. 2025`.
- Change the NYU education end date from December 2025 to January 2026.
- Do not change the Georgia Tech or NYU research-assistant entries.

### `_pages/projects.md`

- Update Qwopus-Reasoning to the latest resume wording and metrics: GPQA-Diamond accuracy improves by 6%, and inference throughput improves by 11%.
- Add `AgenticRL-Long Horizon Coding Agent` using the latest resume wording and metrics.
- Add `Tiny OneRec` using the latest resume wording and metrics.
- Place the three current agent/model-training projects at the top of Selected Projects and retain all existing projects below them.

## Explicit Non-Goals

- Do not edit `_pages/research.md`, which is unpublished.
- Do not replace `files/daniel-guo-resume.pdf`.
- Do not change navigation, styling, layouts, blog content, Georgia Tech RA content, or NYU RA content.
- Do not publish or deploy the site as part of this change.

## Verification

- Search public page sources to confirm all approved dates and project metrics.
- Confirm old public values no longer appear in `_pages/about.md` or `_pages/projects.md`.
- Confirm Georgia Tech and NYU RA entries are unchanged.
- Run the repository's available site build or validation command and inspect the diff for scope compliance.
