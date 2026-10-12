# Network & Security Demo

A vendor-neutral Obsidian knowledge base for organizing networking, cybersecurity, systems, and troubleshooting documentation.

This repository is a demonstration of the structure and methodology behind my personal technical knowledge base. It is designed as a reusable starting point rather than a complete technical encyclopedia.

---

## 🧠 Project Purpose

Technical notes often become scattered across documents, bookmarks, tickets, and temporary files.

This vault provides a structured system for:

- Documenting frequently used commands
- Explaining technical concepts
- Creating reusable technical guides
- Recording troubleshooting cases
- Storing diagnostic and automation scripts
- Connecting related information through tags and internal links

The goal is to transform technical notes into a searchable and reusable **second brain**.

---

## 📂 Vault Structure

- `Attachments` — Images and supporting files
- `Commands`
  - `Linux` — Linux commands and utilities
  - `Windows` — Windows and PowerShell commands
- `Concepts` — Networking, security, and system concepts
- `Guides` — Reusable technical procedures
- `Index` — Navigation pages grouped by category
- `Scripts` — Diagnostic and automation scripts
- `Templates` — Templates for creating consistent notes
- `Troubleshooting` — Problems, investigations, root causes, and resolutions

---

## 🚀 Getting Started

Clone the repository:

```bash
git clone https://github.com/emanueleorlietti/network-security-demo.git
```

Open Obsidian and select:

```text
Open folder as vault
```

Then choose the cloned `network-security-demo` folder.

---

## 🔌 Community Plugins

The vault uses the following Obsidian community plugins:

- **Dataview** — Creates dynamic indexes and views
- **QuickAdd** — Creates notes using predefined actions
- **Templater** — Applies templates and dynamic properties

If the plugins are not already installed:

1. Open **Settings**
2. Go to **Community plugins**
3. Disable Restricted Mode
4. Install and enable the required plugins

---

## 🟣 Basic Git Commands

1. `git status` — Shows modified, staged, and untracked files
2. `git add .` — Stages all modified and new files
3. `git commit -m "YYYYMMDD-XX"` — Creates a snapshot with a descriptive message
4. `git push` — Uploads local commits to the remote repository
5. `git pull` — Downloads and integrates remote changes

---

## 🔵 Useful Obsidian Shortcuts

1. `CTRL + E` — Toggle Edit and Preview mode
2. `CTRL + G` — Open Graph View
3. `CTRL/CMD + O` — Search and open notes
4. `CTRL/CMD + P` — Open the Command Palette
5. `CTRL + R` — Run QuickAdd

QuickAdd can be used to create:

- Commands
- Concepts
- Guides
- Scripts
- Troubleshooting notes

---

## 🏷️ Tagging Best Practices

To keep the vault clean, searchable, and consistent, follow these rules.

### Format Rules

- Use **lowercase only**
  - `#networking` ✅
  - `#Networking` ❌
- Use the **singular form**
  - `#command` ✅
  - `#commands` ❌
- Do not use spaces
  - `#network-security` ✅
  - `#network security` ❌
- Use hyphens for multi-word tags
- Avoid abbreviations unless they are widely recognized
  - `#dns`, `#vpn`, `#nat`, and `#tcp` are acceptable
- Reuse existing tags instead of creating similar alternatives

### Examples

| Good Tag | Bad Tag | Reason |
|---|---|---|
| `#networking` | `#Networking` | Uppercase creates inconsistencies |
| `#command` | `#commands` | Singular form is preferred |
| `#network-security` | `#network security` | Tags cannot contain spaces |
| `#troubleshooting` | `#troubleshoot` | Use one consistent form |

---

## 🛠️ Managing Tags

- Use the **Tag Pane** in the Obsidian sidebar to review existing tags
- Use **Rename tag** to correct inconsistent tags
- Remember that tags are case-sensitive
- Use tags to connect notes across different folders

For example, `#dns` can appear in:

- Commands such as `dig` or `Resolve-DnsName`
- Concepts such as DNS resolution
- Guides about DNS analysis
- Troubleshooting cases involving name resolution failures

Folders define where a note belongs. Tags describe what the note is about.

---

## 🧱 Code Block Standards

Use the appropriate code block language to improve readability.

### Linux and Shell Commands

```bash
ping -c 4 8.8.8.8
dig example.com
tcpdump -ni any port 53
```

### Windows and PowerShell Commands

```powershell
Test-NetConnection example.com -Port 443
Resolve-DnsName example.com
Get-NetTCPConnection
```

### Plain Output

```text
PING 8.8.8.8: 56 data bytes
64 bytes from 8.8.8.8: time=15 ms
```

Use inline code for commands, parameters, file paths, ports, and short output values.

Examples:

- Run `ping` to test basic reachability
- DNS commonly uses port `53`
- Linux logs are commonly stored under `/var/log/`
- Windows system files are stored under `C:\Windows\System32\`

---

## 🔗 Internal Linking

Use Obsidian links to connect related information:

```text
[[DNS Resolution]]
[[dig]]
[[DNS Resolution Failure]]
```

A troubleshooting note should link to the commands and concepts used during the investigation.

This creates connections between:

- Theory
- Diagnostic tools
- Procedures
- Real troubleshooting scenarios

---

## 🧠 Local Graph Tip

If linked notes do not show their names in the Local Graph:

- Zoom in using the mouse wheel or trackpad
- Confirm that you opened the **Local Graph**, not the global Graph View

The labels may be hidden when the graph is zoomed out.

---

## ⚠️ Disclaimer

This repository is intended for educational and documentation purposes.

Always review commands and procedures before using them in production environments. Examples may require adaptation based on the operating system, permissions, network topology, and security policies.

---

## 🤝 Contributions and Feedback

Suggestions, corrections, and improvements are welcome.

You can open a GitHub Issue or submit a Pull Request if you want to contribute to the project.