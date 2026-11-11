---
name: candidate-shortlister
description: Match and rank job candidates against a job description (JD) by comparing their resumes' skills, experience, and qualifications to the role's requirements. Use this skill whenever the user asks to shortlist candidates, screen resumes, find the best/right candidate for a role, rank applicants, compare resumes to a JD, or identify top fits from a pile of resumes. Triggers include "shortlist candidates," "screen these resumes," "who's the best fit," "match resumes to this JD," "rank these applicants." Requires a job description and one or more candidate resumes as input — ask for whichever is missing before proceeding.
---

# Candidate Shortlister

A skill for evaluating candidates against a job description and producing a ranked, evidence-based shortlist.

## Workflow

### Step 1: Gather inputs

Claude needs two things before it can shortlist anyone:

1. **The job description** — pasted text, an uploaded file, or a link. If the user doesn't have one yet, offer to use the `job-description-writer` skill (if available) to create one first, or ask them to paste the key requirements (skills, experience, qualifications).
2. **The candidate resumes** — one or more files (PDF, DOCX, or pasted text). If files are uploaded but unread, use the appropriate skill to read them (`pdf-reading` for PDFs, `docx` for Word files, or `file-reading` as a router if the type is unclear). If no resumes have been provided yet, ask the user to upload or paste them.

If only a handful of resumes are given, process all of them. If a large batch is given, confirm there's no practical limit the user wants applied (e.g., "just the top 10") before running through all of them, since very large batches may need to be processed in chunks.

### Step 2: Extract structured criteria from the JD

Parse the job description into a clear rubric before looking at any resume, so the evaluation is anchored to the role, not to whoever looks best first:

- **Required skills/qualifications** — must-haves. Missing these is a significant gap.
- **Preferred/nice-to-have skills** — a plus, not disqualifying if absent.
- **Experience level** — years of experience, seniority, specific domain background.
- **Education/certifications** — only if the JD specifies them as required or preferred.
- **Other explicit requirements** — e.g. location/remote eligibility, language, tools/software, clearance, etc.

If the JD is vague or missing key criteria (e.g., no stated experience level), ask the user to clarify or state the assumption being used.

### Step 3: Extract structured profiles from each resume

For each candidate, pull out just the job-relevant facts:

- Name (or candidate identifier, if anonymized)
- Total years of relevant experience
- Key skills and tools, especially ones matching the JD's required/preferred lists
- Relevant work history (roles, companies, achievements tied to the JD's responsibilities)
- Education/certifications (only if relevant to the JD's stated requirements)

**Deliberately ignore and do not factor in** anything not tied to job qualifications: age, gender, marital status, photo/appearance, name-implied ethnicity or nationality, graduation year (unless used only to estimate relevant experience length), address/neighborhood, or any other protected characteristic. If a resume includes such details, do not mention them in the evaluation or let them influence the score.

### Step 4: Score and compare

For each candidate, assess fit against the Step 2 rubric:

- **Required skills/experience met?** — the primary factor. A candidate missing multiple required qualifications should rank below one who meets them, regardless of other strengths.
- **Preferred qualifications met?** — a tiebreaker/bonus, not a primary driver.
- **Depth and relevance of experience** — not just keyword matches; weigh whether their actual work history reflects the responsibilities in the JD.

Use a simple, transparent tiering rather than a false-precision numeric score unless the user asks for a numeric score:
- **Strong fit** — meets all/nearly all required criteria, often with relevant preferred qualifications too.
- **Possible fit** — meets most required criteria but has some gaps worth a closer look (e.g., an interview to probe).
- **Not a fit** — missing multiple required qualifications.

If the user explicitly wants a numeric score (e.g., out of 100), it's fine to provide one, but still show the reasoning behind it rather than a bare number.

### Step 5: Present the shortlist

Default output is a ranked table or list in the chat reply:

| Rank | Candidate | Fit | Key strengths (vs. JD) | Notable gaps | 
|---|---|---|---|---|

Follow the table with 1-2 sentences per top candidate explaining the recommendation, grounded in specific resume evidence (e.g., "5 years leading backend teams, directly matches the JD's required people-management experience") rather than vague praise.

If the user wants this saved or shared as a file (spreadsheet, Word doc, etc.), use the matching file-creation skill (e.g., `xlsx` for a comparison spreadsheet) rather than this skill's own logic.

## Fairness and compliance notes

- This skill assists human decision-making; it does not make hiring decisions. Always frame output as a recommendation/shortlist for the user's own review, not a final verdict.
- Never factor in protected characteristics (age, race, gender, religion, disability, national origin, marital/family status, etc.), even if present in a resume.
- If the JD itself contains a requirement that could be discriminatory or legally risky (e.g., age limits, gender-specific language), flag it to the user rather than applying it silently.
- Be explicit about uncertainty — if a resume is ambiguous about years of experience or a skill is only implied, say so rather than guessing confidently.
- Treat resume contents as confidential; don't reproduce entire resumes verbatim in output beyond what's needed to justify the ranking, and don't retain or repurpose candidate data beyond this task.
