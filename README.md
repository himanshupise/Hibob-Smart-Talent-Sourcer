# Smart Talent Sourcer (HiBob) - Workato Package

## 1. Project Overview

**Purpose.** This Workato package delivers Smart Talent Sourcer, an AI-assisted recruitment solution built around HiBob as the applicant tracking system (ATS). It syncs job postings and candidate applications, scores candidates against job requirements using OpenAI, and gives recruiters a conversational Genie in Slack to manage the pipeline.

**Business problem solved.** Recruiters otherwise review applications and compare candidates to job descriptions by hand, and chase candidate follow-ups manually. This package automates that work:

* Job descriptions and applications flow from HiBob into Workato automatically.
* Each candidate receives an AI match score with reasoning, visible to the recruiter in Slack.
* Recruiters browse jobs, check scores and status, approve or reject candidates, generate reports and trigger outreach by chatting with a Genie, without opening Workato.
* Outreach, candidate-reply monitoring and follow-up emails run automatically.

**Systems integrated.**

| System | Role |
| --- | --- |
| HiBob | ATS and system of record for jobs (Career module) and applications (Hiring module) |
| OpenAI | Candidate match scoring and reply classification |
| Gmail (Google) | Outreach, rejection and follow-up emails |
| Slack | Conversational chat interface for recruiters |
| Workato | Orchestration: recipes, skills, Genies, data tables |

## 2. Features and Workflows

### 2.1 Core sync recipes (run automatically once activated)

| Recipe | What it does |
| --- | --- |
| [HSTS] REC 001 - Job Description Sync | Syncs job requisitions, including the job owner (recruiter), from HiBob into the job description data table. Trigger: every 10 minutes. |
| [HSTS] REC 002 - Applied Candidate Sync | Syncs candidate applications into `candidates_profile`, tied to the correct job. Trigger: every 5 minutes. |
| [HSTS] REC 003 - Matchscore Sync (with Matchscore-Automatically skill) | Calculates match score and reasoning for new candidates and syncs them back. |

### 2.2 Candidate engagement recipes

| Recipe | What it does |
| --- | --- |
| REC 001 - Monitor Candidate Response with OpenAI | Monitors candidate email replies and classifies them. |
| REC 002 - Follow Up Candidate Email Trigger | Sends follow-ups to candidates who have not replied within the defined window. |

### 2.3 Callable recipes (skills)

| Group | Skills |
| --- | --- |
| Job skills | Fetch JDs, Fetch Single JD Details, Fetch JDs by department |
| Recruiter skills | Fetch score of candidate, Fetch status of candidate, Fetch applied candidates by job role, Fetch applied candidates by role and tier, Matchscore explicitly (on-demand scoring), Report generation, Approve or Reject Candidate (calls another Genie) |
| Engagement skills | Send out-reach email, Send Reject email |

### 2.4 Genies

| Genie | Skills |
| --- | --- |
| Recruitment Support Genie (main, used in Slack) | 10 |
| Match Scoring Genie | 1 |
| Hibob Candidate Engagement Genie | 2 |

### 2.5 Data tables (lookup and working storage)

`job_description`, `candidates_profile`, `candidate_matchscore`, `openai_audit_log`, `sync_run`, `Hibob_candidate_outreach_and_follow_up_mail_details`

### 2.6 Connections

HiBob Account, Google account (Gmail) and Drive, OpenAI account, Slack connection

### 2.7 Other assets

Knowledge bases: Match Scoring Rules, Project KB.

## 3. Package Contents

| Item | Value |
| --- | --- |
| Package name | [Package name, e.g. Smart Talent Sourcer] |
| Package version | [1.0.0] |
| Root project folder | `Hibob Connector` |
| Managed with | Workato Recipe Lifecycle Management (RLCM) |
| Maintained by | [Team Name] |

### 3.1 Included assets

| Asset type | Count |
| --- | --- |
| Background sync recipes | 3 |
| Recruiter skill recipes | 11 |
| Candidate engagement recipes | 4 |
| Genies | 3 |
| Data tables | 6 |
| Connections | 4 |

### 3.2 Folder layout and purpose

| Folder | Purpose |
| --- | --- |
| `Hibob Connector` | Root folder for all assets |
| `Recipes` | Core background sync recipes and job skills |
| `Skill Recipes / Recruiter` | Recruiter-facing skills used through the Slack Genie |
| `Candidate Engagement and Conversion` | Outreach, follow-up and reply-monitoring recipes plus the engagement Genie |
| `Datatable` | Six working data tables |
| `Connection` | HiBob, Gmail, OpenAI and Slack connection shells |
| `_Config`, `_Ops` (optional) | Environment property documentation, scoring threshold lookups, health-check and cleanup utilities |

