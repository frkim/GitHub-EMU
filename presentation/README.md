# EMU migration presentation (60 minutes)

A one-hour, GitHub-styled slide deck that summarizes the guides in this repository. It is written in [Marp](https://marp.app/) Markdown and exported to an HTML deck.

| File | Purpose |
|---|---|
| [`emu-migration.md`](emu-migration.md) | **Source** of the deck (Marp Markdown, includes speaker notes and timings) |
| [`emu-migration.html`](emu-migration.html) | **Generated** HTML deck – open it in a browser (keep it next to the `assets/` folder) |
| [`assets/`](assets) | Diagrams (hand-made SVG), a QR code and [Octicons](https://primer.style/octicons/) (MIT licensed, © GitHub Inc.) |

## Present it

1. Open `emu-migration.html` in a browser.
2. Use the arrow keys to navigate, `f` for full screen, and `p` for the **presenter view** with speaker notes and a timer.

## Agenda and timing

| Min | Section | Source |
|---:|---|---|
| 3 | Opening and objectives | – |
| 7 | 1 · Why migrate to EMU | [docs/01](../docs/01-why-migrate-to-emu.md) |
| 7 | 2 · EMU fundamentals and decision guide | [docs/02](../docs/02-emu-fundamentals-and-decision-guide.md) |
| 7 | 3 · Strategy and planning | [docs/03](../docs/03-migration-strategy-and-planning.md) |
| 7 | 4 · Identity and access | [docs/04](../docs/04-identity-and-access.md) |
| 6 | 5 · Governance, security and compliance | [docs/05](../docs/05-governance-security-compliance.md) |
| 9 | 6 · Technical migration | [docs/06](../docs/06-technical-migration-guide.md) |
| 4 | 7 · Post-migration and change management | [docs/07](../docs/07-post-migration-and-change-management.md) |
| 3 | 8 · Pitfalls and mitigations | [docs/08](../docs/08-pitfalls-and-mitigations.md) |
| 2 | Wrap-up and next steps | [checklist](../checklists/migration-checklist.md) |
| 5 | Q&A | – |

## Rebuild the HTML

Run from this folder (the options in [`.marprc.yml`](.marprc.yml) are picked up automatically):

```bash
npx @marp-team/marp-cli@latest emu-migration.md -o emu-migration.html
```

Live preview while editing:

```bash
npx @marp-team/marp-cli@latest -s .
```

Other formats (PDF/PPTX need a local Chrome/Chromium and `--allow-local-files`):

```bash
npx @marp-team/marp-cli@latest emu-migration.md --pdf --allow-local-files
npx @marp-team/marp-cli@latest emu-migration.md --pptx --allow-local-files
```

You can also edit and preview the deck with the [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) extension.

## Keeping the deck accurate

The Markdown guides in `docs/` remain the single source of truth. When a fact changes there, update the matching slide and the "last verified" date on the title slide, then rebuild the HTML.
