# GitHub.com → GitHub Enterprise Managed Users (EMU) Migration Checklist

A single, printable checklist covering the whole migration: decision, organization, identity, governance, technical migration, cutover, post-migration and decommissioning. Copy this file into your project tracker (or paste into an issue) and tick items off as you go.

> Last verified: 2026-10-15 against GitHub Docs. Each section links to the guide that explains the "why" and "how".

**How to use**

- `[ ]` = open, `[x]` = done. Every item has an **owner role** in *italics* and, where relevant, a **gate** (a go/no-go decision).
- Run the checklist **per wave** for sections 8–10 and **once per program** for the others.
- Items marked 🔒 are **irreversible or hard to reverse** – double-check before doing them.
- Items marked 🚦 are **gates**: do not proceed until they are signed off.

**Contents**

1. [Decision & business case](#1-decision--business-case)
2. [Program organization & governance](#2-program-organization--governance)
3. [Discovery & inventory](#3-discovery--inventory)
4. [New EMU enterprise & identity provider](#4-new-emu-enterprise--identity-provider)
5. [Governance, security & compliance baseline](#5-governance-security--compliance-baseline)
6. [Destination design (orgs, teams, naming)](#6-destination-design-orgs-teams-naming)
7. [Migration tooling & access](#7-migration-tooling--access)
8. [Trial run (per wave)](#8-trial-run-per-wave)
9. [Production migration & cutover (per wave)](#9-production-migration--cutover-per-wave)
10. [Post-migration follow-up tasks (per wave)](#10-post-migration-follow-up-tasks-per-wave)
11. [Developer enablement & change management](#11-developer-enablement--change-management)
12. [Hypercare & decommissioning](#12-hypercare--decommissioning)
13. [Sign-off summary](#13-sign-off-summary)

Related guides: [Why EMU](../docs/01-why-migrate-to-emu.md) · [Fundamentals & decision](../docs/02-emu-fundamentals-and-decision-guide.md) · [Strategy & planning](../docs/03-migration-strategy-and-planning.md) · [Identity & access](../docs/04-identity-and-access.md) · [Governance & security](../docs/05-governance-security-compliance.md) · [Technical guide](../docs/06-technical-migration-guide.md) · [Post-migration](../docs/07-post-migration-and-change-management.md) · [Pitfalls](../docs/08-pitfalls-and-mitigations.md)

---

## 1. Decision & business case

Guide: [02 – Fundamentals & decision guide](../docs/02-emu-fundamentals-and-decision-guide.md), [01 – Why migrate](../docs/01-why-migrate-to-emu.md)

- [ ] Business drivers documented (identity control, offboarding, compliance, data residency, leak prevention, governance). *Executive sponsor*
- [ ] Answered the six enterprise-type questions: control of user accounts, data residency need, supported IdP, need for public repos/gists/Pages, need to collaborate outside the enterprise, tolerance for migration cost. *Architect*
- [ ] Confirmed that **no business-critical workflow requires public repositories, gists, public Pages, or contributing to external repositories** from corporate identities (or a dual-account policy has been defined). *Architect + Legal*
- [ ] Decided hosting model: **GitHub.com with EMU** vs **GHE.com (data residency)** – and region (EU incl. Norway/Switzerland, Australia, US, Japan) if applicable. 🔒 *Architect + Compliance*
- [ ] Reviewed the list of features unavailable on GHE.com (e.g. Marketplace, macOS runners, Maven/Gradle in Packages, dependency insights) against actual usage. *Platform lead*
- [ ] Confirmed that adopting EMU requires a **new enterprise account** (an existing personal-accounts enterprise cannot be converted) and that Sales/procurement agree on commercials. 🔒 *Procurement*
- [ ] Business case approved (benefits, costs, risks, timeline). 🚦 *Steering committee*

## 2. Program organization & governance

Guide: [03 – Strategy & planning](../docs/03-migration-strategy-and-planning.md)

- [ ] Executive sponsor and steering committee named. *Sponsor*
- [ ] Workstreams staffed: Identity · Platform/technical migration · Security & compliance · Developer experience & change · Legal/Procurement. *Program manager*
- [ ] RACI agreed. *Program manager*
- [ ] Program plan with phases, waves, freeze windows and exit criteria published. *Program manager*
- [ ] Risk register created and reviewed (start from [pitfalls](../docs/08-pitfalls-and-mitigations.md)). *Program manager*
- [ ] Communication plan and channels (mailing list, chat channel, office hours) defined. *Change lead*
- [ ] Success metrics/KPIs defined (e.g. % users provisioned, % repos migrated, validation pass rate, support tickets, time-to-onboard). *Program manager*
- [ ] Budget for licences (GitHub Enterprise seats, Copilot Business/Enterprise, Actions minutes/storage) confirmed. *FinOps*

## 3. Discovery & inventory

Guide: [03 – Discovery & inventory](../docs/03-migration-strategy-and-planning.md)

- [ ] Inventory of **source organizations** and owners. *Platform lead*
- [ ] Inventory of **repositories**: size, visibility, archived/active, last activity, default branch, LFS usage, forks, number of PRs and issues (e.g. with the third-party `gh-repo-stats` extension – not supported by GitHub Support). *Platform lead*
- [ ] Size analysis with `git-sizer` for large/complex repos (commit size, blob size, ref length, tree counts). *Migration engineer*
- [ ] Inventory of **users and collaborators**: employees, contractors, vendors, **outside collaborators**, bots/machine users. *Identity lead*
- [ ] Inventory of **integrations**: GitHub Apps, OAuth apps, webhooks, deploy keys, PATs, SSH keys, CI/CD systems, bots. *Platform lead*
- [ ] Inventory of **GitHub Actions** assets: secrets, variables, environments, self-hosted/larger runners, reusable workflows, third-party actions. *DevOps lead*
- [ ] Inventory of **data not migrated by GEI** that must be recreated or exported (Packages, Projects (new), repo-level Discussions, rulesets, tag protection rules, custom properties, sub-issues, code scanning alerts, Dependabot/secret scanning state). *Platform lead*
- [ ] Inventory of **public repositories, gists and Pages sites** (not allowed in EMU – decide: make internal/private, archive, move to a separate public org/account, or delete). 🚦 *Legal + Platform lead*
- [ ] Inventory of **forks** and fork relationships (not migrated). *Platform lead*
- [ ] Inventory of **branch protections / rulesets** (some rules not migrated; rulesets not migrated). *Security*
- [ ] Wave plan drafted with prioritization (pilot first, then low-risk, then complex) and owners per repo. *Program manager*
- [ ] Naming conflicts identified (org names, repo names; org/user share a namespace). *Platform lead*

## 4. New EMU enterprise & identity provider

Guide: [04 – Identity & access](../docs/04-identity-and-access.md)

### 4.1 Enterprise creation

- [ ] IdP/partner chosen: **Entra ID** (SAML or OIDC), **Okta** (SAML), **PingFederate** (SAML), or other SAML 2.0 + SCIM 2.0 system. Use **one partner IdP for both authentication and provisioning**. *Identity lead*
- [ ] Confirmed the **Okta + Entra ID combination (SSO/SCIM in either order) is NOT used** – it is unsupported. *Identity lead*
- [ ] **Shortcode chosen** (3–8 alphanumeric characters on GitHub.com; cannot be changed later). 🔒 *Identity lead + Branding*
- [ ] Authentication method decided: Entra ID → **OIDC recommended** (Conditional Access Policy support); other IdPs → SAML. Note: with several enterprises on one Entra tenant, additional enterprises must use SAML. *Identity lead*
- [ ] New EMU enterprise created (GitHub.com, or your ghe.com subdomain/region). 🔒 *Enterprise owner*

### 4.2 Setup user

- [ ] Setup user (`SHORTCODE_admin`) password set in a private/incognito window. *Enterprise owner*
- [ ] 2FA enabled for the setup user; recovery codes saved. *Enterprise owner*
- [ ] Enterprise SSO recovery codes downloaded and stored in the corporate vault. *Enterprise owner*
- [ ] Setup user credentials stored in a password manager with a documented break-glass procedure. *Security*
- [ ] SCIM **PAT (classic)** created by the setup user (scope `scim:enterprise`, **no expiration**) and stored securely; ownership/renewal process documented. *Identity lead*

### 4.3 SSO and SCIM

- [ ] SSO (OIDC or SAML) configured and tested with a pilot user. *Identity lead*
- [ ] SCIM provisioning configured in the IdP application. *Identity lead*
- [ ] IdP app assignment strategy defined (groups, not individuals). *Identity lead*
- [ ] **Username preview** performed: IdP identifiers normalize to unique usernames (no conflicts; ≤ 39 chars incl. `_SHORTCODE` on GitHub.com; ≤ 30 on GHE.com). *Identity lead*
- [ ] **One email per user** confirmed in the IdP and aligned with the email used for Git commits. *Identity lead*
- [ ] Pilot users provisioned and able to sign in; deprovisioning/reprovisioning tested. *Identity lead*
- [ ] IdP groups for teams created (Entra ID: **security groups only**, no nested groups or Microsoft 365 groups). *Identity lead*
- [ ] Conditional Access Policy (if OIDC) reviewed – **service accounts for migration/automation exempted** as needed; impact on Actions with PATs, Codespaces, github.dev reviewed. *Identity lead + Security*
- [ ] Process for contractors/vendors defined (provision as managed users or guest collaborators). *Identity lead*
- [ ] Developer guidance for dual accounts (corporate EMU account + personal account for open source) published. *Change lead*

## 5. Governance, security & compliance baseline

Guide: [05 – Governance, security & compliance](../docs/05-governance-security-compliance.md)

- [ ] Enterprise owners and org owners nominated; admin roles minimized. *Security*
- [ ] Repository policies set: only **private/internal** repos; creation, forking, deletion and transfer rules. *Platform lead*
- [ ] Policy decided for **user-owned repositories** of managed users (allow/deny). *Platform lead*
- [ ] Actions policy set (allowed actions, workflow permissions, runner usage). *DevOps lead*
- [ ] Copilot (Business/Enterprise) policies and seat assignment model defined. *Platform lead*
- [ ] SSO redirect/unauthenticated-user behavior set (404 vs redirect). *Security*
- [ ] IP allow list / CAP decided; **GEI IP ranges temporarily allowed if needed** for source and destination. *Security*
- [ ] PAT policy decided – note **GEI requires PAT (classic)**; the "restrict classic PATs" policy must allow the migrator accounts during migration. *Security*
- [ ] GitHub Advanced Security features (secret scanning, push protection, code scanning, Dependabot) enabled at destination orgs **before** migration so settings migrate. *Security*
- [ ] Rulesets and branch-protection standards defined for destination; **ruleset bypass for migration** prepared (org rulesets can block migrations). *Security*
- [ ] Audit log retention and streaming/SIEM integration configured. *Security*
- [ ] Compliance requirements mapped (data residency, retention, e-discovery). *Compliance*
- [ ] Billing set up (payment method or Azure subscription, cost centers, budgets). *FinOps*
- [ ] Governance operating model documented (who approves new orgs, policy exceptions, etc.). *Platform lead*

## 6. Destination design (orgs, teams, naming)

Guide: [03 – Target org structure](../docs/03-migration-strategy-and-planning.md), [04 – IdP groups → teams](../docs/04-identity-and-access.md)

- [ ] Target organization structure decided (1:1 with source orgs, or consolidated/re-organized). *Architect*
- [ ] Naming conventions for orgs, repos, teams and enterprise teams approved. *Architect*
- [ ] Destination organizations created (production) **and sandbox orgs** for trial runs (e.g. suffix `-sandbox`). *Enterprise owner*
- [ ] Teams created and **connected to IdP groups** (remove manually added members first; teams linked to groups cannot be parents/children of other teams). *Identity lead*
- [ ] Default repository visibility strategy defined (internal for InnerSource vs private). *Architect*
- [ ] Runner groups / self-hosted & larger runners planned for destination. *DevOps lead*
- [ ] Name reuse plan: if reusing org/enterprise names, **rename** source accounts instead of deleting. *Platform lead*

## 7. Migration tooling & access

Guide: [06 – Technical migration guide](../docs/06-technical-migration-guide.md)

- [ ] Migration engineers identified; **migrator role** granted to a team (in **both** source and destination orgs for repository migrations). *Org owners*
- [ ] For organization migrations: runner is an **enterprise owner of the destination** and an org owner/migrator in the source. *Enterprise owner*
- [ ] **PATs (classic)** created for source (`GH_SOURCE_PAT`) and destination (`GH_PAT`) with the required scopes; authorized for SSO where applicable. *Migration engineer*
- [ ] GitHub CLI ≥ 2.4.0 installed; `gh extension install github/gh-gei` and `gh extension upgrade github/gh-gei` run (extension is updated weekly). *Migration engineer*
- [ ] For GHE.com: `--target-api-url https://api.SUBDOMAIN.ghe.com` (or env var) verified. *Migration engineer*
- [ ] Access from the migration host to source and destination verified (network, proxy, allow lists, CAP exemption). *Migration engineer*
- [ ] Runbook, rollback plan and escalation path (GitHub Support contacts) documented. *Migration lead*
- [ ] Secrets handling: PATs stored in a vault, never committed; tokens rotated/revoked after the migration. *Security*

## 8. Trial run (per wave)

Guide: [06 – Trial run](../docs/06-technical-migration-guide.md)

- [ ] Wave scope frozen and communicated to repo owners. *Migration lead*
- [ ] Repository hygiene checks passed: no commit > 2 GiB, refs ≤ 255 bytes, no file > 400 MiB (and plan for > 100 MiB post-migration limit), repository/metadata < 40 GiB, releases ≤ 10 GiB (else `--skip-releases`). *Migration engineer*
- [ ] Open items documented for manual recreation: webhooks, apps, secrets, variables, environments, rulesets, tag protections, Packages, Projects, Discussions, custom properties. *Repo owners*
- [ ] Trial migration executed into the **sandbox** org (`gh gei generate-script` / `migrate-repo` / `migrate-org`). *Migration engineer*
- [ ] Migration status verified (`SUCCEEDED`) and **migration logs reviewed**; warnings triaged (accept/fix/redo). *Migration engineer*
- [ ] Validation performed by repo owners (commits, branches, tags, PR/issue counts, wikis, releases, workflows, branch protections, Pages settings). *Repo owners*
- [ ] Follow-up tasks rehearsed (Actions secrets, webhooks, apps, LFS push, teams, mannequin reclaim). *Migration engineer*
- [ ] Duration measured and used to size the production window. *Migration lead*
- [ ] Issues found are resolved and documented; trial run repeated if needed. *Migration lead*
- [ ] 🚦 Go/No-Go for production wave. *Steering committee*

## 9. Production migration & cutover (per wave)

Guide: [06 – Production run](../docs/06-technical-migration-guide.md), [07 – Cutover runbook](../docs/07-post-migration-and-change-management.md)

**Before (T-minus)**

- [ ] Freeze window announced (**GEI has no delta migration** – changes made during/after export are not migrated). *Change lead*
- [ ] Open PRs reviewed: merge, close or accept re-review in new location. *Repo owners*
- [ ] Destination GHAS settings enabled; destination org rulesets reviewed for migration bypass. *Security*
- [ ] Destination IP allow list/CAP checked; GEI IP ranges allowed if required. *Security*
- [ ] Optionally lock source repositories at the start of the freeze to prevent writes (confirm the mechanism in the GEI docs). *Platform lead*

**During**

- [ ] Run production migration(s) (up to 5 concurrent repository migrations; org migrations up to 5,000 repos). *Migration engineer*
- [ ] Monitor progress (`wait-for-migration`, log files). *Migration engineer*
- [ ] Capture migration IDs and log locations in the wave tracker. *Migration engineer*
- [ ] Download/review migration logs for **every** repo; resolve failures or warnings. *Migration engineer*

**After**

- [ ] Smoke tests: clone, push, open PR, CI run, branch protection, access for a pilot user. *Repo owners*
- [ ] Repository visibility set (migrated repos are **private by default**; use `internal` for enterprise-wide visibility – public is unavailable in EMU). *Platform lead*
- [ ] Teams and repository access granted (team **membership** is not migrated; sync via IdP groups). *Identity lead*
- [ ] Source repositories archived/locked with a pointer to the new location (after validation). *Platform lead*
- [ ] 🚦 Wave sign-off by repo owners. *Repo owners*

## 10. Post-migration follow-up tasks (per wave)

Guide: [06 – Post-migration tasks](../docs/06-technical-migration-guide.md), [07 – Follow-up](../docs/07-post-migration-and-change-management.md)

- [ ] **Webhooks** re-enabled (migrated disabled) and secrets re-added. *Repo owners*
- [ ] **GitHub Apps** reinstalled; OAuth apps re-authorized; app IPs added to CAP if needed. *Repo owners + Identity lead*
- [ ] **GitHub Actions**: secrets, variables, environments re-created; self-hosted/larger runners reconfigured; workflows re-run successfully (run history is not migrated). *DevOps lead*
- [ ] **Dependabot secrets** and private registry access re-added; Dependabot alerts rebuilt (verify). *Security*
- [ ] **Secret scanning** full scan reviewed; alert remediation states recreated if needed via REST API. *Security*
- [ ] **Code scanning** re-run and/or SARIF uploaded; expected differences documented. *Security*
- [ ] **Git LFS** objects pushed to the migrated repo (not migrated by GEI). *Repo owners*
- [ ] **Packages, Projects, Discussions, rulesets, tag protection rules, custom properties, sub-issues** recreated or replaced. *Platform lead*
- [ ] **Mannequins** reclaimed: CSV generated (`gh gei generate-mannequin-csv`), mapped, reclaimed (`gh gei reclaim-mannequin`, `--skip-invitation` available for EMU). Done **before** any transfer of repos to another org. *Org owners*
- [ ] **Commit signature verification** restored: users add GPG/SSH signing keys and verify the committer email; web-commit "Unverified" status raised with GitHub Support if needed. *Developers*
- [ ] **CODEOWNERS**, `@team` references and docs updated for new org/team/user names (mentions are not rewritten). *Repo owners*
- [ ] Links in wikis, READMEs, docs, tickets, dashboards, badges and package manifests updated to the new URLs. *Repo owners*
- [ ] CI/CD systems, deployment tools, bots, scripts and IaC repointed to new remotes/URLs/credentials. *DevOps lead*
- [ ] Code search re-indexing delay (a few hours) communicated. *Platform lead*
- [ ] IP allow list entries for GEI removed; IdP IP restrictions re-enabled. *Security*
- [ ] Migration PATs revoked/rotated; migrator role removed when waves are complete. *Security*

## 11. Developer enablement & change management

Guide: [07 – Change management](../docs/07-post-migration-and-change-management.md)

- [ ] Communication sent at T-8w/T-4w/T-1w/T-1d and at cutover (adapt to your cadence). *Change lead*
- [ ] Self-service documentation published: first sign-in, new username (`name_SHORTCODE`), clone URLs, credentials, SSH keys, GPG keys, PATs, Git Credential Manager. *Change lead*
- [ ] Dual-account guidance (EMU vs personal) and risk of leaking internal code to public repos explained. *Change lead + Security*
- [ ] Training delivered per persona (developer, maintainer, org owner, IdP admin, security). *Change lead*
- [ ] Known EMU restrictions communicated (no public repos/gists, no outside forks/stars/issues from the managed account, no personal Copilot plans, etc.). *Change lead*
- [ ] Copilot Business/Enterprise licenses assigned where applicable. *Platform lead*
- [ ] Champions/ambassadors network established. *Change lead*
- [ ] Support model defined: intake channel, SLAs, FAQ, office hours, escalation to GitHub Support. *Support lead*

## 12. Hypercare & decommissioning

Guide: [07 – Hypercare & decommissioning](../docs/07-post-migration-and-change-management.md)

- [ ] Hypercare period (e.g. 2–4 weeks per wave) staffed; daily triage of issues. *Support lead*
- [ ] KPIs reported to steering committee (migrated repos, users active, incidents, satisfaction). *Program manager*
- [ ] Remaining personal-account access to **old** organizations removed. *Platform lead*
- [ ] Old organizations/repos archived or locked; retention and legal-hold requirements met **before** any deletion. *Legal + Platform lead*
- [ ] If names must be reused: source accounts **renamed** (not deleted). *Platform lead*
- [ ] Old tokens/SSH keys/deploy keys/webhooks/apps for old orgs revoked. *Security*
- [ ] Final audit-log export of old organizations stored per retention policy. *Security + Compliance*
- [ ] Lessons learned workshop held; docs updated. *Program manager*
- [ ] Ongoing governance cadence established (quarterly policy review, GitHub changelog tracking, seat/budget review). *Platform lead*

## 13. Sign-off summary

| Gate | Criteria | Approver | Date | Status |
|---|---|---|---|---|
| G0 – Business case | Section 1 complete | Steering committee | | ☐ |
| G1 – Foundation ready | Sections 4–7 complete | Platform + Identity + Security leads | | ☐ |
| G2 – Pilot go/no-go | Section 8 complete for pilot wave | Steering committee | | ☐ |
| G3 – Wave go/no-go (repeat per wave) | Section 8 complete for the wave | Migration lead + repo owners | | ☐ |
| G4 – Wave accepted (repeat per wave) | Sections 9–10 complete for the wave | Repo owners | | ☐ |
| G5 – Program closure | Sections 11–12 complete | Executive sponsor | | ☐ |