## 4. Installation and Deployment

### 4.1 Download the package from GitHub

1. Open the repository: [GitHub repository URL].
2. Go to the Releases page (or the designated release branch/tag) and choose the approved package version.
3. Download the package zip, or clone the repository and locate the zip:
   ```
   git clone [GitHub repository URL]
   git checkout [release tag or branch]
   ```
4. Do not unzip the package. RLCM imports the zip as is.
5. Note the package version for your deployment record.

### 4.2 Import into Workato

1. Sign in to the target Workato workspace as an Administrator.
2. Create a folder named `Hibob Connector` to hold the imported assets.
3. Go to Tools > Recipe Lifecycle Management > Import Package.
4. Select `Hibob Connector` as the destination and upload the zip.
5. Confirm the import finishes without errors and that the asset counts in 3.1 match.

### 4.3 Configure connections and properties

Recipes are imported inactive and unconfigured.

1. Set the environment properties (see section 5.3).
2. Re-authorize all four connections (HiBob, Google, OpenAI, Slack) with the target workspace's own credentials. Credentials never travel with a package.
3. Double-check every property and connection before activating anything. A missed value here is the most common cause of failure in later steps.

### 4.4 Deploy (activate in order)

1. **Core sync recipes:** activate Job Description Sync, then Applied Candidate Sync, then Matchscore Sync. They run on their triggers, so no manual run is needed. After each one, check Job history, confirm the expected data table rows, and verify in HiBob (jobs in the Career module, applications in the Hiring module) before moving on.
2. **Candidate engagement recipes:** activate Monitor Candidate Response and Follow Up Candidate Email Trigger only after the sync recipes run cleanly.
3. **Genies:** confirm all skills are registered and active (10 on Recruitment Support Genie, 1 on Match Scoring Genie, 2 on Hibob Candidate Engagement Genie).
4. **Slack:**
   * Authenticate the Slack connection through OAuth using a Slack Admin account.
   * Under Genie > End user access, create an end-user group and add the intended users.
   * In the Genie's Triggers section, add Slack as a "Chat with this genie in" interface and confirm it shows Active.
   * Confirm the correct Genie and its skills are linked to the Slack trigger.
   * Make sure only intended groups have access, not the "Everyone" group unless intended.
5. Leave schedules active only after all recipes have had a clean run, and record the deployment outcome, version and date.

### 4.5 Upgrading an existing deployment

1. Announce a maintenance window if needed and stop all active recipe schedules.
2. Export a backup package of the current project (strongly recommended).
3. Import the new package version through RLCM and map to the existing connections. Do not create new ones.
4. Confirm environment properties and the recruiter details table are intact.
5. Smoke test: run Job Description Sync and confirm a job appears in the Career module, then run Applied Candidate Sync and confirm a test application appears in the Hiring module.
6. Re-enable schedules and record the version and date.

What is replaced and what is kept on upgrade:

| Asset | On new release |
| --- | --- |
| Sync, recruiter and engagement recipes | Replaced with new logic (re-check sync schedules after import) |
| Connection shells | Mapping kept; credentials stay in the workspace |
| Working data tables (`candidates_profile`, `candidate_matchscore`, job description, `openai_audit_log`) | Structure kept; working rows may be cleared |
| Recruiter details table, outreach and follow-up mail details table, `sync_run` | Retained (do not overwrite) |
| Environment properties | Retained (do not overwrite) |

## 5. Configuration and Prerequisites

### 5.1 Required applications and accounts

* Workato workspace with Recipe Lifecycle Management and Genie capability
* HiBob account with API access and the Career and Hiring modules in use
* OpenAI account with an API key
* Google account (Gmail) for outreach emails
* Slack workspace

### 5.2 Permissions and credentials

| Item | Needed |
| --- | --- |
| Workato | Administrator rights in the target workspace for import, connections and Genie setup |
| HiBob | API credentials with access to jobs and candidate applications |
| OpenAI | API key for the target account |
| Gmail | Account authorized to send outreach emails |
| Slack | Slack Admin account able to approve the OAuth authorization and with permission to post and read in the intended channels |

### 5.3 Environment properties

| Property | Value |
| --- | --- |
| `companyName` | The organization name |

Any other customer-specific value added later should also go in environment properties rather than inside recipe steps. The guide's upgrade and go-live checklists also refer to values such as the HiBob tenant ID, Slack channel ID, follow-up email cadence, scoring thresholds and an error notification email. Confirm whether your deployment uses them.

