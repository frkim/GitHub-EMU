# 08. Pitfalls and mitigations

This catalogue lists the most common failure modes and planning mistakes when moving from GitHub.com organizations backed by personal accounts to Enterprise Managed Users (EMU). Use it as a risk register during discovery, trial migrations, production cutover, and hypercare.

> Last verified: 2026-10-15 against GitHub Docs

## Table of contents

- [How to use this catalogue](#how-to-use-this-catalogue)
- [Related documents](#related-documents)
- [Rating scale](#rating-scale)
- [Decision and strategy pitfalls](#decision-and-strategy-pitfalls)
- [Identity and username pitfalls](#identity-and-username-pitfalls)
- [Access and collaboration pitfalls](#access-and-collaboration-pitfalls)
- [Technical migration pitfalls](#technical-migration-pitfalls)
- [Data fidelity and attribution pitfalls](#data-fidelity-and-attribution-pitfalls)
- [Security and policy pitfalls](#security-and-policy-pitfalls)
- [Actions, CI/CD, and integration pitfalls](#actions-cicd-and-integration-pitfalls)
- [Copilot, Codespaces, Pages, Apps, and GHE.com capability pitfalls](#copilot-codespaces-pages-apps-and-ghecom-capability-pitfalls)
- [Organization, people, and change-management pitfalls](#organization-people-and-change-management-pitfalls)
- [Cost, licensing, and decommissioning pitfalls](#cost-licensing-and-decommissioning-pitfalls)
- [Top 10 pitfalls (executive view)](#top-10-pitfalls-executive-view)
- [Pre-mortem questions](#pre-mortem-questions)

## How to use this catalogue

- Pair this file with the [technical migration guide](06-technical-migration-guide.md), the [strategy document](03-migration-strategy-and-planning.md), and the [identity guide](04-identity-and-access.md).
- Treat each pitfall as a checkpoint during readiness reviews.
- Where a mitigation requires features or commands not explicitly verified here, validate the exact implementation with [GitHub documentation and research notes](09-references-and-research-notes.md) or GitHub Support.

## Related documents

- [Repository overview](../README.md)
- [Why migrate to EMU](01-why-migrate-to-emu.md)
- [EMU fundamentals and decision guide](02-emu-fundamentals-and-decision-guide.md)
- [Migration strategy and planning](03-migration-strategy-and-planning.md)
- [Identity and access](04-identity-and-access.md)
- [Governance, security, and compliance](05-governance-security-compliance.md)
- [Technical migration guide](06-technical-migration-guide.md)
- [Post-migration and change management](07-post-migration-and-change-management.md)
- [References and research notes](09-references-and-research-notes.md)
- [Migration checklist](../checklists/migration-checklist.md)

## Rating scale

| Rating             | Meaning                                                  |
| ------------------ | -------------------------------------------------------- |
| Likelihood: Low    | Uncommon if the program follows standard GitHub guidance |
| Likelihood: Medium | Common in large or fast-moving migrations                |
| Likelihood: High   | Expected unless actively mitigated                       |
| Impact: Low        | Local inconvenience or small operational delay           |
| Impact: Medium     | Important team disruption or rework                      |
| Impact: High       | Program-level delay, compliance risk, or broken cutover  |

> **Warning**
> EMU is not the best solution for every customer. The biggest risks usually come from underestimating identity, collaboration, and platform restrictions rather than from the copy operation itself.

## Decision and strategy pitfalls

### P-01 — Assuming EMU can be enabled in place on an existing enterprise

| Field                          | Detail                                                                                                                                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Teams assume EMU is a switch on the current enterprise. It is not. Existing enterprises using personal accounts cannot be converted in place, so the migration is delayed while a new enterprise is created. |
| Likelihood / impact            | High / High                                                                                                                                                                                                  |
| Mitigation / prevention        | Decide on the enterprise type before creation. Budget time for a net-new enterprise, destination org creation, and migration waves.                                                                          |
| Detection / early warning sign | Stakeholders ask for an "in-place conversion" or expect current org URLs to remain unchanged without a migration plan.                                                                                       |

### P-02 — Picking the wrong shortcode

| Field                          | Detail                                                                                                                                                                 |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | On GitHub.com EMU, the shortcode is chosen at enterprise creation and cannot be changed. A poor shortcode affects every managed username and internal naming standard. |
| Likelihood / impact            | Medium / High                                                                                                                                                          |
| Mitigation / prevention        | Review the shortcode against branding, username length, and collision risk before enterprise creation.                                                                 |
| Detection / early warning sign | Teams debate naming only after the enterprise has already been created.                                                                                                |

### P-03 — Choosing an unsupported IdP combination

| Field                          | Detail                                                                                                                                                                                |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Programs try to mix Okta and Entra ID for SSO and SCIM in one EMU enterprise. GitHub Docs explicitly say the Okta + Entra combination is not supported and can cause SCIM API errors. |
| Likelihood / impact            | Medium / High                                                                                                                                                                         |
| Mitigation / prevention        | Use a single supported partner IdP for both authentication and provisioning whenever possible.                                                                                        |
| Detection / early warning sign | Identity design shows Okta for one function and Entra ID for the other in the same enterprise.                                                                                        |

### P-04 — Choosing OIDC without noticing the multi-enterprise tenant limit

| Field                          | Detail                                                                                                                                                                                    |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Entra OIDC is attractive, but if one tenant will back multiple enterprises, only the first can use OIDC; additional enterprises must use SAML. Ignoring this creates redesign work later. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                           |
| Mitigation / prevention        | Decide the target enterprise topology up front and confirm whether a single Entra tenant will back more than one enterprise.                                                              |
| Detection / early warning sign | The identity team plans several EMU enterprises from one Entra tenant but documents OIDC for all of them.                                                                                 |

### P-05 — Choosing EMU without validating public collaboration needs

| Field                          | Detail                                                                                                                                                                                                          |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Managed users cannot collaborate outside the enterprise and cannot create public content. If teams depend on open source contribution, public repos, gists, stars, or external PRs, the operating model breaks. |
| Likelihood / impact            | High / High                                                                                                                                                                                                     |
| Mitigation / prevention        | Inventory all public collaboration use cases and decide whether users need separate personal accounts or whether some repos should stay outside EMU.                                                            |
| Detection / early warning sign | Engineering teams mention OSS contribution, upstream forks, public Pages sites, or public support repos as business-critical.                                                                                   |

### P-06 — Choosing GHE.com without checking feature gaps

| Field                          | Detail                                                                                                                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Data residency can be the right choice, but GHE.com currently lacks some GitHub.com features, including GitHub Marketplace, macOS runners, and Maven/Gradle support in GitHub Packages. |
| Likelihood / impact            | Medium / High                                                                                                                                                                           |
| Mitigation / prevention        | Review the current GHE.com feature overview before committing to the target platform and treat gaps as explicit work items.                                                             |
| Detection / early warning sign | Program requirements assume Marketplace apps, macOS hosted runners, or package types not available on GHE.com.                                                                          |

## Identity and username pitfalls

### P-07 — Setup user lockout or setup-user PAT expiry

| Field                          | Detail                                                                                                                                                                                                            |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | The setup user is the only non-SCIM user in the enterprise. Losing its credentials, recovery codes, or SCIM PAT blocks enterprise identity administration and can stall the migration.                            |
| Likelihood / impact            | Medium / High                                                                                                                                                                                                     |
| Mitigation / prevention        | Create the setup user in a private session, enable 2FA, save all recovery codes, and store the credentials in the approved password manager. Use the documented no-expiration classic PAT for SCIM configuration. |
| Detection / early warning sign | Setup-user credentials are held by one person, or the team cannot show where recovery codes are stored.                                                                                                           |

### P-08 — Username conflicts across source identities

| Field                          | Detail                                                                                                                                                                                 |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | EMU usernames are derived from IdP values plus the enterprise shortcode. If two people normalize to the same managed username, only the first user is created and the other conflicts. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                        |
| Mitigation / prevention        | Run an identity collision review using the actual IdP source attribute and normalization rules before provisioning.                                                                    |
| Detection / early warning sign | Duplicate normalized usernames appear during a pilot or SCIM test.                                                                                                                     |

### P-09 — Username length and normalization surprises

| Field                          | Detail                                                                                                                                                                                                                      |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Non-alphanumeric characters are normalized, and the suffix consumes length. GitHub.com EMU allows 39 characters including the suffix; GHE.com allows only 30. Unexpected truncation or invalid names causes user confusion. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                                                             |
| Mitigation / prevention        | Test the actual managed username output for a sample of long or unusual IdP usernames before rollout.                                                                                                                       |
| Detection / early warning sign | UPNs or login attributes are already near the platform limits, or contain special characters that normalize to dashes.                                                                                                      |

### P-10 — Forgetting the single-email model

| Field                          | Detail                                                                                                                              |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | The IdP can provide only one email to a managed user. Changing that email later unlinks contribution history tied to the old email. |
| Likelihood / impact            | Medium / High                                                                                                                       |
| Mitigation / prevention        | Decide the canonical work email before provisioning and avoid changing it after cutover unless the history impact is accepted.      |
| Detection / early warning sign | HR or identity teams treat email changes as harmless cleanup after users are already active in EMU.                                 |

### P-11 — Commit-email mismatch causes unattributed commits

| Field                          | Detail                                                                                                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Git commit authorship follows commit email, not mannequin mapping. If the managed account email does not match historical commit emails, commit attribution remains incorrect. |
| Likelihood / impact            | High / High                                                                                                                                                                    |
| Mitigation / prevention        | Compare historical commit emails with the IdP-managed email before migration and give developers a post-cutover Git email checklist.                                           |
| Detection / early warning sign | Pilot repos show many commits without attribution to the intended managed user account.                                                                                        |

### P-12 — Treating IdP group sync as a generic directory sync

| Field                          | Detail                                                                                                                                                                                                                                              |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | GitHub team sync has strict rules: one IdP group per team, no parent/child connected teams, and Entra supports only security groups—not nested or Microsoft 365 groups. Reconciliation is daily, so sync timing assumptions can also collapse late. |
| Likelihood / impact            | High / High                                                                                                                                                                                                                                         |
| Mitigation / prevention        | Design team sync specifically for GitHub rather than assuming existing directory patterns will work unchanged, and plan for daily reconciliation rather than instant propagation.                                                                   |
| Detection / early warning sign | The migration plan depends on nested groups, M365 groups, or parent-child team hierarchies.                                                                                                                                                         |

### P-13 — Deprovisioning surprises after IdP changes

| Field                          | Detail                                                                                                                                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | When a managed user is unassigned or deactivated in the IdP, GitHub suspends the account, invalidates sessions, and changes the username to a hash-based value. This can surprise operations teams during testing. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                                                    |
| Mitigation / prevention        | Train identity and support teams on SCIM deprovisioning behavior before pilot testing.                                                                                                                             |
| Detection / early warning sign | Test users disappear or appear under hashed names after IdP access changes.                                                                                                                                        |

## Access and collaboration pitfalls

### P-14 — Assuming team membership migrates with team objects

| Field                          | Detail                                                                                                                                                                   |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Organization migration can migrate team objects and team-to-repo access, but team membership does not migrate. Teams may exist in the target but have no usable members. |
| Likelihood / impact            | High / High                                                                                                                                                              |
| Mitigation / prevention        | Predefine destination teams and map them to IdP groups, then test sync before production cutover.                                                                        |
| Detection / early warning sign | The destination shows teams but users cannot actually access the repositories.                                                                                           |

### P-15 — Broken `@team` mentions and CODEOWNERS routing

| Field                          | Detail                                                                                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Team mentions and CODEOWNERS references are not updated during migration. Review routing, escalation paths, and ownership automation can silently fail. |
| Likelihood / impact            | High / High                                                                                                                                             |
| Mitigation / prevention        | Inventory all CODEOWNERS and high-value docs that reference teams, then update them after teams are recreated.                                          |
| Detection / early warning sign | Post-cutover review requests do not reach the expected teams or CODEOWNERS checks stop behaving as expected.                                            |

### P-16 — Forgetting that managed users cannot contribute outside the enterprise

| Field                          | Detail                                                                                                                                                                              |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Managed users can view public content but cannot push, open issues, create PRs, comment, react, fork, or star outside the enterprise. This breaks external collaboration workflows. |
| Likelihood / impact            | High / High                                                                                                                                                                         |
| Mitigation / prevention        | Identify outside collaboration requirements early and provide approved separate personal-account guidance where needed.                                                             |
| Detection / early warning sign | Teams discover only during pilot that they cannot open PRs upstream or maintain public forks.                                                                                       |

### P-17 — Dual-account leakage risk

| Field                          | Detail                                                                                                                                                                                                                   |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | The usual mitigation for outside collaboration is a separate personal account. Without clear browser, credential, and Git identity hygiene, developers can push internal code with the wrong account or commit identity. |
| Likelihood / impact            | High / High                                                                                                                                                                                                              |
| Mitigation / prevention        | Publish explicit guidance for browser profiles, Git credential storage, commit email settings, and account switching.                                                                                                    |
| Detection / early warning sign | Developers report confusion about which account is active in the browser, CLI, or local Git client.                                                                                                                      |

### P-18 — Misusing guest collaborator roles for internal access

| Field                          | Detail                                                                                                                                                                           |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Docs note that guest collaborators cannot access internal repositories except in orgs where they are added as members. Misunderstanding this produces unexpected access denials. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                  |
| Mitigation / prevention        | Use the right membership role model for internal repository access and validate guest use cases before cutover.                                                                  |
| Detection / early warning sign | Users expected to see internal repos can access private repos they were invited to, but not broad internal content.                                                              |

### P-19 — Assuming public repos, gists, or Pages will work the same way

| Field                          | Detail                                                                                                                                                                                                            |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | EMU organizations can host only private or internal repos. Managed users cannot create public content, gists are unavailable, and Pages has limitations. Existing public workflows may need redesign or archival. |
| Likelihood / impact            | High / High                                                                                                                                                                                                       |
| Mitigation / prevention        | Inventory public repos, gists, and Pages usage before migration and decide whether to convert, archive, or keep them outside EMU.                                                                                 |
| Detection / early warning sign | The source estate includes public documentation sites, public examples, or heavily used gists with no replacement plan.                                                                                           |

## Technical migration pitfalls

### P-20 — Classic PAT restrictions block GEI completely

| Field                          | Detail                                                                                                                                                   |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | GEI supports only classic PATs. If an organization has the policy that restricts classic PATs from accessing the organization, migration cannot proceed. |
| Likelihood / impact            | Medium / High                                                                                                                                            |
| Mitigation / prevention        | Check PAT policy early and obtain an approved exception or policy change before the migration window.                                                    |
| Detection / early warning sign | Security controls allow fine-grained PATs only, or the org has already blocked classic PATs by policy.                                                   |

### P-21 — Forgetting to SSO-authorize PATs

| Field                          | Detail                                                                                                                                                          |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | In SAML SSO organizations, a classic PAT can exist but still fail until it is SSO-authorized. This creates confusing permission errors during trial or cutover. |
| Likelihood / impact            | High / Medium                                                                                                                                                   |
| Mitigation / prevention        | Add explicit SSO authorization checks for both source and destination PATs to the readiness checklist.                                                          |
| Detection / early warning sign | The PAT has the correct scopes but API operations still fail in SSO-protected orgs.                                                                             |

### P-22 — IP allow lists block migration traffic

| Field                          | Detail                                                                                                                     |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Tight network controls can block GEI migration traffic. Migrations then fail even though PATs and permissions are correct. |
| Likelihood / impact            | Medium / High                                                                                                              |
| Mitigation / prevention        | Allow the documented GEI IP ranges for the migration window and test from the exact execution environment.                 |
| Detection / early warning sign | Authentication works, but migration requests fail only from the production runner or admin workstation.                    |

### P-23 — Conditional Access blocks GEI, Actions, Codespaces, or `github.dev`

| Field                          | Detail                                                                                                                                                                                                           |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | With Entra OIDC + CAP IP restrictions, GitHub enforces IP conditions for web, PAT, and SSH access. Actions using PATs can be blocked, and Codespaces or `github.dev` may not work because their IPs are dynamic. |
| Likelihood / impact            | High / High                                                                                                                                                                                                      |
| Mitigation / prevention        | Use a migration service account exempt from CAP, or allow the necessary IP ranges. Validate non-human access paths before cutover.                                                                               |
| Detection / early warning sign | Test users can sign in, but CLI automation or browser-adjacent tools fail from expected environments.                                                                                                            |

### P-24 — Rulesets block the migration itself

| Field                          | Detail                                                                                                                                                              |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Repository or organization rulesets are not migrated and can also prevent imported history from being accepted. Commit-email domain rules are a documented example. |
| Likelihood / impact            | Medium / High                                                                                                                                                       |
| Mitigation / prevention        | Inventory rulesets early and create a migration bypass for the migration actor where needed.                                                                        |
| Detection / early warning sign | Trial migrations fail with policy or validation errors even though repository data itself is healthy.                                                               |

### P-25 — Ignoring the lack of delta migration

| Field                          | Detail                                                                                                                                                                  |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Teams treat trial and production as incremental sync steps. GEI does not support delta migration, so changes made after migration starts are lost from the target copy. |
| Likelihood / impact            | High / High                                                                                                                                                             |
| Mitigation / prevention        | Plan freeze windows and communicate clearly when the source becomes read-only for the wave.                                                                             |
| Detection / early warning sign | Teams continue merging PRs into the source during the migration window.                                                                                                 |

### P-26 — Forgetting that Git LFS objects are separate

| Field                          | Detail                                                                                                                                     |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | GEI does not migrate Git LFS objects. Repositories can appear successful but be functionally incomplete because binary assets are missing. |
| Likelihood / impact            | Medium / High                                                                                                                              |
| Mitigation / prevention        | Identify LFS repos during discovery and schedule a post-migration `git lfs push --all` task for each one.                                  |
| Detection / early warning sign | Repositories clone successfully after cutover, but build or runtime assets are missing.                                                    |

### P-27 — Hitting size or naming limits too late

| Field                          | Detail                                                                                                                                                                                                                     |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | GitHub Docs call out a 2 GiB single-commit limit, 255-byte ref limit, 100 MiB normal file limit, 400 MiB migration-time file limit, and release-size caveats over 10 GiB. Late discovery causes emergency history cleanup. |
| Likelihood / impact            | Medium / High                                                                                                                                                                                                              |
| Mitigation / prevention        | Run size and ref hygiene before the trial using `git-sizer` and repository-specific inspections.                                                                                                                           |
| Detection / early warning sign | Large monorepos, historical binaries, or unusually long branch or tag naming standards appear in the source estate.                                                                                                        |

### P-28 — Choosing organization migration when batching is required

| Field                          | Detail                                                                                                                                                |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | `migrate-org` sounds convenient, but it migrates all repositories in the source org at once, cannot skip subsets, and tops out at 5,000 repositories. |
| Likelihood / impact            | Medium / High                                                                                                                                         |
| Mitigation / prevention        | Use repository migrations for phased programs, large orgs, or any cutover that must be split by team or business unit.                                |
| Detection / early warning sign | Stakeholders ask for "all repos except these few" while the implementation plan is based on `migrate-org`.                                            |

### P-29 — Reusing names by deleting instead of renaming

| Field                          | Detail                                                                                                                                                                       |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Organizations, users, and enterprises share namespaces. If teams delete names instead of renaming the old source account, they create avoidable recovery and audit problems. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                              |
| Mitigation / prevention        | Prefer renaming the old source org or enterprise if you must free a name for reuse.                                                                                          |
| Detection / early warning sign | A cutover plan includes deleting source orgs simply to free a namespace.                                                                                                     |

### P-30 — Forgetting that migrated repos arrive private

| Field                          | Detail                                                                                                                                                                                       |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Repositories migrating into EMU are private by default and initially visible only to the migrator and org owners. Teams may misread this as an access failure rather than expected behavior. |
| Likelihood / impact            | High / Medium                                                                                                                                                                                |
| Mitigation / prevention        | Define the target visibility for every repository and communicate the expected private-first state during validation.                                                                        |
| Detection / early warning sign | Users report that migrated repos are missing, while org owners and migrators can see them.                                                                                                   |

## Data fidelity and attribution pitfalls

### P-31 — Leaving mannequins unreclaimed

| Field                          | Detail                                                                                                                                         |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Non-member activity migrates as mannequins. If key mannequins are not reclaimed, issue, PR, and comment attribution remains confusing.         |
| Likelihood / impact            | High / Medium                                                                                                                                  |
| Mitigation / prevention        | Generate the mannequin CSV right after each wave and reclaim high-value contributors early; use `--skip-invitation` for EMU where appropriate. |
| Detection / early warning sign | Important historical contributors appear only as mannequin identities in the destination org.                                                  |

### P-32 — Expecting mentions to rewrite to managed identities

| Field                          | Detail                                                                                                                                        |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | GEI does not rewrite mentions of users, teams, or orgs. Old handles remain in comments and docs, which can break routing and confuse readers. |
| Likelihood / impact            | High / Medium                                                                                                                                 |
| Mitigation / prevention        | Accept the limitation, then update the most important docs, templates, and CODEOWNERS references after cutover.                               |
| Detection / early warning sign | Pilot migrations show `@old-name` references everywhere in issues and PRs.                                                                    |

### P-33 — Losing fork relationships

| Field                          | Detail                                                                                                                                     |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | GEI does not migrate fork relationships. Workflows that rely on fork lineage, network graphs, or parent/child context lose that structure. |
| Likelihood / impact            | Medium / Medium                                                                                                                            |
| Mitigation / prevention        | Identify repositories whose process depends on fork relationships and redesign the workflow before cutover.                                |
| Detection / early warning sign | Teams depend on fork-based contribution models or internal fork trees.                                                                     |

### P-34 — Cross-repository links break silently

| Field                          | Detail                                                                                                                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Cross-repository issue and PR references, reciprocal links, and links to specific comments or events are not preserved reliably. Investigation trails and historical navigation weaken. |
| Likelihood / impact            | High / Medium                                                                                                                                                                           |
| Mitigation / prevention        | Capture critical cross-repo references before cutover and preserve the source as read-only for lookups.                                                                                 |
| Detection / early warning sign | Release management or incident workflows depend heavily on deep historical links across repositories.                                                                                   |

### P-35 — Assuming Projects, Packages, Discussions, rulesets, or custom properties will come across

| Field                          | Detail                                                                                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Several important collaboration and governance features are simply out of scope for GEI. If that manual work is not planned, the target org looks incomplete. |
| Likelihood / impact            | High / High                                                                                                                                                   |
| Mitigation / prevention        | Add explicit workstreams for every out-of-band feature and assign owners before the production window.                                                        |
| Detection / early warning sign | The migration plan focuses only on repositories and ignores package feeds, projects, discussions, or metadata frameworks.                                     |

### P-36 — Treating stars and watchers as preserved state

| Field                          | Detail                                                                                                                                                |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Stars and watchers are not migrated. Teams often notice only after cutover that notification preferences and lightweight popularity markers are gone. |
| Likelihood / impact            | High / Low                                                                                                                                            |
| Mitigation / prevention        | Communicate the loss up front and ask users to re-watch critical repositories after cutover.                                                          |
| Detection / early warning sign | Stakeholders ask where their watches or starred lists went in the target org.                                                                         |

### P-37 — Validating with code search too early

| Field                          | Detail                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Code search re-indexing can take hours after migration. Teams that validate too early conclude incorrectly that files or repos are missing. |
| Likelihood / impact            | Medium / Medium                                                                                                                             |
| Mitigation / prevention        | Separate object-count validation from search-based validation and give search time to finish indexing.                                      |
| Detection / early warning sign | Repositories are visible and cloneable, but search still returns incomplete results shortly after migration.                                |

### P-38 — Unexpectedly unverified commit signatures

| Field                          | Detail                                                                                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | User signing keys do not migrate, and some web commits can show as Unverified after migration. This can alarm security reviewers and developers. |
| Likelihood / impact            | Medium / Medium                                                                                                                                  |
| Mitigation / prevention        | Include signing-key re-registration and committer-email verification in the developer cutover checklist.                                         |
| Detection / early warning sign | Security dashboards or users report a spike in unverified historical commits right after migration.                                              |

## Security and policy pitfalls

### P-39 — Enabling GHAS too late

| Field                          | Detail                                                                                                                                                                                |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | GitHub Docs say GHAS should be enabled on the destination org before migration if you want supported settings to migrate. Missing that step creates avoidable rework and blind spots. |
| Likelihood / impact            | Medium / High                                                                                                                                                                         |
| Mitigation / prevention        | Make GHAS enablement a destination-readiness gate rather than a post-cutover afterthought.                                                                                            |
| Detection / early warning sign | Destination orgs exist, but security tooling enablement is still on the backlog days before trial.                                                                                    |

### P-40 — Assuming security findings and remediation states migrate

| Field                          | Detail                                                                                                                                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Code scanning results do not migrate, secret scanning remediation states do not migrate, and Dependabot alerts rebuild only from the default branch. Security teams can misread the target as clean or incomplete. |
| Likelihood / impact            | High / High                                                                                                                                                                                                        |
| Mitigation / prevention        | Plan security reseeding explicitly: upload SARIF again where needed, rerun secret scanning, and reapply remediation states through the REST API.                                                                   |
| Detection / early warning sign | The destination repo has GHAS enabled but shows no historical code-scanning findings or missing remediation context.                                                                                               |

### P-41 — Overlooking Actions secrets, variables, environments, and runner gaps

| Field                          | Detail                                                                                                                                             |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Workflow files migrate, but the surrounding execution context does not. Pipelines then fail even though the YAML looks correct in the destination. |
| Likelihood / impact            | High / High                                                                                                                                        |
| Mitigation / prevention        | Inventory all Actions dependencies and rebuild them before teams resume deployments.                                                               |
| Detection / early warning sign | The first workflow run after cutover fails immediately on missing secrets, environments, or runner labels.                                         |

### P-42 — Forgetting that GitHub Apps, webhook secrets, and installations do not migrate

| Field                          | Detail                                                                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | App integrations and webhooks often carry business-critical automation. GEI may bring the webhook object, but it arrives disabled and without its secret; apps do not reinstall themselves. |
| Likelihood / impact            | High / High                                                                                                                                                                                 |
| Mitigation / prevention        | Inventory every app, webhook, endpoint owner, and secret owner before production. Re-enable hooks only after the secret is back in place.                                                   |
| Detection / early warning sign | Downstream systems stop receiving events, or app-driven automation no longer posts status or comments after cutover.                                                                        |

## Actions, CI/CD, and integration pitfalls

### P-43 — Underestimating workflow breakage because the YAML migrated

| Field                          | Detail                                                                                                                                                                              |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Teams see workflow files in the target and assume CI/CD is ready. In reality, secrets, variables, environments, runner configuration, artifacts, and history are outside GEI scope. |
| Likelihood / impact            | High / High                                                                                                                                                                         |
| Mitigation / prevention        | Treat workflow files as only one dependency among many and rehearse end-to-end pipeline execution in the sandbox.                                                                   |
| Detection / early warning sign | Trial validation checks that YAML exists but does not actually run representative pipelines.                                                                                        |

### P-44 — Relying on Marketplace-delivered dependencies on GHE.com

| Field                          | Detail                                                                                                                       |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | GHE.com does not provide GitHub Marketplace, so app procurement or Marketplace-based workflows may not transfer cleanly.     |
| Likelihood / impact            | Medium / High                                                                                                                |
| Mitigation / prevention        | Inventory every Marketplace dependency before choosing GHE.com and verify replacement patterns with current docs or vendors. |
| Detection / early warning sign | CI/CD or governance workflows depend on Marketplace procurement or install flows.                                            |

### P-45 — Missing runner capability gaps on GHE.com

| Field                          | Detail                                                                                                                            |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | macOS GitHub-hosted runners are unavailable on GHE.com. Teams discover this only when mobile or Apple-platform builds cannot run. |
| Likelihood / impact            | Medium / High                                                                                                                     |
| Mitigation / prevention        | Validate platform-specific build needs during target platform selection and plan self-hosted alternatives if necessary.           |
| Detection / early warning sign | Repositories build iOS, macOS, or other Apple-platform artifacts but have no self-hosted fallback design.                         |

### P-46 — Forgetting package ecosystem differences on GHE.com

| Field                          | Detail                                                                                                                                        |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Maven and Gradle support in GitHub Packages is unavailable on GHE.com, so package publishing or dependency resolution can fail after cutover. |
| Likelihood / impact            | Medium / High                                                                                                                                 |
| Mitigation / prevention        | Inventory package types and confirm whether any target package ecosystem is unsupported on GHE.com.                                           |
| Detection / early warning sign | Java build pipelines depend on GitHub Packages for Maven or Gradle artifacts.                                                                 |

## Copilot, Codespaces, Pages, Apps, and GHE.com capability pitfalls

### P-47 — Assuming managed users can use Copilot like personal accounts

| Field                          | Detail                                                                                                                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Managed users cannot sign up for Copilot Free or Pro; they need Copilot Business or Enterprise. On GHE.com, some Copilot features are unavailable and extra IDE auth setup is required. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                         |
| Mitigation / prevention        | Treat Copilot as a separate readiness stream: licensing, IDE sign-in guidance, and platform-specific feature validation.                                                                |
| Detection / early warning sign | Developers expect to carry over existing personal Copilot entitlements unchanged.                                                                                                       |

### P-48 — Assuming Codespaces works everywhere the same way

| Field                          | Detail                                                                                                                                                                          |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Managed users can use only enterprise-owned Codespaces, not personal repos, external repos, or public templates. CAP IP restrictions can also break Codespaces or `github.dev`. |
| Likelihood / impact            | Medium / High                                                                                                                                                                   |
| Mitigation / prevention        | Validate Codespaces scenarios specifically for EMU and CAP before promising them as day-one tooling.                                                                            |
| Detection / early warning sign | Developer experience plans rely on personal template repos, public templates, or browser-based editing from restricted networks.                                                |

### P-49 — Treating GitHub Pages as unchanged

| Field                          | Detail                                                                                                                                   |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Pages has EMU-specific limits. Teams migrating documentation or microsites can discover late that their publishing model no longer fits. |
| Likelihood / impact            | Medium / Medium                                                                                                                          |
| Mitigation / prevention        | Review current Pages limits for EMU and validate every production site pattern before migration.                                         |
| Detection / early warning sign | Documentation, demos, or internal portals depend on Pages but have not been tested in the destination model.                             |

### P-50 — Assuming app behavior under CAP is the same for all app types

| Field                          | Detail                                                                                                                                                                                                                   |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | GitHub/OAuth apps acting on behalf of users can fail CAP checks because the app server IP is sent to the IdP, while GitHub Apps acting as themselves or as installations are allowed. Teams often miss this distinction. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                                                          |
| Mitigation / prevention        | Validate every critical app flow under CAP and document whether it uses user-delegated or app/installation identity.                                                                                                     |
| Detection / early warning sign | Only some apps fail under CAP even though others continue to work.                                                                                                                                                       |

### P-51 — Assuming managed users can create or use public developer identity features

| Field                          | Detail                                                                                                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Managed users cannot use gists, personalized profiles, or individual public collaboration features the way personal accounts do. This can surprise developer enablement teams. |
| Likelihood / impact            | Medium / Low                                                                                                                                                                   |
| Mitigation / prevention        | Reset expectations early and move internal how-to content to approved internal repositories or portals.                                                                        |
| Detection / early warning sign | Developer enablement materials reference gists, public profiles, or similar personal-account-centric practices.                                                                |

## Organization, people, and change-management pitfalls

### P-52 — Underestimating the migration timeline

| Field                          | Detail                                                                                                                                                                                                      |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Teams estimate based on repository count alone, even though GitHub Docs say duration depends mostly on pull requests and issues. Heavy collaboration history can turn a "small" repo into a long migration. |
| Likelihood / impact            | High / High                                                                                                                                                                                                 |
| Mitigation / prevention        | Use trial migrations to measure timing and build waves around PR/issue density, not just repo count.                                                                                                        |
| Detection / early warning sign | Scheduling discussions mention only repository count or code size, not collaboration history.                                                                                                               |

### P-53 — Weak communication around freezes and account changes

| Field                          | Detail                                                                                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Users hear "new GitHub account" or "brief freeze" only at the last minute, then continue working in the source or miss required cutover actions. |
| Likelihood / impact            | High / High                                                                                                                                      |
| Mitigation / prevention        | Publish freeze windows, account guidance, remote update instructions, and manual follow-up expectations well before cutover.                     |
| Detection / early warning sign | Pilot users ask basic questions about sign-in, repo location, or contribution freezes just before the wave starts.                               |

### P-54 — No hypercare plan after cutover

| Field                          | Detail                                                                                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Teams assume the migration ends when GEI reports success. In practice, webhooks, apps, teams, LFS, security findings, and signing keys often need rapid follow-up. |
| Likelihood / impact            | High / High                                                                                                                                                        |
| Mitigation / prevention        | Reserve hypercare staffing and define ownership for identity, apps, CI/CD, security, and developer support for every wave.                                         |
| Detection / early warning sign | There is no staffed support rota or war room for the first one to three business days after cutover.                                                               |

### P-55 — Decommissioning the source too early

| Field                          | Detail                                                                                                                                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | Because GEI does not delete the source, some teams rush to archive or remove access before manual validation and legal retention checks are complete. That eliminates the best fallback and audit reference. |
| Likelihood / impact            | Medium / High                                                                                                                                                                                                |
| Mitigation / prevention        | Keep the source read-only until validation, manual recreation, and hypercare are complete.                                                                                                                   |
| Detection / early warning sign | Decommissioning tasks are scheduled to begin immediately after the GEI copy finishes.                                                                                                                        |

## Cost, licensing, and decommissioning pitfalls

### P-56 — License overage or wrong commercial assumptions

| Field                          | Detail                                                                                                                                                                               |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | EMU on GitHub.com or GHE.com carries enterprise licensing implications, and Copilot for managed users requires Business or Enterprise licensing. Budget mismatches can slow rollout. |
| Likelihood / impact            | Medium / High                                                                                                                                                                        |
| Mitigation / prevention        | Reconcile enterprise seat counts, managed-user scope, and Copilot licensing before provisioning users.                                                                               |
| Detection / early warning sign | The staffing model or pilot user count changes, but no one revisits license assumptions.                                                                                             |

### P-57 — Forgetting GHE.com billing and platform choices change planning

| Field                          | Detail                                                                                                                                                                               |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why it happens / impact        | GHE.com brings data-residency benefits but also its own subdomain, API hostname, and billing options. If those differences are ignored, automation and financial planning can drift. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                                      |
| Mitigation / prevention        | Treat GitHub.com EMU and GHE.com as different operating environments and validate both technical and billing assumptions.                                                            |
| Detection / early warning sign | The program plan uses GitHub.com examples everywhere even though the chosen target is GHE.com.                                                                                       |

### P-58 — Decommission plans forget namespace strategy

| Field                          | Detail                                                                                                                                                                       |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Why it happens / impact        | Because names are shared across organizations, users, and enterprises, poor namespace planning can create last-minute pressure to delete source entities just to free names. |
| Likelihood / impact            | Medium / Medium                                                                                                                                                              |
| Mitigation / prevention        | Include namespace reuse rules in the cutover plan and prefer renaming to deletion.                                                                                           |
| Detection / early warning sign | Source teardown tasks are tightly coupled to destination naming with no fallback path.                                                                                       |

## Top 10 pitfalls (executive view)

| Rank | ID   | Pitfall                              | Why executives should care                                    | First control to require                           |
| ---- | ---- | ------------------------------------ | ------------------------------------------------------------- | -------------------------------------------------- |
| 1    | P-01 | Expecting in-place EMU conversion    | The entire program can be mis-scoped from day one             | Require a net-new enterprise and migration plan    |
| 2    | P-05 | Ignoring public collaboration limits | Can invalidate developer and OSS operating models             | Demand an outside-collaboration assessment         |
| 3    | P-11 | Commit-email mismatch                | Historical attribution quality drops immediately              | Require a commit-email readiness review            |
| 4    | P-14 | Team membership not migrated         | Users lose access even when repos are present                 | Make IdP-to-team mapping a hard gate               |
| 5    | P-23 | CAP blocks migration or tooling      | Cutover fails despite correct repo data                       | Approve a CAP-exempt migration service account     |
| 6    | P-25 | No delta migration                   | Source and target diverge during cutover                      | Enforce a source freeze window                     |
| 7    | P-35 | Major features are outside GEI scope | The target can look incomplete after a “successful” migration | Fund separate workstreams for non-GEI features     |
| 8    | P-41 | Actions context not migrated         | CI/CD fails after cutover                                     | Require an app and pipeline rehearsal in sandbox   |
| 9    | P-52 | Timeline underestimated              | Windows and staffing collapse under real data volumes         | Base estimates on PR/issue history, not repo count |
| 10   | P-55 | Source decommissioned too early      | Removes the best rollback and audit reference                 | Keep source read-only until hypercare is complete  |

## Pre-mortem questions

Use these questions before approving production cutover:

1. Have we explicitly accepted that EMU requires a **new enterprise** rather than an in-place conversion?
2. Is the enterprise **shortcode** approved by identity, branding, and platform teams?
3. Have we confirmed the **IdP pattern** is supported and that we are not mixing Okta and Entra ID in an unsupported way?
4. If we chose Entra OIDC, have we checked whether future additional enterprises from the same tenant will require SAML?
5. Did we inventory **all public repos, gists, Pages sites, OSS workflows, and upstream fork needs**?
6. Have we tested the **setup user**, recovery-code storage, and SCIM PAT handling?
7. Are the source and destination organizations able to use **classic PATs**, and are those PATs **SSO-authorized** where needed?
8. If Conditional Access is in play, is the **migration service account exempt** or otherwise allowed?
9. Did we document **branch protections, rulesets, webhook endpoints, app installations, Actions secrets/variables/environments, Packages, Projects, Discussions, and custom properties**?
10. Have we identified repositories that use **Git LFS**, large release assets, oversized files, oversized commits, or long refs?
11. Did we decide whether each wave will use **repository migration** or **organization migration**, and does that match the batching requirement?
12. Did we test **team sync** with actual IdP groups, including Entra security-group constraints?
13. Did we compare historical **commit emails** with the managed-user email that the IdP will send?
14. Did we rehearse **mannequin reclaim** and confirm org-owner coverage for it?
15. Did we enable **GHAS before migration** where required and define how code scanning and secret scanning states will be rebuilt?
16. Have we run a **trial migration** recently enough that source drift is still manageable?
17. Is there a staffed **hypercare plan** for identity, access, CI/CD, apps, security, and developer support?
18. Do we have an explicit **read-only source retention** period before any archive or decommission action begins?

> **Note**
> If several answers above are still uncertain, pause the production wave and return to the planning documents rather than letting the migration window discover those problems for you.
