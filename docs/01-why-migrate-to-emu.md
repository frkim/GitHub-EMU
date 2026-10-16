# Why migrate to EMU

This guide explains the executive case for moving from GitHub Enterprise with personal accounts to GitHub Enterprise Managed Users (EMU), so security, technology, and platform leaders can decide whether tighter identity control, stronger enterprise isolation, and cleaner lifecycle automation justify the migration effort.

> Last verified: 2026-10-15 against GitHub Docs

## Table of contents

- [Executive summary](#executive-summary)
- [Who this document is for](#who-this-document-is-for)
- [What problem EMU solves](#what-problem-emu-solves)
  - [1) Shadow identities and fragmented ownership](#1-shadow-identities-and-fragmented-ownership)
  - [2) Offboarding gaps and delayed deprovisioning](#2-offboarding-gaps-and-delayed-deprovisioning)
  - [3) No authoritative control of username and email](#3-no-authoritative-control-of-username-and-email)
  - [4) Public-external collaboration as a leakage path](#4-public-external-collaboration-as-a-leakage-path)
  - [5) Inconsistent control boundaries](#5-inconsistent-control-boundaries)
- [What EMU changes at a glance](#what-emu-changes-at-a-glance)
- [Benefits of EMU](#benefits-of-emu)
  - [Security and identity lifecycle](#security-and-identity-lifecycle)
  - [Compliance and data residency](#compliance-and-data-residency)
  - [Governance and policy enforcement](#governance-and-policy-enforcement)
  - [Leak prevention and isolation from public GitHub](#leak-prevention-and-isolation-from-public-github)
  - [Operational efficiency](#operational-efficiency)
  - [Developer experience](#developer-experience)
  - [Cost and risk](#cost-and-risk)
- [Comparison: personal accounts enterprise vs EMU](#comparison-personal-accounts-enterprise-vs-emu)
- [When EMU is not the right fit](#when-emu-is-not-the-right-fit)
- [Business-case template](#business-case-template)
- [What changes for developers](#what-changes-for-developers)
- [Recommended next reads](#recommended-next-reads)
- [Official documentation](#official-documentation)

## Executive summary

GitHub Enterprise Managed Users moves account lifecycle and sign-in authority from employee-owned personal GitHub accounts to your identity provider (IdP). In EMU, the IdP provisions accounts with SCIM, users authenticate through the IdP with SAML or OIDC, and the IdP controls usernames, profile data, organization membership, and repository access.

For leadership teams, the value proposition is straightforward:

- You reduce reliance on user-managed identities for enterprise source code access.
- You make provisioning and deprovisioning part of the same lifecycle process used for other enterprise applications.
- You create a clearer enterprise boundary because managed accounts cannot create public content or collaborate outside the enterprise.
- You can use internal repositories for InnerSource while still preventing accidental collaboration with the broader public GitHub network.
- If you need GitHub Enterprise Cloud with data residency (`ghe.com`), EMU is required.

The trade-off is equally important:

- EMU is not the best solution for every customer.
- Existing enterprises that use personal accounts cannot be converted in place; adopting EMU requires a migration to a **new** enterprise account.
- Managed users lose common personal-account patterns such as public repositories, gists, and external collaboration from the managed identity.
- Developers who need outside collaboration usually need a separate personal GitHub account.

> **Warning**
> GitHub Docs state that you must choose the enterprise type before creating the enterprise, and an existing enterprise with personal accounts cannot be converted to EMU. Plan EMU as a migration program, not as a settings change.

## Who this document is for

This document is written for:

- **CISOs and security leaders** evaluating identity assurance, offboarding control, and data leakage risk.
- **CTOs and engineering executives** deciding how GitHub should fit into enterprise architecture and developer operating models.
- **Platform engineering leaders** who will own the target-state identity, organization, and automation design.
- **Enterprise architects** comparing GitHub Enterprise with personal accounts versus EMU as control models.

If you already know EMU is the target, continue with [02-EMU fundamentals and decision guide](02-emu-fundamentals-and-decision-guide.md), [03-migration strategy and planning](03-migration-strategy-and-planning.md), and [../checklists/migration-checklist.md](../checklists/migration-checklist.md).

## What problem EMU solves

GitHub Enterprise with personal accounts can be a valid model, especially when you need public collaboration or want to preserve one account for both internal and external work. In that model, users create and own their GitHub accounts, SAML links the enterprise identity to the personal account, and SCIM provisions organization access rather than the account itself.

That model is flexible. It is also different from full enterprise identity ownership.

### 1) Shadow identities and fragmented ownership

With personal accounts, the enterprise does not own the account itself. Even if access to your organizations is centrally managed, the base GitHub identity remains user-controlled.

Typical leadership concerns include:

- The same human identity can exist as both an enterprise user and a public GitHub persona.
- Username and profile behavior are not centrally governed by the enterprise.
- Internal and external activity can happen from the same account context.
- Control evidence is split between enterprise settings and user-managed account state.

EMU addresses this by making the IdP authoritative for the managed account.

### 2) Offboarding gaps and delayed deprovisioning

In EMU, deprovisioning in the IdP suspends the managed account, invalidates sessions, and retains the account information. If the user is reassigned later, the account is reactivated and the username is restored.

That is materially different from depending on a personal account relationship that must be disconnected from organizations. For many enterprises, the core value is not only access removal, but also having access removal run through the same joiner-mover-leaver process already used for other enterprise systems.

Leadership questions to ask:

- How quickly can we suspend access today?
- Is access removal driven from HR/IdP lifecycle events, or from manual ticket workflows?
- How much evidence do we need to prove offboarding control to auditors or regulators?

### 3) No authoritative control of username and email

GitHub Docs state that for managed users:

- Profile name and email come from the IdP.
- Users cannot change them on GitHub.
- The IdP can provide only **one** email address.

This matters when enterprises want tighter identity consistency across audit evidence, source control attribution, and access reviews.

It also creates migration implications:

- If the IdP email changes, the user becomes unlinked from contribution history tied to the old email.
- Commit authorship on GitHub is tied to commit email, not mannequins.
- A managed user's IdP email should match the commit email strategy you want after migration.

> **Warning**
> Email strategy is not a cosmetic detail. If you change a developer's email in the IdP, GitHub Docs say the user becomes unlinked from contribution history associated with the old email.

### 4) Public-external collaboration as a leakage path

Managed accounts cannot create public content or collaborate outside the enterprise. They can view public repositories read-only, but they cannot push, open issues or pull requests, comment, react, fork repositories outside the enterprise, or follow outside users.

For leaders, that restriction is often the point:

- It narrows the paths by which internal code or context can leave the enterprise-managed boundary.
- It reduces accidental use of the wrong account in public contexts.
- It creates a cleaner separation between enterprise work and personal/open-source activity.

The trade-off is that some engineering organizations depend heavily on open-source contribution from the same daily identity. Those organizations must decide whether separate personal accounts are operationally acceptable.

### 5) Inconsistent control boundaries

In EMU:

- Organizations in the enterprise can have only **private** or **internal** repositories.
- Managed accounts are visible only to enterprise members.
- Users cannot be invited outside the enterprise.
- Unauthenticated access to enterprise resources returns `404` by default, unless you enable an optional SSO redirect setting.

Taken together, this creates a tighter boundary around code, people, and collaboration surfaces than the personal-account model.

## What EMU changes at a glance

The shift to EMU is best understood as a control-model change.

| Area | Enterprise with personal accounts | EMU |
| --- | --- | --- |
| Account lifecycle | User owns account; enterprise governs organization access | IdP provisions and deprovisions the managed account itself |
| Authentication | Enterprise can link SAML identity to the personal account | Authentication happens through the IdP using SAML or OIDC |
| Provisioning | SCIM provisions organization access | SCIM provisions managed users |
| Username and profile email | Not enterprise-owned in the same way | Controlled by the IdP |
| Public and outside collaboration | Possible from the personal account | Blocked from the managed account |
| Repository visibility in EMU orgs | Varies by enterprise design | Private and internal only |
| Data residency on `ghe.com` | Not the EMU model | Requires EMU |

## Benefits of EMU

### Security and identity lifecycle

EMU gives security teams a more authoritative identity plane for GitHub access.

Key benefits:

- **Central lifecycle control**: SCIM creates, updates, suspends, and restores managed accounts from the IdP.
- **Deterministic offboarding**: Deprovisioning invalidates sessions and suspends the account.
- **Controlled sign-in**: Authentication is delegated to the IdP, using SAML or OIDC depending on the supported pattern.
- **Consistent profile data**: Username inputs, name, and email are controlled through the IdP configuration.
- **Stronger enterprise isolation**: Managed identities are not general-purpose GitHub identities.

For Entra ID, GitHub recommends OIDC and notes that it supports Conditional Access Policy (CAP). That can strengthen location-based sign-in controls, though GitHub Docs also describe important CAP caveats that must be reviewed during design.

### Compliance and data residency

If your enterprise requires GitHub Enterprise Cloud with data residency, EMU is foundational.

GitHub Docs state that data residency:

- Requires EMU.
- Uses the `SUBDOMAIN.ghe.com` domain model.
- Uses the API endpoint `https://api.SUBDOMAIN.ghe.com`.
- Offers regions in the **EU**, **Australia**, **US**, and **Japan**.
- Has network details that differ from GitHub.com.

For leadership teams, this creates a cleaner story for:

- Regulatory or customer requirements around data location.
- Region-specific enterprise designs.
- Standardizing enterprise access through managed identities.

> **Note**
> Region selection is a major design decision. Treat it as an early architecture choice and verify any future-change options with GitHub documentation or GitHub Support.

### Governance and policy enforcement

EMU shifts more responsibility into systems the enterprise already governs.

Relevant governance benefits include:

- **IdP-owned account authority** instead of user-owned account authority.
- **Group-to-team synchronization** from the IdP, including enterprise teams.
- **Automatic organization addition** when a synced team adds a user.
- **Default privacy boundary** because enterprise resources return `404` when the user is not authenticated.
- **Role assignment from the IdP** for member, owner, and guest collaborator.

There are still governance choices to make:

- Repository access for a team remains managed on GitHub.
- Connected teams have structure constraints.
- Team maintainer cannot be assigned on an IdP-managed team.

Those details do not remove the benefit; they define the operating model you must design for. See [04-identity and access](04-identity-and-access.md) and [05-governance, security, and compliance](05-governance-security-compliance.md).

### Leak prevention and isolation from public GitHub

For many executives, this is the headline benefit.

Managed accounts:

- Cannot create public repositories.
- Cannot create gists.
- Cannot collaborate outside the enterprise.
- Cannot star, watch, fork, or socially interact outside the enterprise.
- Can only contribute to private and internal repositories in the enterprise and to their own private repositories.

That means EMU is well aligned when the enterprise wants:

- A strict separation between internal engineering and public GitHub activity.
- Lower risk of accidental publication from the enterprise-managed identity.
- A collaboration boundary that is easier to explain to legal, compliance, and security stakeholders.

### Operational efficiency

EMU can reduce administrative complexity when designed well.

Examples:

- **SCIM-driven lifecycle automation** reduces manual onboarding and offboarding steps.
- **IdP groups mapped to GitHub teams** reduce repeated membership changes in GitHub.
- **Enterprise teams** can centralize some access patterns.
- **Automatic org membership** from team sync shortens time-to-access.
- **Suspension and restoration** occur through standard identity operations rather than bespoke GitHub cleanup.

Operational caveats still matter:

- Each team can connect to only one IdP group.
- Existing manual members must be removed before enabling sync.
- Some IdP-specific limitations apply, such as Entra security-group-only support and no nested groups.

### Developer experience

EMU changes developer experience, but not only in restrictive ways.

Positive outcomes often include:

- **Consistent sign-in path** through the corporate IdP.
- **Internal repositories for InnerSource** across the enterprise.
- **Predictable access flows** when teams are group-synced from the IdP.
- **Copilot Business or Copilot Enterprise support** for managed users.

Developers do need to adapt to the new model:

- They authenticate through the IdP, not with GitHub passwords or GitHub-native 2FA on the managed account.
- They may need to keep a separate personal GitHub account for open-source or external work.
- They must pay attention to Git configuration and account switching if they use both identities.

GitHub Docs explicitly recommend supporting developers who use multiple accounts.

### Cost and risk

The strongest EMU business cases typically combine **risk reduction** and **operational simplification**, not just tooling preference.

Potential value levers to quantify internally:

- Lower manual effort for onboarding, access changes, and offboarding.
- Reduced time between employment status change and GitHub access suspension.
- Fewer exceptions created by uncontrolled usernames or profile emails.
- Lower leakage risk from internal work being performed with public-capable identities.
- Better fit for regulated or residency-bound environments.

Potential costs and risks to include honestly:

- Migration effort to a **new** enterprise account.
- Developer change-management effort.
- Feature trade-offs, especially for external collaboration and public workflows.
- Possible architecture changes for automation, Actions, team design, and service accounts.
- Possible GHE.com feature gaps if you need data residency.

Do not fabricate ROI numbers. Build the case from your own baseline data.

## Comparison: personal accounts enterprise vs EMU

| Dimension | Enterprise with personal accounts | EMU | Executive implication |
| --- | --- | --- | --- |
| Account authority | User creates and owns the GitHub account | IdP provisions and governs the managed account | EMU provides stronger enterprise identity ownership |
| Authentication | SAML links enterprise identity to the personal account model | SAML or OIDC via the IdP, depending on supported design | EMU centralizes sign-in control |
| Provisioning scope | SCIM provisions organization access | SCIM provisions the managed user account lifecycle | EMU strengthens joiner-mover-leaver automation |
| Username control | User-controlled identity model | Username input comes from the IdP and enterprise shortcode | EMU improves naming consistency |
| Profile email control | User account model | Email comes from the IdP; only one email is supported | EMU improves control but raises email-design importance |
| Offboarding | Enterprise access is removed from the personal account context | Deprovisioning suspends the managed account and invalidates sessions | EMU is cleaner for audit and response workflows |
| Public repositories | Possible from personal accounts | Not allowed from managed accounts | EMU reduces public publication risk |
| Outside collaboration | Possible from personal accounts | Not allowed from the managed account | EMU is poor fit where broad external collaboration is routine |
| Repository visibility | Depends on enterprise design | EMU enterprise orgs use private and internal repos only | EMU favors enterprise-only collaboration |
| InnerSource | Possible by enterprise design | Internal repos are a built-in enterprise-wide sharing pattern | EMU supports internal open collaboration |
| Data residency on `ghe.com` | Not the EMU operating model | Requires EMU | EMU is the path when residency is mandatory |
| Developer account model | One account can often span internal and external work | Many developers need separate managed and personal accounts | EMU requires stronger account hygiene |
| Copilot licensing | Depends on enterprise licensing | Managed users need Copilot Business or Enterprise | Plan licensing and rollout explicitly |
| Migration path | Existing model continues in place | Requires a new enterprise and migration | EMU should be sponsored as a program |

## When EMU is not the right fit

GitHub Docs explicitly note that **EMU is not the best solution for every customer**.

EMU may be the wrong fit when one or more of the following is true:

- You need developers to contribute to public repositories from the same account they use for internal work.
- You rely heavily on broad collaboration outside the enterprise from the same daily identity.
- You need features that are unavailable with EMU or currently unavailable on `ghe.com` for data residency.
- Your IdP pattern is unsupported or operationally fragile.
- You cannot tolerate migration to a new enterprise account.
- Your developer population is unlikely to accept separate managed and personal accounts.

Use these documents before deciding against EMU or designing around its trade-offs:

- [02-EMU fundamentals and decision guide](02-emu-fundamentals-and-decision-guide.md)
- [08-pitfalls and mitigations](08-pitfalls-and-mitigations.md)
- [04-identity and access](04-identity-and-access.md)
- [07-post-migration and change management](07-post-migration-and-change-management.md)

> **Note**
> A common compromise is to use EMU for internal enterprise work and require separate personal accounts for external/open-source work. That can be effective, but it introduces user training, account-switching complexity, and a real risk of using the wrong account if controls and guidance are weak.

## Business-case template

Use the template below to build an organization-specific case without inventing benefits.

| Business-case input | What to measure now | Why it matters | Likely source |
| --- | --- | --- | --- |
| Onboarding effort | Steps, approvals, and manual touchpoints to grant GitHub access | Shows automation opportunity from SCIM and group sync | Platform team runbooks, service desk data |
| Offboarding latency | Time from HR termination event to GitHub access suspension | Quantifies lifecycle risk reduction | IAM team, HRIS-to-IdP workflow data |
| Access review effort | Time spent validating team and org membership | Shows governance simplification potential | IAM, security governance |
| Identity inconsistency | Count of naming/email exceptions or attribution issues | Highlights value of IdP-controlled profile data | Security, platform, audit teams |
| External collaboration dependency | Repos, workflows, or teams that require outside collaboration | Identifies where EMU may hurt delivery if unmanaged | Engineering leadership |
| Public publishing dependency | Public repos, gists, Pages, or open-source contribution expectations | Flags non-fit conditions early | Engineering, OSS office |
| Residency requirement | Regulatory, contractual, or customer requirements by region | Determines whether `ghe.com` is necessary | Legal, privacy, sales, compliance |
| Migration scope | Number of orgs, repos, teams, Actions assets, and users | Estimates program size and sequencing | Platform engineering inventory |
| Feature gap exposure | Current use of capabilities unavailable on `ghe.com` or EMU | Prevents avoidable surprises | Platform engineering, developer experience |
| Copilot target state | Which users need Copilot and under which license | Aligns rollout cost and enablement | Engineering leadership, procurement |
| Change-management load | Number of developers needing multiple-account guidance | Sizes training and support effort | Developer experience, support |
| Security incident pattern | Incidents or near misses involving identity misuse or unintended exposure | Helps connect EMU to risk reduction | Security operations |

A simple executive worksheet can use four columns:

| Driver | Baseline | Expected change with EMU | Confidence / notes |
| --- | --- | --- | --- |
| Access lifecycle control |  |  |  |
| Leakage risk from public-capable identities |  |  |  |
| Administrative effort |  |  |  |
| Residency or regulatory fit |  |  |  |
| Developer friction |  |  |  |
| Migration effort and disruption |  |  |  |

Suggested discussion prompts:

1. Which risks are we trying to reduce that the personal-account model cannot reduce enough?
2. Which EMU restrictions are acceptable, and which would force exceptions or parallel processes?
3. Do we need `ghe.com` data residency, and if so, which region?
4. Can we standardize commit email, IdP attributes, and team/group ownership before migration?
5. Can we fund the migration as a program rather than an admin project?

## What changes for developers

The table below is useful for change-management messaging.

| Topic | Before EMU (typical personal-account model) | After EMU | Migration impact |
| --- | --- | --- | --- |
| Sign-in | Developer signs in with a personal GitHub account linked to enterprise SSO | Developer signs in through the IdP for the managed account | Retrain sign-in flow and recovery process |
| Password and 2FA | Managed at the personal account level | No GitHub password or GitHub-native 2FA for managed users; auth is via IdP | Coordinate with IAM and support teams |
| Username | Personal GitHub username | Username derived from IdP input plus enterprise shortcode on GitHub.com; different rules on `ghe.com` | Update scripts, docs, and naming expectations |
| Profile email | User-managed pattern | Email comes from IdP; only one email supported | Align commit email strategy early |
| Internal collaboration | Uses org membership and team access | Uses org membership, team access, and optionally IdP group sync | Access becomes more standardized |
| Outside collaboration | Same account can often work externally | Managed account cannot collaborate outside enterprise | Many developers need a separate personal account |
| Public repositories and gists | Possible from the personal account | Not available from managed accounts | Update OSS contribution guidance |
| Repository visibility | Depends on current design | EMU org repositories are private or internal | Internal repos become a primary InnerSource mechanism |
| Copilot | Personal/enterprise licensing varies | Managed users need Copilot Business or Enterprise | Re-plan licensing and setup |
| Commit attribution | Often tied to whatever email the user commits with | Must align with the IdP email strategy because EMU supports only one email from the IdP | Fix email mapping before cutover |
| Team membership | Often managed partly in GitHub | Can be managed through IdP groups linked to teams | Clarify source of truth |
| Account switching | Often unnecessary | Required when a developer also uses a personal GitHub account | Provide training for web, Git, and credential tools |
| Visibility to external users | Normal GitHub identity visibility model | Managed users are visible only to enterprise members | Adjust collaboration expectations |
| User-owned repositories | Depends on enterprise policy | If allowed, managed-user repos are private only and can invite only enterprise members | Revisit personal sandbox patterns |

> **Warning**
> Developer experience problems in EMU are usually not caused by GitHub alone. They are often caused by unclear identity ownership, poor email planning, weak multiple-account guidance, or late decisions about which work must remain outside the enterprise boundary.

## Recommended next reads

- [../README.md](../README.md)
- [02-EMU fundamentals and decision guide](02-emu-fundamentals-and-decision-guide.md)
- [03-migration strategy and planning](03-migration-strategy-and-planning.md)
- [04-identity and access](04-identity-and-access.md)
- [05-governance, security, and compliance](05-governance-security-compliance.md)
- [06-technical migration guide](06-technical-migration-guide.md)
- [07-post-migration and change management](07-post-migration-and-change-management.md)
- [08-pitfalls and mitigations](08-pitfalls-and-mitigations.md)
- [09-references and research notes](09-references-and-research-notes.md)
- [../checklists/migration-checklist.md](../checklists/migration-checklist.md)

## Official documentation

Core GitHub Docs pages used or referenced here:

- Enterprise Managed Users overview: https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users
- Choosing an enterprise type: https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/enterprise-fundamentals/choose-an-enterprise-type
- Abilities and restrictions of managed users: https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/abilities-and-restrictions-of-managed-user-accounts
- GitHub Enterprise Cloud with data residency: https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency

For implementation details, continue with [02-EMU fundamentals and decision guide](02-emu-fundamentals-and-decision-guide.md) and [09-references and research notes](09-references-and-research-notes.md).
