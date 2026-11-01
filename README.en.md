<div align="center">

# CTF.Term

**Dynamic cheatsheets for cybersecurity labs and CTFs.**

A static web application for organizing commands, filling in lab-specific variables, and copying commands for Kali Linux — without executing commands against the target.

[Português](README.md) · **English**

</div>

---

## About the project

**CTF.Term** is a browser-based application that turns terminal commands and technical notes into a reusable cheatsheet. Instead of editing IP addresses, domains, usernames, and paths in every command, you enter these values once in the **Variables** section and use placeholders such as `{{ip}}` and `{{domain}}` in your templates.

> **Responsible use:** the application only assembles and copies text. It does not connect to the target, verify permissions, or execute commands. Use these examples only in environments where you are authorized to do so.

## Features

- **62 built-in variables**, organized into groups: target/network, web, authentication, wordlists, Active Directory/Windows, and tunnels/pivoting.
- **Dynamic variables** using the `{{variable_name}}` syntax, with previews of derived values.
- **Nmap as the only initial command category**, with three editable examples.
- **Custom categories:** create, edit, choose symbols for, and delete categories from the sidebar.
- **Custom commands:** create, edit, delete, search, and copy commands, including multiline commands.
- **Syntax highlighting** for tools, options, strings, placeholders, and Bash comments beginning with `#`.
- **Search** within the current command category and filtering of variables.
- **Automatic local storage**, JSON workspace import/export, and Markdown cheatsheet export.
- **Responsive interface**, with no backend and no external JavaScript dependencies.

## Getting started

### Option A — Offline access (cloning the repository)

Clone the project once while you are connected to the internet:

```bash
git clone https://github.com/e-pke/ctf-term.git
cd ctf-term
```

After cloning, you can use CTF.Term **locally, without an internet connection**. On Kali Linux, run the following command from the project directory:

```bash
python3 -m http.server 8000
```

Open **http://localhost:8000** in your browser. The server runs locally and does not require Node.js, a database, or `npm install`.

### Option B — Online access (GitHub Pages)

Access CTF.Term directly in your browser, without downloading or installing anything:

```text
https://e-pke.github.io/ctf-term/
```

The online version retains the variable, category, command, and import/export features. Customizations are saved in your browser's `localStorage`, **not in the GitHub repository**, and are not synchronized across devices.

## How to use

### 1. Configure your variables

Open **Variables** in the sidebar and fill in the fields relevant to your lab. The example values shown in the input fields are only *placeholders*; they are not filled in automatically.

| Field | Example | Usage in a command |
| --- | --- | --- |
| Target IP | `10.10.10.10` | `{{ip}}` |
| Domain | `domain.htb` | `{{domain}}` |
| TCP ports | `22,80,443` | `{{ports_tcp}}` |
| Username | `john` | `{{username}}` |
| Directory wordlist | `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` | `{{dir_wordlist}}` |
| HTTP port | `8080` | `{{http_port}}` |

You can also create your own variables using **+ New variable**. Identifiers may contain lowercase letters, numbers, and `_`, and must begin with a letter.

### 2. Use an existing command

The initial **Nmap** category contains three commands. Here is an example template:

```bash
nmap -sC -sV -p {{ports_tcp}} {{ip}}
```

With `ports_tcp = 22,80,443` and `ip = 10.10.10.10`, the card will display:

```bash
nmap -sC -sV -p 22,80,443 10.10.10.10
```

Click **Copy** to copy the resolved command to your clipboard. If required variables are missing, the application will show their names and disable the Copy button on that card.

### 3. Create categories

Use **+ New category** in the sidebar to enter a name, identifier, symbol, title, and description. The new category appears in the menu immediately. Then select it and click **+ New command**.

### 4. Create or edit commands

Each command has a **category**, **summary/title**, **optional description**, and **template**. Include variables directly in the text:

```bash
# Directory discovery
ffuf -u 'http://{{domain}}{{http_port}}/FUZZ' -w '{{dir_wordlist}}' -ac
```

You decide where to put `http://`, `https://`, single quotes, or double quotes. The application performs **literal substitution**, without automatically adding quotes or escaping special shell characters.

For multiline commands, comments beginning with `#` are visually highlighted. The `$` symbol is only decorative for single-line commands and **is not copied**.

### 5. Search

The search field below the category description filters commands by title, description, and template. In the **Variables** section, the search filters fields by name, identifier, group, and hint. When viewing commands, you can also press `Ctrl+K` (or `⌘+K`) or `/` to focus the search field, as long as you are not typing in another input.

