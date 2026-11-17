# OfferBot: Automated Offer Letter Generation

An end-to-end **UiPath** automation that generates personalised offer letters as PDFs and emails them to hired candidates. It uses the **Dispatcher–Performer** model with **Orchestrator queues and assets**, built on **REFramework**, with retries, logging and exception handling.

**Tools:** UiPath Studio, REFramework, UiPath Orchestrator, Excel, Word and Mail activities

## Architecture

```
Candidate details.xlsx ──► Dispatcher ──► Orchestrator Queue ──► Performer (REFramework) ──► PDF offer letter ──► Email to candidate
```

### 1. Dispatcher (`Dispatcher/`)
- Reads the Excel file path from an **Orchestrator asset**.
- Reads candidate data (ID, Name, Address, Date of joining, Status, Email) from `Candidate details.xlsx`.
- Filters to candidates with status **Hired**.
- Loads them into the Orchestrator queue with **Bulk Add Queue Items**.

### 2. Performer (`Performer_Offer_Letter/`)
Built on the REFramework state machine (Init → Get Transaction Data → Process → End Process):
- Gets each candidate from the queue (`HR_OfferLetter_Hired`).
- Opens the Word offer letter template and replaces the placeholders: candidate name, address, date of joining and employer name.
- Saves the letter as `.docx` and exports it to **PDF**.
- Emails the PDF to the candidate over SMTP, with the subject and body taken from `Config.xlsx`.
- Throws a **Business Rule Exception** when a candidate's email is blank. System exceptions are retried, and a screenshot is saved.

## Configuration
Settings live in `Performer_Offer_Letter/Data/Config.xlsx` and in Orchestrator assets:

| Setting | Purpose |
|---|---|
| `OrchestratorQueueName` | Queue the Dispatcher loads and the Performer reads |
| `EmployerName` | Employer name in the letter |
| `TemplatePath` | Path to the Word offer letter template |
| `OfferLetterFolderPath` | Output folder for generated letters |
| `EmailSubject` / `EmailBody` | Email content |
| `MaxRetryNumber`, `MaxConsecutiveSystemExceptions` | Retry and failure handling |

SMTP credentials are read from Orchestrator assets or Windows Credential Manager. They are never stored in the project.

## How to run
1. Open both projects in UiPath Studio (24.10+).
2. In Orchestrator, create the queue `HR_OfferLetter_Hired`, the Excel path asset and the SMTP credential asset.
3. Update `TemplatePath` and `OfferLetterFolderPath` in `Config.xlsx` to match your machine.
4. Run **Dispatcher** to load the queue, then run **Performer_Offer_Letter** to generate and send the letters.

## Sample output
Sample letters for test candidates are in `Performer_Offer_Letter/FI - Generated Offer Letters PDFs/`.

## Author
**Rekha S.C.** — built during an RPA Developer internship at Technolabz Business Solutions (2025).
