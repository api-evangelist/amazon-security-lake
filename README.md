# Amazon Security Lake

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Amazon Security Lake is a service that automatically centralizes an organization's security data from cloud, on-premises, and custom sources into a purpose-built data lake stored in your own Amazon S3. It manages the data lifecycle to help you optimize storage and supports OCSF (Open Cybersecurity Schema Framework) for normalized security data analysis.

**Areas:** AWS · Data Lake · Security · SIEM · Threat Detection

## Amazon Security Lake on APIs.io

Everything profiled for Amazon Security Lake is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/amazon-security-lake/](https://apis.io/providers/amazon-security-lake/)** — or go straight to one of the APIs or properties below.

### APIs (5)

- [Amazon Security Lake Data Lakes API](https://apis.io/apis/amazon-security-lake/amazon-security-lake-data-lakes-api/) — Data lake creation and management
- [Amazon Security Lake Log Sources API](https://apis.io/apis/amazon-security-lake/amazon-security-lake-log-sources-api/) — AWS and custom log source management
- [Amazon Security Lake Subscribers API](https://apis.io/apis/amazon-security-lake/amazon-security-lake-subscribers-api/) — Subscriber management for data access
- [Amazon Security Lake Security Lake Notification API](https://apis.io/apis/amazon-security-lake/amazon-security-lake-notification-api/) — The Security Lake Notification API from Amazon Security Lake — 1 operation(s) for security lake notification.
- [Amazon Security Lake Security Lake Tags API](https://apis.io/apis/amazon-security-lake/amazon-security-lake-tags-api/) — The Security Lake Tags API from Amazon Security Lake — 1 operation(s) for security lake tags.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### Arazzo workflows (7)

Multi-step workflows that chain operations into a business task.

- [Amazon Security Lake Decommission Data Lake](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-decommission-data-lake-workflow/)
- [Amazon Security Lake Offboard Subscriber](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-offboard-subscriber-workflow/)
- [Amazon Security Lake Onboard AWS Log Source](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-onboard-aws-log-source-workflow/)
- [Amazon Security Lake Provision Data Lake](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-provision-data-lake-workflow/)
- [Amazon Security Lake Provision Subscriber](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-provision-subscriber-workflow/)
- [Amazon Security Lake Register Custom Source](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-register-custom-source-workflow/)
- [Amazon Security Lake Rename Subscriber](https://apis.io/arazzos/amazon-security-lake/amazon-security-lake-rename-subscriber-workflow/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Amazon Security Lake Agentic Access](https://apis.io/agentic-access/amazon-security-lake/amazon-security-lake-agentic-access/)

#### JSON Schema (3)

The data models the API reads and writes.

- [DataLake](https://apis.io/schemas/amazon-security-lake/amazon-security-lake-data-lake/)
- [LogSource](https://apis.io/schemas/amazon-security-lake/amazon-security-lake-log-source/)
- [Subscriber](https://apis.io/schemas/amazon-security-lake/amazon-security-lake-subscriber/)

#### JSON Structure (3)

Data models expressed in JSON Structure.

- [Amazon Security Lake Data Lake Structure](https://apis.io/json-structure/amazon-security-lake/amazon-security-lake-data-lake-structure/)
- [Amazon Security Lake Log Source Structure](https://apis.io/json-structure/amazon-security-lake/amazon-security-lake-log-source-structure/)
- [Amazon Security Lake Subscriber Structure](https://apis.io/json-structure/amazon-security-lake/amazon-security-lake-subscriber-structure/)

#### JSON-LD (1)

Linked-data contexts for the provider's vocabulary.

- [Amazon Security Lake Context](https://apis.io/jsonld/amazon-security-lake/amazon-security-lake-context/)

#### Vocabulary (1)

The provider's domain terms, defined.

- [Amazon Security Lake Vocabulary](https://apis.io/vocabularies/amazon-security-lake/amazon-security-lake-vocabulary/)

#### Examples (3)

Example requests and responses.

- [Amazon Security Lake Data Lake Example](https://apis.io/examples/amazon-security-lake/amazon-security-lake-data-lake-example/)
- [Amazon Security Lake Log Source Example](https://apis.io/examples/amazon-security-lake/amazon-security-lake-log-source-example/)
- [Amazon Security Lake Subscriber Example](https://apis.io/examples/amazon-security-lake/amazon-security-lake-subscriber-example/)

#### Collections (5)

Postman and Open Collection request collections.

- [Amazon Security Lake API](https://apis.io/collections/amazon-security-lake/open-amazon-security-lake/)
- [Amazon Security Lake Data Lakes API](https://apis.io/collections/amazon-security-lake/open-amazon-security-lake-data-lakes-api/)
- [Amazon Security Lake Data Lakes Log Sources API](https://apis.io/collections/amazon-security-lake/open-amazon-security-lake-log-sources-api/)
- [Amazon Security Lake Data Lakes Subscribers API](https://apis.io/collections/amazon-security-lake/open-amazon-security-lake-subscribers-api/)
- [Amazon Security Lake API](https://apis.io/collections/amazon-security-lake/postman-amazon-security-lake/)

#### Governance rules (2)

Spectral rulesets for the provider's contracts.

- [Amazon Security Lake — jsonschema rules](https://apis.io/rules/amazon-security-lake/amazon-security-lake-jsonschema-spectral-rules/)
- [Amazon Security Lake API Rules](https://apis.io/rules/amazon-security-lake/amazon-security-lake-spectral-rules/)

#### Security (4)

Authentication, trust centre, and vulnerability disclosure.

- [Amazon Security Lake Authentication](https://apis.io/security/amazon-security-lake/amazon-security-lake-authentication/)
- [Amazon Security Lake Domain Security](https://apis.io/security/amazon-security-lake/amazon-security-lake-domain-security/)
- [Amazon Security Lake Trust Center](https://apis.io/security/amazon-security-lake/amazon-security-lake-trust-center/)
- [Amazon Security Lake Vulnerability Disclosure](https://apis.io/security/amazon-security-lake/amazon-security-lake-vulnerability-disclosure/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Amazon Security Lake Plans Pricing](https://apis.io/plans/amazon-security-lake/amazon-security-lake-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Amazon Security Lake Rate Limits](https://apis.io/rate-limits/amazon-security-lake/amazon-security-lake-rate-limits/)

#### FinOps (1)

What the API costs to run, by operation.

- [Amazon Security Lake Finops](https://apis.io/finops/amazon-security-lake/amazon-security-lake-finops/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://aws.amazon.com/security-lake/)
- [Documentation](https://docs.aws.amazon.com/security-lake/)
- [Getting started](https://aws.amazon.com/security-lake/getting-started/)
- [API reference](https://docs.aws.amazon.com/security-lake/latest/APIReference/)
- [Pricing](https://aws.amazon.com/security-lake/pricing/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/security/tag/amazon-security-lake/)
- [Compliance](https://aws.amazon.com/compliance/)
- [Terms of service](https://aws.amazon.com/service-terms/)
- [Privacy policy](https://aws.amazon.com/privacy/)
- [GitHub](https://github.com/aws)
- [Status](https://health.aws.amazon.com/health/status)

## API Evangelist rating

**Kin Score 55.0** — strong · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 69.6 |
| Contract quality | 32.3 |
| Contract governance | 27.3 |
| Operational transparency | 36.8 |
| Developer ergonomics | 65.5 |
| Access clarity | 68.4 |

**Agent readiness 24.6** — agent-aware

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | — |
| MCP Server | — |
| Auth Clarity | bearer (partial credit) |
| Idempotency | — |
| Error Semantics | — |
| OpenAPI Examples | verified |
| Rate Limit Signal | derived (partial credit) |
| Event Surface Described | — |
| Agent Skills | — |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | — |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | na |
| Operation Distinctiveness | distinct |

**Regulatory alignment 35.3** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/amazon-security-lake/](https://apis.io/providers/amazon-security-lake/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Amazon Security Lake: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/amazon-security-lake/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