### 5.4 Dependencies and data prerequisites

* Every job in HiBob must have an owner (its creator). That person is treated as the job's recruiter, and all outreach emails and candidate response alerts for the job go to them. Without an owner, there is nobody to notify.
* The six data tables must exist before recipes run.
* The Match Scoring Rules and Project KB knowledge bases must be attached to the Genies.
* Run order matters: Job Description Sync, then Applied Candidate Sync, then Matchscore Sync, then the engagement recipes.

### 5.5 Environments (recommended)

| Environment | Use |
| --- | --- |
| Development | Build and unit test |
| QA / Staging | End-to-end testing with non-production HiBob, non-production Gmail and a test Slack channel |
| Production | Released, validated packages only |

Use the same connector types with different credentials in each. Alternatively, use separate workspaces (for example HSTS-Dev, HSTS-QA, HSTS-Prod). Never hand-fix Production recipes outside the package process.

## 6. Security, Testing and Support

### 6.1 Credential handling

* Never commit credentials, API keys, tokens or passwords to this repository. Review exported files and remove sensitive information before committing or sharing.
* Never hard-code secrets in recipe steps. Use Connections for HiBob, OpenAI, Gmail and Slack.
* Put customer-specific values in environment properties.
* Credentials are re-authorized after every import and stay within the customer workspace.
* Use one isolated workspace per customer. Invite users by email to that workspace only, and enable SSO if required.
* Use least-privilege roles:

| Role | Intended for | Not allowed |
| --- | --- | --- |
| Administrator (customer side, limited number) | Customer IT or HR owner | Access to other customer workspaces |
| Operator | Day-to-day recruitment ops | Redesigning core recipe or scoring logic |
| Analyst / Viewer | Auditors, HR leadership | Changing credentials or recipes |
| Recruiter (Genie end user) | Recruiters, hiring managers | Recipe build view, connections, environment properties |

* Customers may change connection credentials, environment property values, follow-up and batch scoring schedules (within agreed windows), recipe start/stop, and Genie skill active/inactive toggles.
* Customers should not change the core scoring logic (Matchscore-Automatically, Matchscore explicitly) or the scoring weight formula structure.
* Document who owns credential rotation for HiBob, OpenAI, Google/Gmail and Slack.
* Follow organizational security and retention policies.

### 6.2 Validation and testing

After import and configuration, verify:

* Package version recorded and zip stored in the release location.
* All recipes, skills, Genies, data tables and connections are present (counts in 3.1).
* All four connections show as connected.
* Expected data table rows are populated after each recipe: a `job_description` row after JD sync, a `candidates_profile` row after candidate sync, a `candidate_matchscore` row after scoring.
* Jobs appear in HiBob's Career module, applications in the Hiring module, and scores on the candidate record in Slack.
* The Slack chat interface works with test recruiter prompts: fetch jobs, candidate status, match score, approve or reject, and report generation. The Genie also appears in the Slack workspace under the group name.
* The Genie answers using Match Scoring Rules and the Project KB, and does not hallucinate on out-of-scope questions.
* Candidate-facing boundary tests pass: candidates cannot see scores, other candidates' data, or approve or reject.
* The Genie routes bulk or automatic scoring and explicit on-demand scoring requests correctly.
* Each Genie has its user group under End user access.

### 6.3 Troubleshooting

| Symptom | What to check |
| --- | --- |
| A recipe fails on its first run | Re-check every environment property and connection authorization. A missed value is the most common cause. |
| Failed step in a sync recipe | Open Job history and inspect the failed step. Confirm the connection is authorized. |
| No recruiter notifications or outreach emails | Confirm the HiBob job has an owner and that Job Description Sync has run since it was set. |
| Slack Genie not responding | Check the Slack connection, that the Slack trigger shows Active, and that the user is in the End user access group. |
| Duplicate processing after an upgrade | Confirm the `sync_run` table was retained and not replaced. |
| Missing recruiter record after an upgrade | Restore the recruiter details table from your backup package. |

### 6.4 Version control

* Keep approved changes in the organization's designated repository.
* Use meaningful commit messages and release identifiers.
* Document package updates and deployment-related changes.
* Follow the organization's review and approval process before promoting changes.

### 6.5 Ownership and support

* Project Owner: Harbinger Group

### 6.6 References

* Workato Documentation: https://docs.workato.com/
* Workato RLCM Export Guide: https://docs.workato.com/en/recipe-development-lifecycle/export.html
* Smart Talent Sourcer (HiBob) Workato Deployment & Recipe Lifecycle Guide, v1.0 (2026)
