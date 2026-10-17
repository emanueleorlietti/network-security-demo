# Network & Security Demo

A public, vendor-neutral Obsidian knowledge base demonstrating reusable technical documentation: commands, networking concepts, guides, diagnostic scripts and evidence-based troubleshooting cases.

The current examples focus on Linux/Windows networking and DNS. This is a small, connected demonstration of a documentation method, not a complete security reference or an account of production incidents.

## Start here

```bash
git clone https://github.com/emanueleorlietti/network-security-demo.git
```

Open the cloned folder as an Obsidian vault, then open **Dashboard.md**. On GitHub, use the links below; Obsidian wikilinks in note bodies are intended for the vault.

- [Dashboard](Dashboard.md)
- [Command Index](Index/Command%20Index.md)
- [Concept Index](Index/Concept%20Index.md)
- [Guide Index](Index/Guide%20Index.md)
- [Script Index](Index/Script%20Index.md)
- [Troubleshooting Index](Index/Troubleshooting%20Index.md)

## Actual vault structure

| Folder | Contents |
| --- | --- |
| Commands/ | Eight notes, including Linux, Windows and cross-platform tools; OS metadata controls the index tables. No platform subfolders are required. |
| Concepts/ | Five fundamentals: OSI, TCP/IP, IP addressing, TCP/UDP and DNS. |
| Guides/ | A connectivity procedure with prerequisites, validation and rollback considerations. |
| Scripts/ | A documented Bash collector with per-check timeouts and partial-failure reporting. |
| Troubleshooting/ | A simulated DNS failure, evidence sequence, correction and expected validation. |
| Templates/ | Six templates for the QuickAdd workflows below. |
| Index/ | Five category indexes with static links and Dataview tables. |

Attachments/ is the configured destination for future attachments; it is not populated in this repository.

## Note creation and plugins

The repository includes Dataview, QuickAdd and Templater configuration and plugin files. Enable community plugins only after reviewing and trusting their code. The core notes and static navigation remain readable without them. Dynamic tables require Dataview; QuickAdd and Templater provide the creation workflow.

| QuickAdd action | Folder | Template |
| --- | --- | --- |
| New Linux command | Commands | linux-command-template.md |
| New Windows command | Commands | windows-command-template.md |
| New concept | Concepts | concept-template.md |
| New guide | Guides | guide-template.md |
| New script | Scripts | script-template.md |
| New troubleshooting case | Troubleshooting | troubleshooting-template.md |

All templates are in Templates/. QuickAdd action IDs and destinations are preserved. Both command types share Commands/, so Templater deliberately has no automatic command-folder default: choose the correct QuickAdd action or insert the matching Templater template manually. Automatic folder templates cover Concepts, Guides, Scripts and Troubleshooting. Do not use Obsidian's core Templates insertion for Templater expressions.

## Documentation standards

- Preserve property names Command, Concept, Guide, Script, Case, Description, OS, Language, Difficulty, Status and Date where applicable. Template titles are JSON-quoted for valid YAML punctuation.
- Use lowercase, consistent tags: command, concept, guide, script and troubleshooting identify note categories. Platform/topic tags connect notes across categories. Reuse existing tags; choose singular forms where natural.
- OS values Linux, Windows and Linux / Windows drive command indexes. Use both platform tags for a shared tool.
- Explain syntax placeholders before execution. Use bash, powershell or text fences as appropriate; keep illustrative output separate from input.
- Document platform/tool versions, target/interface, privileges, impact and interpretation limits. Bound active diagnostics and explain cleanup.
- New cases start Under investigation. Record symptoms, context, observations and hypotheses; state a root cause only when evidence verifies it. Record actions actually performed and validate the original user workflow before marking a real case Resolved.
- Label lab cases, example outputs and expected results explicitly. The DNS case is Simulated - resolved; that status is part of the fictional scenario.
- Link actual notes, such as DNS, dig and DNS Resolution Failure. All current notes have unique filenames; use folder-qualified links if duplicate names are introduced.

## Lab scope and validation

Private addresses in the connected examples belong to a fictional 192.168.10.32/27 lab. Names under demo.test require local lab DNS. They are not live public targets. Public example domains and probe addresses are separate examples; adapt them to authorized destinations.

The HTTPS lab requires a configured service and a certificate trusted by the client for portal.demo.test. A DNS override does not disable certificate checks. A successful ping, TCP handshake or script completion does not by itself prove a healthy application.

The maintenance pass checked note metadata, links, template rendering cases, workflow mappings and shell/PowerShell syntax separately from runtime behavior. No live Linux/Windows lab or production service was exercised. Source references are attached to the relevant notes; syntax checks do not prove operational success.

## Public-repository hygiene

Use fictional roles and documentation values. Never publish credentials, tokens, private keys, customer identities, real organizational topology, full production configurations or raw diagnostic captures. Collector output may contain hostnames, IPs, search domains and listening sockets: redact it before sharing. Changes to current files do not remove content from earlier Git history.

## Git synchronization

Run from your checkout and confirm the remote repository before pulling:

```bash
git remote -v
git status
git pull --ff-only origin main
```

Commit or stash local edits before pulling if they conflict. Reapply a stash deliberately and resolve conflicts; do not discard local work just to synchronize.

## 🟣 Basic Git Commands

| Command | Purpose |
| --- | --- |
| `git status` | Inspect the branch and staged, unstaged and untracked changes. |
| `git add .` | Stage additions, modifications and deletions below the current directory. Use explicit paths when selecting files. |
| `git diff --cached` | Review the exact staged content before committing. |
| `git commit -m "YYYYMMDD-XX"` | Record the staged snapshot; use a meaningful message. |
| `git push` | Upload commits to the configured upstream. |
| `git pull --ff-only origin main` | Retrieve main and update only when a fast-forward is possible. |

## 🔵 Useful Obsidian Shortcuts

| Shortcut | Action |
| --- | --- |
| Ctrl/Cmd+E | Toggle editing and reading view. |
| Ctrl/Cmd+G | Open Graph View. |
| Ctrl/Cmd+O | Open Quick Switcher. |
| Ctrl/Cmd+P | Open Command Palette. |
| Ctrl/Cmd+R | Run QuickAdd (configured in this vault). |

Other shortcuts can be customized. Ctrl is typical on Windows/Linux and Cmd on macOS. Use Local Graph to explore related notes; zoom affects label visibility.

## Feedback

Corrections and focused contributions are welcome through GitHub Issues or pull requests. Keep examples reproducible, distinguish expected results from observed evidence and preserve the vendor-neutral scope.
