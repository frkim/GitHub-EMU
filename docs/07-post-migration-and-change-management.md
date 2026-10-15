# Post-migration and change management for GitHub Enterprise Managed Users

Purpose: provide the operational runbook, follow-up task list, developer re-onboarding guidance, hypercare model, and legacy decommission approach needed after each EMU migration wave completes.

> Last verified: 2026-10-15 against GitHub Docs

## Table of contents

- [How to use this document](#how-to-use-this-document)
- [Post-migration operating model](#post-migration-operating-model)
- [Cutover runbook](#cutover-runbook)
- [Validation and soundness checks](#validation-and-soundness-checks)
- [Official follow-up tasks after migration](#official-follow-up-tasks-after-migration)
- [Developer re-onboarding](#developer-re-onboarding)
- [CI/CD and integration repointing](#cicd-and-integration-repointing)
- [Communication and training plan](#communication-and-training-plan)
- [Support model and hypercare](#support-model-and-hypercare)
- [Adoption metrics](#adoption-metrics)
- [Decommissioning legacy organizations](#decommissioning-legacy-organizations)
- [Continuous improvement](#continuous-improvement)
- [Post-migration checklist](#post-migration-checklist)
- [Related documents](#related-documents)

## How to use this document

Use this document after a pilot or production migration wave enters cutover.
It complements the strategy in [03-migration-strategy-and-planning.md](03-migration-strategy-and-planning.md), the migration mechanics in [06-technical-migration-guide.md](06-technical-migration-guide.md), the access design in [04-identity-and-access.md](04-identity-and-access.md), the control baseline in [05-governance-security-compliance.md](05-governance-security-compliance.md), the risk catalog in [08-pitfalls-and-mitigations.md](08-pitfalls-and-mitigations.md), the source notes in [09-references-and-research-notes.md](09-references-and-research-notes.md), the higher-level overview in [../README.md](../README.md), the rationale in [01-why-migrate-to-emu.md](01-why-migrate-to-emu.md), the fundamentals guide in [02-emu-fundamentals-and-decision-guide.md](02-emu-fundamentals-and-decision-guide.md), and the execution checklist in [../checklists/migration-checklist.md](../checklists/migration-checklist.md).
GitHub Docs make clear that a successful repository transfer is only part of the outcome.
Many operational elements are not migrated, and several migrated elements arrive disabled or incomplete.
That means post-migration work should be treated as a planned wave phase, not an afterthought.

> **Warning**
> Because GEI has no delta migration capability, changes made in the source during the final production migration are lost.
> The cutover runbook assumes the source has already entered a strict freeze.

## Post-migration operating model

After each wave, the program should shift quickly from migration execution to service restoration and user adoption.
A practical operating model has four lanes:

| Lane                  | Goal                                                              | Primary owners                   |
| --------------------- | ----------------------------------------------------------------- | -------------------------------- |
| Technical restoration | Re-enable or rebuild non-migrated platform components             | Platform engineering, org owners |
| Access restoration    | Ensure users, teams, and service identities can work              | IAM, org owners, support         |
| User adoption         | Help developers switch accounts, credentials, remotes, and habits | DevEx, enablement, service desk  |
| Control confirmation  | Re-apply governance and verify security posture                   | Security, enterprise owners      |

### Exit states to define up front

For each wave, define:

- when cutover starts
- when the destination becomes the system of record
- when critical integrations are restored
- when hypercare begins
- when hypercare ends
- when legacy orgs move to locked or archived status

## Cutover runbook

The cutover runbook should be owned by the TPM and executed with named owners for every step.
Use a single source of truth for timestamps, decision notes, and issue tracking.

### T-minus checklist

#### T-10 to T-5 business days

- [ ] Confirm source scope, destination scope, and freeze window.
- [ ] Reconfirm owners for every repo and every critical integration.
- [ ] Re-run inventory deltas for apps, hooks, Actions, LFS, and rulesets.
- [ ] Confirm service-desk staffing, office hours, and escalation routes.
- [ ] Confirm developers have the re-onboarding guide.
- [ ] Reconfirm identity mapping and target team/group assignments.
- [ ] Review trial-run findings and update timings.
- [ ] Validate ruleset bypass or temporary policy changes approved for migration.

#### T-4 to T-2 business days

- [ ] Confirm GEI credentials and authorizations are still valid.
- [ ] Confirm any required PAT SSO authorization if SAML SSO is involved.
- [ ] Confirm IP allow lists temporarily permit GEI if needed.
- [ ] Confirm Entra CAP exceptions or service-account design if CAP could block migration or automation.
- [ ] Confirm sandbox-to-production differences are documented.
- [ ] Confirm destination orgs exist and baseline settings are applied.
- [ ] Confirm communication templates and announcement channels.

#### T-1 business day

- [ ] Announce freeze start time and expected migration window.
- [ ] Pause planned releases or merges that would create source churn.
- [ ] Pause automations that write issues, PR comments, or labels if relevant.
- [ ] Confirm current webhook endpoints and app installations for restoration.
- [ ] Confirm backup/export expectations if required by policy; verify with GitHub Docs/Support for any product-specific export limitations.

#### T-0: freeze and final migration

1. Start the source freeze.
2. Verify no unauthorized source changes are occurring.
3. Execute final migration jobs.
4. Track migration IDs, status, and warnings.
5. Wait for migration completion.
6. Review logs before announcing success.

### T-zero execution model

| Step                            | Owner                  | Evidence                          |
| ------------------------------- | ---------------------- | --------------------------------- |
| Freeze source repos/orgs        | Org owners             | Freeze confirmation in change log |
| Start GEI job(s)                | Platform lead          | Migration IDs and command record  |
| Monitor job status              | Platform + TPM         | Status checks and logs            |
| Review warnings/errors          | Platform + repo owners | Logged disposition                |
| Validate destination visibility | Org owners             | Repo accessibility check          |
| Open hypercare bridge           | TPM                    | War-room / bridge opened          |

### Immediate post-completion tasks

- Validate that destination repos exist and are accessible.
- Validate that migrated repos have the intended visibility (`private` or `internal`).
- Validate default branch names.
- Validate whether code search is still re-indexing and communicate the expected delay.
- Repoint CI/CD, webhooks, and app integrations.
- Re-enable or rebuild Actions configuration.
- Start developer access validation.

### Announce go-live

A minimum go-live announcement should include:

- scope completed
- source is now read-only or retired for that wave
- destination URLs and owner contacts
- known limitations still being restored
- support channels and office hours

> **Note**
> If migrating to GHE.com, remember that API endpoints, IP ranges, and SSH fingerprints differ from GitHub.com.
> Repoint automation accordingly.

## Validation and soundness checks

Treat validation as a formal acceptance gate.
Separate repo-level checks from org-level checks.

### Repository-level checks

| Check                                                         | Why it matters                                          | Source fact or rationale    |
| ------------------------------------------------------------- | ------------------------------------------------------- | --------------------------- |
| Repository exists in expected destination org                 | Confirms routing                                        | Basic migration acceptance  |
| Visibility is correct (`private` or `internal`)               | Public is not allowed in EMU                            | Verified docs fact          |
| Default branch is correct                                     | Prevents CI/CD and PR confusion                         | Basic operational soundness |
| Git history is present                                        | Core migration success                                  | Repo migration acceptance   |
| PRs and issues are present                                    | Major timing drivers; core migrated content             | Verified docs fact          |
| Milestones, wiki, topics, settings are present where expected | Confirms supported content arrived                      | Verified docs fact          |
| Releases are present if not skipped and within size limits    | Large releases may have been skipped                    | Verified docs fact          |
| Branch protections match planned state                        | Some protection settings are not migrated               | Verified docs fact          |
| Webhooks are re-enabled and secrets restored                  | Hooks migrate disabled and without secrets              | Verified docs fact          |
| Apps are reinstalled                                          | App installations are not migrated                      | Verified docs fact          |
| Actions workflows exist and run as intended                   | Workflow files migrate, but operational config does not | Verified docs fact          |
| LFS content is available                                      | LFS objects are not migrated                            | Verified docs fact          |
| CODEOWNERS and team references still make sense               | Team membership and some team structures change         | Verified docs fact          |
| Critical links in docs/readmes still resolve                  | Cross-repo references may not survive perfectly         | Generic validation practice |

### Organization-level checks

| Check                                                        | Why it matters                           |
| ------------------------------------------------------------ | ---------------------------------------- |
| Org exists in the correct enterprise                         | Confirms governance boundary             |
| Owners can administer the org                                | Required for steady-state operations     |
| Intended teams exist                                         | Team structure restored                  |
| IdP group sync is connected where planned                    | Access restoration at scale              |
| Users joining synced teams are added to the org              | Expected behavior per docs               |
| Internal repos are visible to enterprise members as intended | Confirms InnerSource design              |
| Org webhooks are re-enabled                                  | Org-level integrations restored          |
| GHAS settings are correct                                    | Security baseline restored               |
| Rulesets and branch protections are re-applied or validated  | Prevents policy drift                    |
| IP allow list exceptions used for migration are cleaned up   | Return to baseline control               |
| Audit and evidence records are stored                        | Supports compliance and program closeout |

### Validation evidence pack

Store, at minimum:

- migration IDs and logs
- warning/error disposition notes
- repo and org validation checklist results
- access test results for representative users
- integration restoration evidence
- sign-off from repo or org owners

## Official follow-up tasks after migration

GitHub Docs explicitly call out the following follow-up items.
Plan them as named tasks, not as ad hoc cleanup.

### Follow-up task matrix

| Follow-up task                         | Why it is needed                                                                            | Notes                                                     |
| -------------------------------------- | ------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Check migration status and review logs | Warnings may require action                                                                 | Use logs as part of acceptance                            |
| Migrate Git LFS objects                | LFS objects are not migrated                                                                | Manual push required                                      |
| Set repository visibility              | Destination may default to private                                                          | Public is unavailable in EMU                              |
| Configure Actions                      | Secrets, variables, environments, runners, artifacts, and run history are not migrated      | Actions is disabled during migration and re-enabled after |
| Clean up IP allow list changes         | Temporary changes may have been made for GEI                                                | Return to baseline                                        |
| Restore GHAS posture                   | Some security data is not migrated                                                          | See GHAS subsection below                                 |
| Enable webhooks                        | Migrated active webhooks arrive disabled                                                    | Re-add secrets                                            |
| Reinstall GitHub Apps                  | Installations are not migrated                                                              | Reauthorize and validate permissions                      |
| Recreate or reconnect teams            | Team membership is not migrated in org migration; repo migration also requires team rebuild | Prefer IdP group sync where appropriate                   |
| Reclaim mannequins                     | Restores user attribution for non-commit activity                                           | Optional but usually important                            |
| Restore commit signature verification  | Signing keys and some web-commit verification do not carry over                             | User and support actions required                         |

### Git LFS follow-up

- Identify repos that use LFS.
- Push LFS objects to the destination after repo migration.
- Validate large-file workflows before closing the wave.

### Actions follow-up

GitHub Docs state the following are **not** migrated:

- Actions secrets
- Actions variables
- Actions environments
- self-hosted runners
- larger runners
- artifacts
- run history
  Operational guidance:
- Recreate secrets and variables from approved secret sources.
- Recreate environments and their protections.
- Re-register self-hosted or larger runners to the destination scope.
- Trigger a smoke workflow after repointing.
- Verify service accounts and network allow lists used by workflows.

### GitHub Advanced Security (GHAS) follow-up

Verified facts include:

- Enable GHAS on the destination org **before** migration if you want settings to migrate.
- Secret scanning performs a full rescan and remediation states are not preserved.
- Code scanning alerts are not migrated.
- Dependabot alerts are rebuilt from the default branch.
- Dependabot secrets must be re-added.
  Operational guidance:
- Reconfirm GHAS enablement at the destination org level.
- Re-upload SARIF if code scanning results are needed immediately.
- Rebuild secret scanning remediation workflow if prior states matter.
- Re-add Dependabot secrets before relying on automated updates.

### Webhooks follow-up

GitHub Docs state active webhooks are migrated **disabled** and webhook secrets are not migrated.
The runbook should therefore include:

- endpoint validation
- secret retrieval from the system of record
- controlled re-enable
- test delivery confirmation
- owner sign-off

### App follow-up

GitHub Docs state GitHub Apps and app installations are not migrated.
After migration:

- reinstall required apps
- validate org and repo permissions
- revalidate callback URLs or host allow lists if hostnames changed
- run end-to-end tests for deployment or automation apps

### Team and access follow-up

For org migration, GitHub Docs state teams, team access to repos, and member privileges migrate, but **team membership does not**.
For EMU, a durable approach is usually IdP-group-to-team sync.
Remember the documented constraints:

- a synced team connects to one IdP group
- that same group can feed multiple teams
- synced teams cannot be parent/child teams
- manual members must be removed before converting a team to synced membership
- membership is then managed only in the IdP

### Mannequin reclaim

GitHub Docs define mannequins as placeholder identities for migrated activity other than Git commits.
Reclaim is optional and can happen after the migration, but only org owners can do it.
The target user must already be an org member.
If a repo will later move to another org, reclaim before that transfer.

#### Exact CLI commands

Generate the CSV of mannequins:

```bash
gh gei generate-mannequin-csv --github-target-org DEST --output FILE.csv
```

Reclaim from a CSV:

```bash
gh gei reclaim-mannequin --github-target-org DEST --csv FILE.csv
```

Reclaim a specific mannequin to a specific user:

```bash
gh gei reclaim-mannequin --github-target-org DEST --mannequin-user LOGIN --target-user USER
```

For EMU, reclaim immediately without an email invitation:

```bash
gh gei reclaim-mannequin --github-target-org DEST --mannequin-user LOGIN --target-user USER --skip-invitation
```

> **Note**
> Future migrations to the same org automatically reuse the same mannequin mapping.

### Commit signature verification

GitHub Docs state:

- user GPG/SSH signing keys are not migrated
- users must add signing keys to the new account
- users must verify the committer email on the new account
- web commits signed with a deployment-specific key may show **Unverified** after migration in some cases; GitHub Docs say to contact GitHub Support for that scenario
  Post-migration actions:
- publish developer instructions for adding signing keys
- validate commit email alignment with the IdP-provided email
- identify critical repos where signed-commit policy enforcement should be rechecked

## Developer re-onboarding

Managed users authenticate only through the IdP.
They do not use GitHub passwords or GitHub-native 2FA on the managed account.
That means user onboarding must be rewritten for EMU.

### What changes for developers

| Topic                | New expectation                                          |
| -------------------- | -------------------------------------------------------- |
| Username             | New managed username ends with `_SHORTCODE`              |
| Authentication       | Sign in through the IdP                                  |
| Credentials          | Old personal-account PATs and SSH keys do not carry over |
| Local Git remotes    | Update remotes to destination org/repo URLs              |
| Signing              | Re-add GPG/SSH signing keys and verify commit email      |
| Public collaboration | Use a separate personal account for open-source work     |

### Developer day-one checklist

- [ ] Sign in to the managed account through the IdP.
- [ ] Confirm the managed username and profile email are correct.
- [ ] Join the expected org(s) and team(s).
- [ ] Update local Git remotes.
- [ ] Create the credential type approved by enterprise policy; verify with GitHub Docs/Support if policy specifics are unclear.
- [ ] Re-add SSH keys if SSH is used.
- [ ] Re-add GPG or SSH signing keys if commit signing is required.
- [ ] Test clone, fetch, push, and pull request creation in the destination.
- [ ] Review dual-account guidance for open-source work.

### Updating Git remotes

Example HTTPS remote update:

```bash
git remote set-url origin https://github.com/DEST-ORG/REPO.git
```

Example SSH remote update:

```bash
git remote set-url origin git@github.com:DEST-ORG/REPO.git
```

If using GHE.com, substitute the correct host and verify host fingerprints, API URLs, and network rules.

### Credentials and Git Credential Manager

- Publish enterprise-standard credential guidance.
- If Git Credential Manager is used and SSO redirect is **not** enabled, GitHub Docs note that account filtering may need to be turned off for developers working with multiple accounts.
- Validate developer tooling flows for both browser and CLI paths.

### Working with two accounts for open source

GitHub Docs explicitly note that developers who need outside collaboration should keep a separate personal account.
Training should cover:

- when to use the managed account
- when to use the personal account
- how to keep local Git identities separate
- how to avoid accidentally pushing internal code from the wrong context

> **Warning**
> Dual-account workflows add real complexity.
> Treat them as a training topic, not as tribal knowledge.

## CI/CD and integration repointing

Many post-migration incidents come from integrations that still point at the source org or old credentials.
Track every integration to a named owner.

### Common repointing areas

| Area                          | What to update                                             |
| ----------------------------- | ---------------------------------------------------------- |
| External CI systems           | Repo URLs, webhooks, tokens, app installations             |
| Deployment tooling            | Source repo references, status callbacks, app permissions  |
| Secrets managers              | New PATs/SSH keys or app credentials as approved           |
| ChatOps / bots                | Org/repo identifiers, app installations, webhook endpoints |
| Ticketing / ALM integrations  | Repo links, issue references, auth configuration           |
| Source mirrors or backup jobs | Remote URLs and service identities                         |
| Compliance or archiving jobs  | API endpoints and org names                                |

### Repointing checklist

- [ ] Update destination repository URLs in all pipelines.
- [ ] Rotate credentials that referenced personal accounts or source org scope.
- [ ] Reinstall and reauthorize GitHub Apps.
- [ ] Re-enable and test webhooks.
- [ ] Re-register or reconnect runners.
- [ ] Run smoke builds and deployments.
- [ ] Update documentation and runbooks with new org/repo names.

### Special considerations

- If Entra Conditional Access Policy is used with OIDC, verify whether automation using PATs or SSH will be blocked and whether approved service-account exceptions exist.
- Deploy keys are not subject to CAP according to the verified facts, but verify whether deploy keys fit your control model.
- GitHub Apps acting as themselves or as installations are allowed according to the verified facts; this may make app-based automation preferable to user PATs.

## Communication and training plan

Post-migration communication should be persona-based.
Different groups need different instructions and evidence.

### Persona matrix

| Persona                | What they need to know                                                                  | Best format                                    |
| ---------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Developer              | New login, username, credentials, remotes, dual-account workflow, support contacts      | Quick-start guide, office hours, recorded demo |
| Team lead / maintainer | Freeze responsibilities, validation tasks, app/hook ownership, branch protection checks | Maintainer runbook and live briefing           |
| Org owner              | Owner access, team restoration, app reinstall approval, webhook validation              | Owner checklist and change log                 |
| IT / IdP admin         | Provisioning, deprovisioning, group sync, recovery, CAP exceptions                      | Technical runbook                              |
| Security               | GHAS restoration, ruleset validation, incident path, evidence collection                | Control checklist and review session           |

### Training package contents

- EMU overview and why the company is changing
- managed vs personal account behavior differences
- login flow and expected username format
- credential setup guidance
- remote-update examples
- signing key restoration guidance
- open-source dual-account guidance
- FAQ and escalation routes

### Suggested communication rhythm

- **Before cutover:** what is changing and when
- **At cutover:** freeze confirmation, go-live timing, current status
- **Immediately after:** destination ready, actions required, known issues
- **During hypercare:** daily update, top known issues, next office hours
- **At hypercare close:** steady-state support model and remaining backlog

## Support model and hypercare

Hypercare should be staffed as an explicit service window.
Do not treat it as ad hoc heroics.

### Recommended hypercare components

| Component         | Purpose                                            |
| ----------------- | -------------------------------------------------- |
| War-room / bridge | Fast triage for critical blockers                  |
| Office hours      | Lightweight help for developer setup and remotes   |
| FAQ               | Repeated answers for predictable onboarding issues |
| Ticket triage     | Prioritize break/fix work and route ownership      |
| Daily checkpoint  | Publish status and top blockers                    |

### Triage model

| Severity | Example                                             | Target response                    |
| -------- | --------------------------------------------------- | ---------------------------------- |
| Sev 1    | Production deployment blocked for a critical repo   | Immediate bridge escalation        |
| Sev 2    | Team cannot access required repos after cutover     | Same-business-day response         |
| Sev 3    | Individual remote or credential setup issue         | Office hours or service desk queue |
| Sev 4    | Documentation clarification or non-blocking request | Backlog / FAQ update               |

### Common hypercare issue categories

- user not provisioned or not in expected team
- webhook not re-enabled
- app not reinstalled
- Actions secret or environment missing
- runner not registered
- branch protection mismatch
- commit signing or email mismatch
- local Git credential confusion
- dual-account workflow confusion

## Adoption metrics

Use post-migration metrics to decide when a wave can leave hypercare.

| Metric                                          | Why it matters                       |
| ----------------------------------------------- | ------------------------------------ |
| Successful sign-ins to managed accounts         | Confirms identity readiness          |
| Percentage of users with restored repo access   | Confirms access restoration          |
| CI/CD smoke-test success rate                   | Confirms delivery pipeline readiness |
| Number of critical integration failures         | Measures platform stabilization      |
| Open hypercare tickets by severity              | Measures support burn-down           |
| Mannequin reclaim completion for critical repos | Measures attribution restoration     |
| Team sync completion                            | Measures steady-state access model   |
| Commit signature restoration completion         | Measures control restoration         |

> **Note**
> Use trend lines over the first few waves.
> The goal is not just success, but a shrinking support burden and a more repeatable playbook.

## Decommissioning legacy organizations

Decommission only after the destination is accepted and records requirements are satisfied.
The exact retention and legal-hold approach must be confirmed locally and, where product behavior matters, **verify with GitHub Docs/Support**.

### Recommended sequence

1. Confirm destination repos and integrations are stable.
2. Remove or reduce legacy write access according to the approved shutdown plan.
3. Lock or archive source repos/orgs where appropriate.
4. Remove personal-account memberships that should no longer retain access.
5. Preserve or export records required by policy.
6. Rename source orgs if you need to reuse names in the destination.
7. Delete legacy orgs only if legal, retention, and supportability questions are fully cleared.

### Rename vs delete guidance

GitHub Docs state that org and user names share a namespace, and that enterprise names share a namespace.
GitHub Docs also state that if you want to reuse names, you should prefer **renaming** the source account rather than deleting it, because renaming frees the name immediately.
Practical guidance:

- prefer rename when destination naming matters
- prefer lock/archive before delete when you need a cooling-off period
- treat deletion workflow details and retention impacts as **verify with GitHub Docs/Support** if not already established in policy

### Data retention and evidence

Plan to retain or export, as required by policy:

- migration logs
- validation evidence
- access approval records
- decommission approvals
- any required audit log extracts
  GitHub Docs verified for this project do **not** specify your required retention period.
  Use enterprise policy and **verify with GitHub Docs/Support** where product behavior is material.

## Continuous improvement

The program should not end with the last migration wave.
EMU changes identity, support, and governance operating models.

### Quarterly governance review topics

- team-to-IdP-group design quality
- effectiveness of internal vs private repo usage
- service-account and automation patterns
- GHAS and ruleset drift
- open-source dual-account pain points
- onboarding/offboarding effectiveness
- support ticket themes and documentation gaps

### Keep guidance current

- Review GitHub Docs for EMU, GEI, and GHAS updates.
- Review the GitHub changelog for new product announcements.
- Recheck unsupported or preview items before changing design assumptions.
- Revalidate GHE.com feature availability if data residency is in scope.

> **Note**
> The research set for this repository could not verify current changelog entries directly.
> Track the GitHub changelog continuously rather than relying on static migration documentation alone.

## Post-migration checklist

- [ ] Freeze completed and destination declared system of record.
- [ ] Migration logs reviewed and warnings dispositioned.
- [ ] Repo-level validation completed.
- [ ] Org-level validation completed.
- [ ] Webhooks re-enabled and secrets restored.
- [ ] GitHub Apps reinstalled and tested.
- [ ] Actions secrets, variables, environments, and runners restored.
- [ ] LFS objects pushed where needed.
- [ ] GHAS posture restored.
- [ ] Teams and IdP group sync validated.
- [ ] Mannequins reclaimed where required.
- [ ] Commit signing guidance completed.
- [ ] Developer re-onboarding support delivered.
- [ ] Legacy decommission plan approved.
- [ ] Hypercare exit criteria met.

## Related documents

- [../README.md](../README.md)
- [01-why-migrate-to-emu.md](01-why-migrate-to-emu.md)
- [02-emu-fundamentals-and-decision-guide.md](02-emu-fundamentals-and-decision-guide.md)
- [03-migration-strategy-and-planning.md](03-migration-strategy-and-planning.md)
- [04-identity-and-access.md](04-identity-and-access.md)
- [05-governance-security-compliance.md](05-governance-security-compliance.md)
- [06-technical-migration-guide.md](06-technical-migration-guide.md)
- [08-pitfalls-and-mitigations.md](08-pitfalls-and-mitigations.md)
- [09-references-and-research-notes.md](09-references-and-research-notes.md)
- [../checklists/migration-checklist.md](../checklists/migration-checklist.md)