## Automatic variables and special rules

| Variable | Behavior |
| --- | --- |
| `{{dc_ip}}` | If the field is empty, uses `{{ip}}`. |
| `{{dns_server}}` | If the field is empty, uses `{{domain}}`. For queries such as `dig @...`, that hostname must refer to an accessible DNS server. |
| `{{http_port}}` | `8080` becomes `:8080`; an empty value inserts nothing. It does not add a protocol. |
| `{{base_dn}}` | Derived from `domain.htb` as `DC=domain,DC=htb`. |
| `{{ldap_bind}}` | Derived from the username and domain, such as `john@domain.htb`. |

LDAP example:

```bash
ldapsearch -x -H 'ldap://{{dc_ip}}' -D '{{ldap_bind}}' -W -b '{{base_dn}}'
```

**Notes:** derived values depend on the fields you fill in; the `http_port` field accepts valid ports from `1` to `65535`. If you use SSTI/Jinja2 templates, remember that expressions such as `{{config}}` may be interpreted as CTF.Term placeholders; an expression with spaces, such as `{{ config }}`, does not match the current substitution pattern.

## Privacy and storage

CTF.Term uses the browser's **`localStorage`** with the key `ctf-term:workspace:v6`. Changes made in the interface normally persist after a page reload, but they are not synchronized with GitHub, other computers, or other browsers.

The following built-in fields are treated as sensitive and remain **only in the tab's memory**, without persisting or exporting their values:

`password`, `ntlm_hash`, `fake_password`, `jwt_token`, `krbtgt_hash`, `service_hash`, and `snmp_community`.

Controls are available to show or hide sensitive values. To copy a command that depends on one of these populated fields, you must allow its value to be displayed in the interface.

> **Important:** custom variables **do not automatically receive** this protection. Do not store real secrets in them. **Passwords, tokens, or hashes written literally in a command's text will also be saved and exported**. Additionally, copied or executed commands may expose data through the clipboard, shell history, or tool logs.

If you clear the site's data or change its origin (for example, from `file://` to `http://localhost:8000`, or to the GitHub Pages domain), the storage may be different. Export your data before switching addresses.

## Import, export, and backups

| Feature | What it does |
| --- | --- |
| **Export JSON** | Downloads the workspace with non-sensitive variables, categories, and commands. |
| **Import JSON** | Replaces **the entire current workspace** with the contents of the file after confirmation. |
| **Cheatsheet .md** | Exports commands from the categories as Markdown, with non-sensitive values resolved. |
| **Reset all** | Restores the 62 empty variables and the six default Nmap commands, removing local customizations. |

The importer accepts the official **`ctf-term.workspace` / schema `6`** format. A simplified structure looks like this:

```json
{
  "format": "ctf-term.workspace",
  "schema": 6,
  "workspace": {
    "variables": {},
    "customVariables": [],
    "categories": [],
    "commands": []
  }
}
```

- The JSON file must be no larger than **1 MB** to be imported through the interface.
- The application supports up to **60 custom categories**, **80 custom variables**, and **300 commands** per workspace.
- **Third-party Markdown or JSON files are not automatically importable** just because they contain commands: a cheatsheet JSON must be converted to the schema above and use valid categories/IDs. Importing still **replaces** the workspace; it does not merge data.

**Back up your data before importing another workspace, resetting defaults, or clearing your browser's site data.**

## Security and limitations

- Commands are **educational templates**, not scan results produced by the application.
- CTF.Term **does not execute commands**, does not perform enumeration automatically, and does not check whether tools are installed.
- Literal substitution does not protect against shell expansion. Review quotes, special characters, and entered values before executing commands.
- Some external examples may be payloads, inputs for specialized consoles, or configuration snippets rather than ready-to-run Bash commands.
- Tool availability, options, and syntax depend on the versions installed in Kali Linux.

## Credits and attribution

CTF.Term was developed as a cheatsheet organization tool for studying in authorized labs.

- **Design:** interface based on `#212121`, with highlights inspired by the [Nord Wave](https://cmuxthemes.com/themes/nord-wave/) theme.
- **Font:** [JetBrains Mono](https://www.jetbrains.com/lp/mono/).
- **Optional cheatsheets:** customized cheatsheet based on [momenbasel/htb-writeups](https://github.com/momenbasel/htb-writeups).

The third-party collection **is not included in the application's default categories**.

---

<div align="center">

**CTF.Term — configure once, reuse your commands in every lab.**

[Leia esta documentação em português](README.md)

</div>
