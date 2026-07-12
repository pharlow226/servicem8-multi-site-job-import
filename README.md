# n8n Workflows

A collection of [n8n](https://n8n.io) automation workflows. Each workflow is stored as an exported `.json` file that can be imported directly into any n8n instance (cloud or self-hosted).

## 📋 Table of Contents

- [Workflows](#workflows)
  - [ServiceM8 – Import Multi-Site Job Cards](#servicem8--import-multi-site-job-cards)
- [Requirements](#requirements)
- [Setup & Import Instructions](#setup--import-instructions)
- [Credentials](#credentials)
- [Repository Structure](#repository-structure)
- [Security Notes](#security-notes)
- [Contributing](#contributing)
- [License](#license)

---

## Workflows

### ServiceM8 – Import Multi-Site Job Cards

**File:** `ServiceM8_Import_Multi_Site_Job_Cards.json`

Bulk-creates job cards in [ServiceM8](https://www.servicem8.com/) for a customer with multiple sites, from a single uploaded spreadsheet — instead of creating each job manually in the ServiceM8 UI.

#### What it does

1. **Form trigger** – A user submits a form with the customer name and an uploaded spreadsheet (`.xlsx`, `.xls`, or `.csv`) containing site addresses, contacts, job numbers, and emails.
2. **Client lookup** – Searches ServiceM8 for a client matching the given customer name. Halts with a clear error if no match, or more than one match, is found.
3. **Parse spreadsheet** – Extracts rows from the uploaded file.
4. **Normalize & validate** – Trims/cleans each row, splits contact names into first/last, and flags rows with missing job numbers, missing/invalid emails, or missing site addresses.
5. **Deduplicate** – Removes duplicate job numbers within the uploaded file itself.
6. **Check for existing jobs** – Queries ServiceM8 for jobs already existing under that client with the same PO/job number, and skips them instead of creating duplicates.
7. **Create job card** – For every valid, non-duplicate row: creates a ServiceM8 job (status `Work Order`), sets the PO number and job address, and attaches the site contact.
8. **Error handling** – Distinguishes between auth errors (halts the whole run) and per-row failures (logged and continues).
9. **Summary email** – Sends an HTML summary email (via Gmail) listing how many jobs were created, skipped, or failed, with a details table for anything that needs attention.

#### Expected spreadsheet columns

| Column             | Required | Notes                                  |
|---------------------|:--------:|-----------------------------------------|
| `Job Number`         | ✅ | Used as the ServiceM8 PO/job number and for deduplication |
| `Site Address`       | ✅ | Becomes the job address |
| `Email`              | ✅ | Must be a valid email format |
| `Site Contact Name`  | ➖ | Split into first/last name for the job contact |

#### Integrations used

- **ServiceM8** – official ServiceM8 node (client lookup, job creation) + raw HTTP calls (job lookups, contact creation)
- **Gmail** – sends the results summary email

---

## Requirements

- An [n8n](https://n8n.io) instance — n8n Cloud, self-hosted, or [desktop](https://n8n.io/download)
- A [ServiceM8](https://www.servicem8.com/) account and API access
- The community node [`@servicem8/n8n-nodes-servicem8`](https://www.npmjs.com/package/@servicem8/n8n-nodes-servicem8) installed on your n8n instance
- A Gmail account for sending result emails (or swap the Gmail node for any other email node)

## Setup & Import Instructions

1. **Install the ServiceM8 community node** (if self-hosting): in n8n go to **Settings → Community Nodes** and install `@servicem8/n8n-nodes-servicem8`.
2. **Import the workflow**: in n8n, go to **Workflows → Import from File**, and select the desired `.json` file from this repo.
3. **Reconnect credentials**: n8n does not export credential secrets. After importing, open each node that references a credential (see table below) and select/create the matching credential in your own n8n instance.
4. **Set the notification email**: open the **Config** node and update `notifyEmail`, or better, replace it with an environment variable — see [Security Notes](#security-notes).
5. **Activate** the workflow once credentials are wired up and tested.

## Credentials

These workflows reference credentials by name — you'll need to create your own with matching names/types after import (n8n does **not** export secrets):

| Credential Name (in workflow) | Type | Used For |
|---|---|---|
| `ServiceM8 Credentials account` | ServiceM8 API | Client lookup, job creation |
| `Header Auth account` | HTTP Header Auth | Raw ServiceM8 REST calls (job/contact lookups) |
| `Gmail OAuth2 API` | Gmail OAuth2 | Sending the results summary email |

> ⚠️ The credential *IDs* baked into the exported JSON are specific to the original n8n instance and are meaningless outside it — n8n will prompt you to remap them on import.

## Repository Structure

```
.
├── README.md
├── .gitignore
└── ServiceM8_Import_Multi_Site_Job_Cards.json
```

As more workflows are added, consider grouping them by system/integration, e.g.:

```
.
├── servicem8/
│   └── ServiceM8_Import_Multi_Site_Job_Cards.json
├── slack/
│   └── ...
└── README.md
```

## Security Notes

- **Never commit real credentials, API keys, or tokens.** n8n exports don't include secrets by default, but always double-check before pushing.
- **Watch for hardcoded personal data.** Exported workflows can contain hardcoded values left in `Set`/`Config` nodes (e.g. a personal notification email) — sanitize these into environment variables (`{{ $env.MY_VAR }}`) or n8n [Variables](https://docs.n8n.io/code/variables/) before committing to a public repo.
- **Sanitize `pinData`.** If you pin test data while building a workflow, it gets saved in the export and may contain real customer data. Clear pinned data before exporting/committing.
- **Review webhook IDs** before making a repo public — while not secret by themselves, treat production webhook URLs as sensitive if the workflow is internet-facing.

## Contributing

1. Export your workflow from n8n as JSON (**Workflow → Download**).
2. Strip any pinned test data and personal/hardcoded values.
3. Add it to this repo with a clear filename and a short entry in this README under [Workflows](#workflows).
4. Open a PR.

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
