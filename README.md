# YoungNug: architecture case study

A written case study of **[youngnug.com](https://youngnug.com)**, a UK job-application platform whose development I lead and which I co-founded.

The product source is private. This repository explains how the system is put together and why, at the level a technical interviewer or a hiring manager would want. No code, credentials, infrastructure detail or user data appears here, by design.

---

## The problem

A UK student applying for graduate schemes, placements and apprenticeships repeats the same four steps by hand, hundreds of times: find the vacancy, decide whether it is worth applying to, rewrite the CV and cover letter for it, and keep track of what went where. The second step is the one people get wrong most expensively, and it is the one nothing on the market actually helps with.

## What the system does

1. **Collect.** Vacancies come in from licensed job APIs, applicant tracking system boards and RSS feeds. Each source has its own adapter that normalises into one shared schema, then the set is de-duplicated.
2. **Score.** Each vacancy is scored against the candidate's profile, and the reasons for and against are shown alongside the number.
3. **Draft.** A tailored, ATS-safe CV and cover letter are generated for one specific advert.
4. **Track.** Applications move through a pipeline the candidate can see.

A companion browser extension captures adverts and fills forms in the user's own browser. It is built to stop before submitting, and it does not bypass a login or a captcha.

## Shape of the system

| Layer | What it is | Why it is separate |
|---|---|---|
| Collectors | One adapter per source, each normalising into a shared vacancy schema | A source changing its format, or being dropped entirely, touches one adapter and nothing else |
| Engine | Ranking, scoring and document generation, as plain Python | Testable with no web server and no browser, which is where most of the test suite lives |
| API | FastAPI over SQLAlchemy, with versioned migrations | The engine has no idea it is being served over HTTP |
| Frontend | React and TypeScript | Talks to the API and nothing else |
| Companion | Chrome extension, Manifest V3 | Runs in the user's own browser and holds no server credentials |

The rule underneath the table: the part that makes decisions is the part with no dependencies. Scoring and document generation are pure functions over data, so they can be tested exhaustively without standing anything up.

## Decisions worth defending

**Licensed sources, not scraping.** The obvious way to fill a job board is to scrape one. We use licensed APIs and public applicant tracking system endpoints instead. It is slower to build and narrower in coverage, and it is the only version of this product that survives contact with a legal review.

**The extension never submits.** It fills a form and stops. A user who is one click from an application they have not read is a worse outcome than a user who applied to three fewer jobs, and an automation that submits on someone's behalf is a liability we chose not to carry.

**Reasons, not just a score.** A bare fit score is unfalsifiable, so nobody trusts it and nobody learns from it. Showing what is for and against a role turns the output into something a candidate can argue with, which is what makes it useful.

**Isolation in the query layer, not the interface.** Every query is scoped to the current user at the point the data is fetched. Hiding a row in the interface is not access control; if the isolation is not enforced where the data leaves the database, it is not enforced.

**Local models over metered APIs in the generation path.** Generating a CV cannot cost more than the plan it sits inside. Keeping generation off metered per-call APIs is what makes the unit economics hold at any volume rather than only at low ones.

## Data protection and security

- Argon2 password hashing.
- Short-lived JWT access token with an httpOnly refresh cookie.
- Per-user data isolation enforced on every query.
- Sensitive fields encrypted at rest.
- GDPR subject access export and account deletion, both self-service.
- Registered with the Information Commissioner's Office.
- The minimum user age is 15, so the system is designed against the Children's Code rather than only the general regime.

## Testing

The engine, the collectors and the document generation are covered by a headless suite that runs with no network and no display. Collectors are tested against captured fixtures so a provider outage cannot turn into a red build. Frontend and extension suites run separately. Secrets scanning runs across the repository.

## Where it stands

Built and live. Billing is proven end to end in test mode; the live key has not been switched on. Distribution has not started yet. That order is deliberate, not an accident: the product had to be finished and lawful before it was worth putting in front of anyone.

## My role

Lead developer and co-founder. Day to day the work is system design for new features and building the security layer, alongside general implementation.

---

*This repository is documentation only. YoungNug's source code is proprietary and is not published here.*
