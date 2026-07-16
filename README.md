# Recruitment AI — Automated Resume Screening Workflow

An n8n automation that watches a recruiter's inbox for incoming resumes, extracts and standardizes the candidate's information regardless of file format, evaluates the resume against a job description using an LLM, and logs a structured screening report to Google Sheets — with zero manual triage.

🎥 **Demo video:** [add your Loom/YouTube link here]

---

## Problem it solves

Recruiters manually opening every resume email, reading the CV, and cross-checking it against job requirements is slow and inconsistent. This workflow automates that first-pass screening step: it reads the resume, compares it against the job description, and produces a structured, scored report — ready for a human recruiter to review and make the final call.

## How it works

**1. Intake (Gmail Trigger)**
Polls the recruiter's Gmail inbox every minute for new emails with resume attachments.

**2. Storage (Google Drive)**
Each incoming attachment is uploaded to a dedicated Google Drive folder, keeping a permanent, linkable copy of every resume received.

**3. Format handling (Switch node)**
Resumes arrive as `.docx`, `.pdf`, or `.txt`. A Switch node branches the flow by MIME type:
- **Word docs** are converted to PDF via the Google Drive API (copy + convert), then downloaded and text-extracted.
- **PDFs** are downloaded and text-extracted directly.
- **Plain text files** are downloaded and read as-is.

All three branches converge into a single **Standardize** step, so downstream nodes always work with one consistent `text` field regardless of the original format.

**4. Job description lookup**
The workflow pulls the target job description from a Google Drive file and extracts its text for comparison.

**5. AI evaluation (AI Agent + GPT-4o-mini)**
The candidate's resume text and the job description are passed to an LLM (GPT-4o-mini) with a system prompt that frames it as an expert technical recruiter. It returns a structured evaluation — enforced via a JSON schema (Structured Output Parser) — containing:
- Candidate strengths
- Candidate weaknesses
- Risk factor (Low / Medium / High + worst-case explanation)
- Reward factor (Low / Medium / High + best-case explanation)
- Overall fit rating (0–10)
- Justification for the rating

**6. Contact extraction (Information Extractor)**
A second LLM step pulls the candidate's first name, last name, and email directly from the resume text, so the report can be properly attributed without manual data entry.

**7. Output (Google Sheets)**
A new row is appended to a shared "Resume Screener" spreadsheet containing: timestamp, link to the stored resume, candidate name and email, strengths, weaknesses, risk/reward assessment, overall fit score, and the LLM's justification — giving recruiters a ready-to-scan shortlist.

## Tech stack

| Component | Tool |
|---|---|
| Trigger | Gmail (OAuth2) |
| File storage | Google Drive |
| Document parsing | n8n Extract from File (PDF/text), Google Drive doc conversion |
| LLM | OpenAI GPT-4o-mini |
| Structured output | LangChain Structured Output Parser (JSON schema) |
| Data extraction | LangChain Information Extractor |
| Output/reporting | Google Sheets |
| Orchestration | n8n |

## What this project demonstrates

- Multi-format document ingestion and normalization within a single pipeline
- Prompt design for structured, evaluative LLM output (not just free text)
- Enforcing reliable JSON output from an LLM using schema-based parsing
- Chaining multiple AI steps (evaluation + entity extraction) in one workflow
- End-to-end automation from raw email to a decision-ready spreadsheet, removing manual data entry entirely

## Current limitations

- The job description is currently a fixed file (single role); making it dynamic per posting would be the natural next step.
- No de-duplication logic yet if the same candidate applies twice.
- Runs on personal n8n/Drive/Sheets accounts — not yet deployed on a persistent hosted instance.

## Repository contents

- `Recruitment_AI.json` — exported n8n workflow (importable into any n8n instance)
- `README.md` — this file
- `screenshots/` — workflow canvas views (optional but recommended)
- Demo video link above

---

*Built by Shaheer as a self-directed project exploring practical LLM integration into real-world automation workflows.*
