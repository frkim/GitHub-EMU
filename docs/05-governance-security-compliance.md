# Governance, security, and compliance baseline for the new EMU enterprise

This guide provides a practical baseline for governing a new GitHub Enterprise Cloud enterprise with Enterprise Managed Users (EMU), with emphasis on policies to set before migration and the control gaps to close immediately after migration.

> Last verified: 2026-10-15 against GitHub Docs

## Table of contents

- [Where this guide fits](#where-this-guide-fits)
- [Governance operating model](#governance-operating-model)
- [Pre-migration policy priorities](#pre-migration-policy-priorities)
- [Baseline policy matrix](#baseline-policy-matrix)
- [Rulesets and branch protection](#rulesets-and-branch-protection)
- [GitHub Advanced Security baseline](#github-advanced-security-baseline)
- [Audit log and SIEM streaming](#audit-log-and-siem-streaming)
- [Network controls, SSO redirect, and SSH CA](#network-controls-sso-redirect-and-ssh-ca)
- [Third-party apps and automation governance](#third-party-apps-and-automation-governance)
- [Software supply chain baseline](#software-supply-chain-baseline)
- [InnerSource governance with internal repositories](#innersource-governance-with-internal-repositories)
- [Licensing and billing](#licensing-and-billing)
- [Compliance and data residency considerations](#compliance-and-data-residency-considerations)
- [GHE.com feature differences that affect governance](#ghecom-feature-differences-that-affect-governance)
- [Migration gaps to track explicitly](#migration-gaps-to-track-explicitly)
- [Old organization retention and legal hold](#old-organization-retention-and-legal-hold)
- [30-60-90 day maturity roadmap](#30-60-90-day-maturity-roadmap)
- [Related reading](#related-reading)

## Where this guide fits

Use this guide with:

- [../README.md](../README.md)
- [01-why-migrate-to-emu.md](01-why-migrate-to-emu.md)
- [02-emu-fundamentals-and-decision-guide.md](02-emu-fundamentals-and-decision-guide.md)
- [03-migration-strategy-and-planning.md](03-migration-strategy-and-planning.md)
- [04-identity-and-access.md](04-identity-and-access.md)
- [06-technical-migration-guide.md](06-technical-migration-guide.md)
- [07-post-migration-and-change-management.md](07-post-migration-and-change-management.md)
- [08-pitfalls-and-mitigations.md](08-pitfalls-and-mitigations.md)
- [09-references-and-research-notes.md](09-references-and-research-notes.md)
- [../checklists/migration-checklist.md](../checklists/migration-checklist.md)

## Governance operating model

### Recommended operating structure

| Layer                         | Primary role                                          | Responsibilities                                                            |
| ----------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------- |
| Enterprise platform authority | Enterprise owners                                     | Enterprise settings, global policies, billing, audit, delegated role design |
| Security authority            | Security managers and/or designated enterprise owners | GHAS posture, audit review, policy exceptions, incident support             |
| Organization operators        | Organization owners                                   | Team/repo administration inside enterprise guardrails                       |
| Cost authority                | Billing managers                                      | Licensing, usage tracking, cost center alignment                            |
| Application governance        | App managers and organization owners                  | GitHub App registration and installation governance                         |
| Repository stewards           | CODEOWNERS / maintainers                              | Branch discipline, review gates, repo-level compliance                      |

### Policy-as-code mindset

Use a small, internal governance repository to track:

- enterprise standards,
- migration exceptions,
- approved patterns for Actions, Apps, and repository creation,
- CODEOWNERS for governance-owned paths,
- policy change records.
  This is generic governance practice rather than a GitHub product requirement, but it aligns well with GitHub's repository-centric control model.

### Governance design principles

1. **Centralize enterprise-wide controls** at the enterprise where possible.
2. **Delegate only what teams need** to organization owners.
3. **Prefer group-managed access** over individual grants.
4. **Prefer GitHub Apps over user PATs** for automation.
5. **Treat migration gaps as backlog items**, not future assumptions.

## Pre-migration policy priorities

Configure or consciously decide these controls **before** migrating production data:

1. Repository creation and visibility
2. User namespace repository creation for EMU users
3. Forking policy
4. Repository deletion and transfer policy
5. Outside collaborator / repository collaborator policy
6. GitHub Actions allowlist, runner model, workflow permissions, and retention
7. GitHub Pages posture
8. Copilot enterprise policy
9. PAT policy, including temporary exceptions for migrations
10. Rulesets bypass for repository migrations
11. GHAS availability for destination organizations
12. SSO redirect behavior for unauthenticated users
13. IP allow list or Entra CAP approach
14. SSH certificate authority strategy
15. GitHub App / OAuth app approval posture
16. Audit log streaming destination

> **Warning**
> GitHub Enterprise Importer requires **personal access tokens (classic)**. If you restrict classic PATs too early, migrations can fail.

## Baseline policy matrix

| Policy                                                 | Recommended baseline                                       | Why                                                                                        | Where to configure                                     |
| ------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| Organization repository visibility                     | **Private and internal only**                              | Managed users cannot create public content or collaborate outside the enterprise           | Enterprise/org repository settings                     |
| Default repo visibility for new org repos              | **Internal by default** unless a repo must be private      | Internal repositories support InnerSource across the enterprise                            | Enterprise member privileges / org settings            |
| User namespace repositories for EMU users              | **Block by default**                                       | Reduces shadow repositories and governance blind spots                                     | Enterprise member privileges                           |
| Repository creation in orgs                            | **Owners only** or tightly limited member creation         | Minimizes sprawl during migration                                                          | Enterprise member privileges or repository policy      |
| Private/internal repo forking                          | **Allow only where needed**; block broad forking initially | Forking affects data spread and access boundaries                                          | Enterprise member privileges / org setting             |
| Repository deletion                                    | **Restrict** to tightly scoped admins                      | Lowers accidental destructive change risk                                                  | Repository policy / org permissions                    |
| Repository transfer                                    | **Restrict** to tightly scoped admins                      | Prevents data movement outside intended org structure                                      | Repository policy / org permissions                    |
| Outside collaborator / repository collaborator invites | **Owners only**                                            | Keeps external access controlled and reviewable                                            | Enterprise member privileges / org programmatic access |
| GitHub Actions availability                            | **Enabled only for approved orgs**                         | Prevents uncontrolled CI activation in staging orgs                                        | Enterprise Actions policy                              |
| Public actions / reusable workflows                    | **Allow enterprise + selected non-enterprise actions**     | Best balance of supply-chain control and usability                                         | Enterprise Actions policy                              |
| Actions pinning                                        | **Require full-length commit SHA for actions**             | Reduces mutable reference risk                                                             | Enterprise Actions policy                              |
| Repository-level self-hosted runners                   | **Disable**                                                | Reduces risk from long-lived or compromised runners                                        | Enterprise Actions policy                              |
| Standard hosted runners                                | **Allow unless compliance requires runner groups only**    | Easier migration; stricter posture can come later                                          | Enterprise Actions policy                              |
| Workflow permissions                                   | **Least privilege; review defaults**                       | Limits write scope of `GITHUB_TOKEN`                                                       | Enterprise/org/repo Actions settings                   |
| Artifact/log retention                                 | **Define enterprise standard**                             | Aligns cost, forensics, and retention expectations                                         | Enterprise Actions policy                              |
| GitHub Pages                                           | **Allow only for approved use cases**                      | Managed users have Pages limitations; public publishing may be inconsistent with EMU goals | Enterprise/org policy                                  |
| GitHub Copilot                                         | **Central enterprise policy**                              | Keeps model/privacy controls consistent                                                    | Enterprise AI controls                                 |
| Classic PAT access                                     | **Allow during migration for approved migrators only**     | GEI requires PAT classic                                                                   | Enterprise PAT policy                                  |
| Fine-grained PATs                                      | **Allow with approval and lifetime controls**              | Better control than classic PATs                                                           | Enterprise PAT policy                                  |
| PAT max lifetime                                       | **Set after migration stabilization**                      | Strong control, but can break provisioning/admin automation if rushed                      | Enterprise PAT policy                                  |
| SSO redirect for unauthenticated users                 | **Enable intentionally**                                   | Default is 404; redirect improves UX for managed users                                     | Enterprise security settings                           |
| IP allow list vs Entra CAP                             | **Choose one primary network control plane**               | Avoid conflicting controls and hard-to-debug outages                                       | Enterprise security settings / Entra                   |
| SSH certificate authority                              | **Adopt if you need managed SSH trust**                    | Centralizes SSH Git access control                                                         | Enterprise security settings                           |
| GitHub App installations                               | **Owner approval model**                                   | Limits third-party risk and shadow integrations                                            | Org app / programmatic-access settings                 |
| OAuth app restrictions                                 | **Restrict and require approval where used**               | OAuth apps are user-context integrations                                                   | Org programmatic-access settings                       |
| Audit log streaming                                    | **Enable before cutover**                                  | Preserves operational evidence from day one                                                | Enterprise audit log settings                          |
| GHAS availability                                      | **Enable for destination orgs before migrating repos**     | Lets feature settings migrate where supported                                              | Enterprise/org security settings                       |
| Secret scanning / code scanning / Dependabot           | **Plan explicit post-migration tasks**                     | GEI does not migrate all security state                                                    | Org/repo security settings                             |
| Custom properties                                      | **Recreate after migration**                               | GitHub documents that repository properties and custom properties are not migrated         | Enterprise/org custom properties                       |

## Rulesets and branch protection

### Rulesets can block migrations

GitHub documents that if the destination organization or enterprise has rulesets enabled, migrated history may violate those rules.
**Recommended baseline:** add **Repository migrations** to the bypass list for every applicable ruleset before production migration.

### Repository migration bypass

GitHub's documented bypass flow is:

1. Open each enterprise or organization ruleset.
2. In **Bypass list**, add **Repository migrations**.
3. Remove or narrow the bypass after migration if it is no longer needed.

### Rulesets are not migrated

GitHub Enterprise Importer does **not** migrate:

- rulesets,
- tag protection rules,
- some repository access settings tied to rulesets.

### Branch protection caveats

GitHub documents that branch protections do migrate, but the following rules do **not** migrate:

- Allow specific actors to bypass required pull requests
- Require approval of the most recent push
- Require deployments to succeed before merging
- Lock branch
- Restrict pushes that create matching branches
- Allow force pushes
  Also not migrated:
- exemptions for people, teams, or apps attached to branch protection rules,
- `Allow force pushes` when configured in **Specify who can force push** mode.

### Governance implication

Treat rulesets and branch protection as **day-0 remediation** after migration validation, not as inherited controls.

> **Warning**
> Organization migrations also do not update team references such as `@org/team` in `CODEOWNERS`. If names change, `CODEOWNERS` can silently stop working until corrected.

## GitHub Advanced Security baseline

### Enterprise baseline

GitHub documents that enterprise owners can enforce availability policies for:

- GitHub Secret Protection,
- GitHub Code Security,
- GitHub Advanced Security,
- dependency insights,
- Dependabot alerts management.

### Pre-migration rule

If you want GitHub Advanced Security settings to carry over where supported, GitHub documents that you should **enable GHAS products for the destination organization before migrating repositories**.

### Post-migration facts to plan for

| Feature                          | What GitHub documents                                                     | Required follow-up                                             |
| -------------------------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------- |
| Secret scanning                  | Destination repo is rescanned; alerts populate without remediation states | Reapply remediation state by API if needed                     |
| Code scanning                    | Alerts are not migrated                                                   | Re-upload SARIF or rerun code scanning                         |
| Dependabot alerts                | Alerts are rebuilt from the current default branch state                  | Re-enable features and re-add any encrypted Dependabot secrets |
| Workflow-based security features | Require Actions to be available                                           | Confirm enterprise/org Actions policy supports them            |

### Secret scanning

GitHub documents:

- a full repository scan is performed after secret scanning is enabled on the destination,
- remediation states are not migrated,
- you can update alerts through the REST API,
- the actor/date on rebuilt remediation reflect the API caller and API time, not the original source history.

### Code scanning

GitHub documents:

- code scanning alerts are not migrated,
- SARIF from the source can be uploaded to the destination,
- rebuilt alerts lose original timeline fidelity,
- rebuilt alerts are represented only as `open` or `fixed`.

### Dependabot

GitHub documents:

- alerts and the dependency graph are rebuilt from the current default branch,
- previous alert history and remediation state are not migrated,
- encrypted secrets for private registries must be re-added.

### Recommended security baseline

1. Enable GHAS products in destination orgs **before** production migration.
2. Re-run or rehydrate code scanning after migration.
3. Enable secret scanning and push protection as soon as destination repos are stable.
4. Rebuild Dependabot with registry secrets and org-wide settings.
5. Use security configurations and org global settings to push consistency at scale.

> **Note**
> GitHub documents that GHAS is billed on a **per-committer** basis. Include that in rollout planning.

## Audit log and SIEM streaming

### Why configure it early

GitHub documents that audit log streaming helps with:

- data exploration,
- data continuity,
- data retention.
  A stream covers audit and Git events for all organizations in the enterprise from the time the stream is enabled onward.

### Operational characteristics from GitHub Docs

| Topic               | GitHub-documented behavior                             |
| ------------------- | ------------------------------------------------------ |
| Delivery model      | At-least-once; duplicates can occur                    |
| Pause buffer        | Seven-day retention window while paused                |
| Health checks       | Every 24 hours                                         |
| Owner notifications | Enterprise owners get email on stream misconfiguration |
| File format         | Compressed JSON `.json.log.gz`                         |

### Streaming targets GitHub documents

- Amazon S3
- Azure Blob Storage
- Azure Event Hubs
- Datadog
- Google Cloud Storage
- Splunk
- Microsoft Purview (Copilot agent session events only)

### GHE.com limitation

GitHub documents that **streaming audit logs to Amazon S3 with OIDC is currently unavailable on GHE.com**.

### Baseline recommendation

| Environment           | Recommended baseline                                                                               |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| GitHub.com enterprise | Stream to your existing SIEM or storage target; S3 with OIDC is an option if it fits your platform |
| GHE.com enterprise    | Use a supported non-S3-OIDC option, or S3 with access keys instead of OIDC                         |

### Evidence retention guidance

- Turn on streaming before user cutover.
- Dedupe events in the SIEM because delivery is at-least-once.
- Preserve source-side logs separately because GEI does **not** migrate audit logs.

## Network controls, SSO redirect, and SSH CA

### SSO redirect for unauthenticated users

GitHub documents that the default experience for an unauthenticated user trying to access the enterprise is a **404**. You can optionally enable automatic redirect to SSO.
**Recommended baseline:** enable redirect unless you intentionally want obscurity over usability.

### IP allow list vs Conditional Access Policy

GitHub documents:

- enterprises using OIDC can choose Entra CAP IP conditions instead of GitHub IP allow lists,
- GitHub enforces CAP IP conditions, not device compliance,
- PAT and SSH auth tied to a user are evaluated against CAP,
- deploy keys are not subject to CAP.

### Baseline decision rule

| Pattern                     | Recommendation                                                                                                              |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Entra OIDC enterprise       | Prefer a single documented network control model; test CAP impacts on Actions, Codespaces, github.dev, Apps, and migrations |
| Non-Entra / SAML enterprise | Use GitHub IP allow lists if you need network restriction                                                                   |
| Mixed controls              | Avoid unless you have a strong operational reason and test coverage                                                         |

### SSH certificate authority

GitHub documents that you can configure an SSH CA at the enterprise level and require members to use SSH certificates for org resources. In EMU enterprises, members can also be allowed to use the certificate for personally owned repositories.
**Recommended baseline:** adopt SSH CA if you already manage SSH certificates centrally; otherwise treat it as a maturity-phase control.

## Third-party apps and automation governance

### Core governance principles

- Prefer **GitHub Apps** over OAuth apps where possible.
- Require owner approval for high-risk integrations.
- Keep a catalog of approved apps, owners, scopes, and business purpose.
- Reinstall apps explicitly after migration because GitHub Enterprise Importer does not migrate GitHub Apps or app installations.

### What GitHub Enterprise Importer does not migrate

GitHub documents that GEI does **not** migrate:

- GitHub Apps,
- GitHub App installations,
- webhook secrets,
- user PATs,
- users' SSH keys or signing keys.

### GitHub Apps vs OAuth apps

The GitHub docs search results point to GitHub documentation that in general prefers GitHub Apps over OAuth apps because of finer-grained permissions. Use that as the default architectural preference.

### Governance baseline

| Integration type       | Baseline                                                            |
| ---------------------- | ------------------------------------------------------------------- |
| GitHub Apps            | Preferred for automation and platform integrations                  |
| OAuth apps             | Allow only when a GitHub App is not viable; require explicit review |
| User PAT-based tooling | Minimize and phase down after migration                             |
| Repo-local secrets     | Recreate only after confirming need and owner                       |

> **Note**
> Exact enterprise-wide versus organization-level app restriction options should be **verified with GitHub documentation/Support** for your deployment, especially if you want one universal enforcement point.

## Software supply chain baseline

### Actions sourcing and pinning

GitHub documents that enterprise policies can:

- allow all actions and reusable workflows,
- allow only enterprise actions,
- allow enterprise actions plus selected non-enterprise actions,
- require actions to be pinned to a full-length commit SHA.

### Recommended baseline

1. Allow **enterprise actions and selected non-enterprise actions**.
2. Allow **GitHub-authored actions** only where needed.
3. Require **full-length commit SHA pinning**.
4. Keep a short allowlist for approved external action namespaces.
5. Disable repository-level self-hosted runners.

### Dependabot and private dependencies

GitHub documents that Dependabot needs explicit access to private/internal dependencies and that registry credentials may need to be configured in `dependabot.yml` or org settings.

### Artifact attestations

Artifact attestation policy details were **not verified from the supplied fact set**.

> **Note**
> If you want artifact attestations in the baseline, **verify with GitHub documentation/Support** before making it a hard requirement.

## InnerSource governance with internal repositories

GitHub documents:

- managed users cannot create public content,
- internal repositories are available for InnerSource,
- regular users added to one organization can access all internal repositories in the enterprise.

### Baseline InnerSource stance

| Topic                                                                       | Recommendation                                            |
| --------------------------------------------------------------------------- | --------------------------------------------------------- |
| Shared code by default                                                      | Use **internal** repositories                             |
| Sensitive code                                                              | Use **private** repositories                              |
| Discovery                                                                   | Standardize repo topics, templates, and governance labels |
| Ownership                                                                   | Require `CODEOWNERS` in internal engineering repos        |
| This combines documented platform behavior with common enterprise practice. |

## Licensing and billing

### What GitHub documents

- GitHub Enterprise Cloud with data residency uses the **GitHub Enterprise plan**.
- The latest billing platform supports **cost centers**.
- You can pay directly through GitHub or connect an **Azure subscription**.
- Usage-based billing is available for GitHub Enterprise and GHAS products.
- Managed users need **Copilot Business** or **Copilot Enterprise** access; they cannot self-subscribe to Copilot Individual/Pro.

### Azure billing facts

GitHub documents:

- connecting Azure installs the **Subscription Permission Validation (SPV)** app,
- tenant-wide admin consent is required unless your tenant consent settings allow otherwise,
- usage-based costs billed through Azure are charged on the **1st of each month**,
- prepaid usage is not available for Azure usage-based billing,
- under a **Microsoft Enterprise Agreement**, connecting Azure is the only way to use GHAS, Codespaces, Copilot, or Actions/LFS/Packages beyond included amounts.

### Billing baseline

| Area                | Baseline recommendation                                       |
| ------------------- | ------------------------------------------------------------- |
| Enterprise licenses | Track ownership in one cost center or agreed allocation model |
| GHAS                | Forecast per-committer impact before broad enablement         |
| Copilot             | Separate seat governance from EMU identity onboarding         |
| Azure payment       | Use when it simplifies enterprise procurement or EA alignment |
| Migration tooling   | Keep temporary PAT exceptions time-boxed and recorded         |

## Compliance and data residency considerations

### Data residency facts from GitHub Docs

GitHub documents these current GHE.com data residency regions:

- EU (including Azure regions in EFTA countries, currently Norway and Switzerland)
- Australia
- US
- Japan

### Compliance framing

Use GitHub controls as inputs to your compliance mapping, not as automatic guarantees.

| Compliance topic                  | Practical guidance                                                                                                                                                        |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SOC 2 / ISO 27001                 | Map GitHub platform controls, audit evidence, and operating procedures to your internal control library; verify current attestations through GitHub documentation/Support |
| GDPR / regional data expectations | Use documented data residency regions where needed; confirm legal interpretation and contractual controls separately                                                      |
| Evidence retention                | Turn on audit streaming and preserve migration logs                                                                                                                       |
| Segregation of duties             | Use enterprise roles, billing managers, app managers, and custom roles                                                                                                    |

> **Warning**
> Do not describe data residency as a blanket compliance guarantee. Use it as one control in a broader legal and compliance design.

## GHE.com feature differences that affect governance

### GitHub-documented differences and gaps

| Feature                                             | Governance implication                                                                                                      |
| --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| GitHub Marketplace unavailable on GHE.com           | You cannot rely on Marketplace as the install/approval surface; apps/actions may need source-based installation and testing |
| Public repositories unavailable with EMU / GHE.com  | Open-source collaboration must stay outside the managed enterprise                                                          |
| S3 audit streaming with OIDC unavailable on GHE.com | Choose another audit streaming auth pattern                                                                                 |
| Certain preview features may lag GitHub.com         | Validate roadmap dependencies before promising parity                                                                       |
| GitHub Importer button unavailable on GHE.com       | Use GitHub Enterprise Importer, not the basic importer                                                                      |

### Codespaces and Copilot restrictions

GitHub documents for managed users:

- users can create codespaces only for enterprise-owned repositories and forks,
- users cannot create codespaces for personal repos or public templates,
- managed users need Copilot Business or Enterprise, not personal Copilot plans,
- Copilot cloud agent is not available in personal repositories owned by managed users.

### API and URL differences on GHE.com

GitHub documents that API, package, raw-content, cloning, SCIM, and Actions OIDC URLs change on GHE.com. Review all integrations for hostname dependencies.

## Migration gaps to track explicitly

GitHub Enterprise Importer does **not** migrate several items that matter to governance.

### High-priority gaps

| Gap                                         | Governance response                        |
| ------------------------------------------- | ------------------------------------------ |
| Rulesets                                    | Recreate manually; plan migration bypasses |
| Custom properties / repository properties   | Recreate taxonomy and backfill values      |
| GitHub Apps and installations               | Reinstall and reapprove                    |
| Webhook secrets                             | Re-enter secrets before re-enabling hooks  |
| Team membership for organization migrations | Rebuild via IdP group sync                 |
| Teams for repository migrations             | Recreate teams and grants                  |
| Audit logs                                  | Preserve source evidence separately        |
| Code scanning results                       | Re-run or re-import SARIF                  |
| Dependabot alerts / secrets                 | Re-enable and reconfigure                  |
| User PATs / SSH keys / signing keys         | Users must re-establish them               |

### Less obvious gaps

| Gap                                                    | Why it matters                      |
| ------------------------------------------------------ | ----------------------------------- |
| Team mentions / CODEOWNERS references after org rename | Can silently break review routing   |
| Fork relationships                                     | Affects development workflows       |
| Workflow artifacts and run history                     | Evidence / operational history loss |
| Repository watchers and stars                          | Social metadata loss                |

## Old organization retention and legal hold

GitHub Enterprise Importer does **not** migrate audit logs. That alone is enough to justify a deliberate retention decision for source organizations.

### Recommended baseline for source orgs

1. Keep source organizations **read-only or tightly controlled** until migration validation completes.
2. Preserve migration logs, audit exports, and any other required evidence.
3. Confirm legal, compliance, and records-retention obligations before deleting source orgs.
4. Document who approves final source decommissioning.

### Legal hold note

Specific GitHub.com legal hold and long-term retention patterns for old organizations were **not fully verified from the supplied fact set**.

> **Note**
> Verify exact legal hold, eDiscovery, and retention requirements with your legal team and **GitHub documentation/Support** before deleting legacy organizations.

## 30-60-90 day maturity roadmap

### First 30 days

- Stand up enterprise owners, billing managers, app managers, and security managers
- Set repository visibility, creation, forking, PAT, and Actions baseline policies
- Enable audit log streaming
- Enable GHAS products for in-scope orgs
- Recreate rulesets, branch protections, custom properties, apps, secrets, and webhooks
- Reclaim mannequins and restore team membership

### Days 31-60

- Move manual access to IdP-driven groups wherever possible
- Tighten Actions allowlists and SHA pinning enforcement
- Review and reduce classic PAT usage
- Reassess guest collaborator and repository collaborator grants
- Validate cost-center tagging and usage reporting
- Publish governance repo with CODEOWNERS and exception process

### Days 61-90

- Introduce SSH CA if required
- Move remaining automation from user PATs to GitHub Apps or OIDC-to-cloud
- Expand security configurations and global org settings
- Audit org-level deviations from enterprise baseline
- Decide whether stricter runner-group-only or CAP-first controls are warranted
- Review source-organization retirement with legal/compliance sign-off

## Related reading

- [03-migration-strategy-and-planning.md](03-migration-strategy-and-planning.md)
- [04-identity-and-access.md](04-identity-and-access.md)
- [06-technical-migration-guide.md](06-technical-migration-guide.md)
- [07-post-migration-and-change-management.md](07-post-migration-and-change-management.md)
- [08-pitfalls-and-mitigations.md](08-pitfalls-and-mitigations.md)
- [09-references-and-research-notes.md](09-references-and-research-notes.md)
- [../checklists/migration-checklist.md](../checklists/migration-checklist.md)
