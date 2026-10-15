# References, Research Notes & Latest Announcements

Sources used to build this guidance, how the research was done, what to re-verify before you start, and where to track new announcements. GitHub evolves quickly; treat this file as the "freshness register" for the whole documentation set.

> Last verified: 2026-10-15 against GitHub Docs.

**Contents**

1. [Research method](#1-research-method)
2. [Primary sources (GitHub Docs)](#2-primary-sources-github-docs)
3. [Notable current-state findings](#3-notable-current-state-findings)
4. [Limitations of this research](#4-limitations-of-this-research)
5. [Where to track the latest announcements](#5-where-to-track-the-latest-announcements)
6. [Re-verification list before kickoff](#6-re-verification-list-before-kickoff)
7. [Glossary](#7-glossary)

---

## 1. Research method

The documentation set was produced by splitting the work into parallel, specialized workstreams (strategy & decision, program planning & change management, identity & security, technical migration & pitfalls). Every workstream drew on one shared, verified fact base that was extracted from the **official GitHub Docs** on 2026-10-15. Statements that could not be verified from official documentation are explicitly marked "verify with GitHub documentation/Support" inside the guides.

No statistics, prices, customer claims or dated announcements have been invented. Where the guides give examples (e.g. a wave plan, a RACI) they are templates, not measurements.

## 2. Primary sources (GitHub Docs)

### Enterprise Managed Users and identity

| Topic | URL |
|---|---|
| About Enterprise Managed Users | <https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users> |
| Choosing an enterprise type | <https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/enterprise-fundamentals/choose-an-enterprise-type> |
| Getting started with EMU | <https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/getting-started-with-enterprise-managed-users> |
| Abilities and restrictions of managed user accounts | <https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/abilities-and-restrictions-of-managed-user-accounts> |
| Username considerations for external authentication | <https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/iam-configuration-reference/username-considerations-for-external-authentication> |
| Configuring SCIM provisioning for EMU | <https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-users> |
| Managing team memberships with IdP groups | <https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/managing-team-memberships-with-identity-provider-groups> |
| About support for your IdP's Conditional Access Policy | <https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/about-support-for-your-idps-conditional-access-policy> |

### Data residency (GHE.com)

| Topic | URL |
|---|---|
| About GitHub Enterprise Cloud with data residency | <https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency> |
| Feature overview for GHE.com | <https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/feature-overview-for-github-enterprise-cloud-with-data-residency> |

### GitHub Enterprise Importer (GEI)

| Topic | URL |
|---|---|
| About GitHub Enterprise Importer | <https://docs.github.com/en/migrations/using-github-enterprise-importer/understanding-github-enterprise-importer/about-github-enterprise-importer> |
| About migrations between GitHub products (data migrated / not migrated, limits) | <https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-between-github-products/about-migrations-between-github-products> |
| Overview of a migration between GitHub products (planning, trial runs, follow-up tasks) | <https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-between-github-products/overview-of-a-migration-between-github-products> |
| Managing access for a migration (migrator role, PAT scopes) | <https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-between-github-products/managing-access-for-a-migration-between-github-products> |
| Migrating repositories from GitHub.com to GitHub Enterprise Cloud | <https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-between-github-products/migrating-repositories-from-githubcom-to-github-enterprise-cloud> |
| Migrating organizations from GitHub.com to GitHub Enterprise Cloud | <https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-between-github-products/migrating-organizations-from-githubcom-to-github-enterprise-cloud> |
| Reclaiming mannequins | <https://docs.github.com/en/migrations/using-github-enterprise-importer/completing-your-migration-with-github-enterprise-importer/reclaiming-mannequins-for-github-enterprise-importer> |
| Mannequins and user activity | <https://docs.github.com/en/migrations/overview/mannequins-and-user-activity> |
| GEI CLI source (`gh-gei`) | <https://github.com/github/gh-gei> |

### Third-party tools mentioned

| Tool | Purpose | Support note |
|---|---|---|
| `gh-repo-stats` (GitHub CLI extension, `mona-actions/gh-repo-stats`) | Inventory of repos, PRs, issues, users | Open source, **not supported by GitHub Support** |
| `git-sizer` | Repository size/shape analysis | Open source |

## 3. Notable current-state findings

These are the findings from the 2026-10-15 research that most influence the migration plan.

| # | Finding | Impact on the plan |
|---|---|---|
| 1 | EMU is available for **new enterprise accounts**; moving from a personal-accounts enterprise to EMU requires migrating to a **new** enterprise. | Budget for a full migration; plan a parallel run. See [02](02-emu-fundamentals-and-decision-guide.md). |
| 2 | Data residency (GHE.com) **requires** EMU. Regions: EU (incl. Norway and Switzerland), Australia, US, Japan. | Region choice is part of the decision. See [01](01-why-migrate-to-emu.md), [05](05-governance-security-compliance.md). |
| 3 | Partner IdPs: Entra ID (SAML/OIDC/SCIM), Okta (SAML/SCIM), PingFederate (SAML/SCIM). **Okta + Entra ID mixed SSO/SCIM is not supported.** | IdP design. See [04](04-identity-and-access.md). |
| 4 | Entra ID **OIDC** is recommended and supports Conditional Access Policy (CAP). Additional enterprises on the same tenant must use SAML. | See [04](04-identity-and-access.md). |
| 5 | CAP can block Actions using PATs, Codespaces, github.dev, apps acting for users, and **GEI migrations** unless service accounts are exempted. | See [04](04-identity-and-access.md), [06](06-technical-migration-guide.md). |
| 6 | The shortcode (GitHub.com) cannot be changed after creation; on GHE.com it is random/hidden and usernames are limited to 30 characters (39 on GitHub.com). | See [04](04-identity-and-access.md), [08](08-pitfalls-and-mitigations.md). |
| 7 | Managed users cannot create public repos/gists, cannot fork/star/watch/comment outside the enterprise, and cannot use Copilot Pro/Free. | Developer communication and dual-account policy. See [07](07-post-migration-and-change-management.md). |
| 8 | GEI migrates from GitHub.com by repository or by organization (up to 5,000 repos per org migration; up to 5 concurrent repo migrations). | See [06](06-technical-migration-guide.md). |
| 9 | GEI does **not** migrate: Actions secrets/variables/environments/runners/history, Apps, Packages, new Projects, repo-level Discussions, rulesets, tag protections, custom properties, stars/watchers, forks relationships, LFS objects, code scanning results, secret-scanning remediation states, user access, team membership (org migration migrates teams, not membership). | Manual recreation plan. See [06](06-technical-migration-guide.md). |
| 10 | User activity becomes **mannequins**; commit authorship follows the commit **email**. Reclaiming can skip invitations for EMU with the CLI. | See [06](06-technical-migration-guide.md), [07](07-post-migration-and-change-management.md). |
| 11 | No delta migrations; trial runs are strongly recommended; migration time is driven mostly by the number of PRs and issues. | See [03](03-migration-strategy-and-planning.md). |
| 12 | Limits: 2 GiB per commit, 255-byte refs, 400 MiB/file during migration (100 MiB afterwards), 40 GiB source and metadata (public preview), releases ≤ 10 GiB. | Pre-migration hygiene. See [06](06-technical-migration-guide.md). |
| 13 | Organization rulesets can **cause migrations to fail** (e.g. commit-email rules); bypasses may be needed. | See [05](05-governance-security-compliance.md), [06](06-technical-migration-guide.md). |
| 14 | GHE.com currently lacks, among others: Marketplace, macOS runners, Maven/Gradle in Packages, dependency insights, S3 audit-log streaming with OIDC. | Feature gap analysis. See [02](02-emu-fundamentals-and-decision-guide.md). |
| 15 | For GHES → GHE.com migrations, Enterprise Live Migrations (GHES 3.17+ patch releases) is an alternative to GEI. | Out of scope here (source is GitHub.com) but relevant if you also run GHES. |

## 4. Limitations of this research

- The GitHub **changelog** (`github.blog/changelog`) was **not reachable** from the research environment, so no dated changelog announcements are quoted. Do not rely on this repository for "what shipped last week" – use the sources in the next section.
- An AI-generated web-search summary that listed many dated 2026 "announcements" was **not used**, because the items could not be verified against primary sources.
- Several features are in public preview (e.g. CAP web-session protection, GEI 40 GiB limits) and can change quickly.
- Pricing, licensing terms and SLAs are intentionally not covered; confirm with your GitHub account team.

## 5. Where to track the latest announcements

| Source | What to watch |
|---|---|
| GitHub Changelog – <https://github.blog/changelog/> | Filter for *Enterprise management tools*, *Identity and access management*, *Migrations*, *GitHub Enterprise Cloud* |
| GitHub Docs – "What's new" and release notes | Enterprise Cloud behaviour changes |
| `github/gh-gei` releases – <https://github.com/github/gh-gei/releases> | New GEI flags/fixes (the extension is updated weekly) |
| GitHub Status – <https://www.githubstatus.com/> | Incidents impacting migrations |
| Your IdP's changelog (Microsoft Entra, Okta, Ping) | SCIM/SSO app changes |
| GitHub account team / Sales / Support | Roadmap, region availability, migration assistance, feature gaps on GHE.com |

## 6. Re-verification list before kickoff

Re-check these items on the day you start, because they change most often:

- [ ] EMU abilities and restrictions page (new capabilities/limitations).
- [ ] GHE.com "currently unavailable features" list and available regions.
- [ ] GEI "data that is not migrated" list and size limits.
- [ ] `gh gei --help` output for the flags used in your scripts (flags may be added or renamed).
- [ ] IdP application versions/gallery apps for GitHub EMU (OIDC vs SAML).
- [ ] Copilot plan names, policies and availability for managed users.
- [ ] Any new GitHub changelog entries tagged enterprise/identity/migrations since the date at the top of this file.

## 7. Glossary

| Term | Meaning |
|---|---|
| **EMU** | Enterprise Managed Users – an enterprise type where the IdP creates and controls user accounts. |
| **GHEC** | GitHub Enterprise Cloud. |
| **GHE.com** | The hosting option for GHEC **with data residency**, on your own subdomain (`SUBDOMAIN.ghe.com`). Requires EMU. |
| **GEI** | GitHub Enterprise Importer – migration service/CLI (`gh gei`). |
| **ELM** | Enterprise Live Migrations – GHES → GHE.com alternative to GEI. |
| **IdP** | Identity provider (Entra ID, Okta, PingFederate, …). |
| **SCIM** | System for Cross-domain Identity Management – provisioning protocol. |
| **SAML / OIDC** | Authentication protocols used for SSO. |
| **CAP** | Conditional Access Policy (Entra ID) – enforced by GitHub with OIDC. |
| **Shortcode** | Enterprise suffix appended to managed usernames (`user_SHORTCODE`). |
| **Setup user** | The first enterprise owner (`SHORTCODE_admin`), the only non-SCIM account. |
| **Mannequin** | Placeholder identity that holds migrated user activity until reclaimed. |
| **Migrator role** | Org-level role allowing GEI use without being an org owner. |
| **Internal repository** | Repository visible to all enterprise members (InnerSource). |
| **Wave** | A batch of repositories/orgs migrated together. |
