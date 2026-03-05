# ReconX

> Advanced Bug Bounty Reconnaissance Tool

A feature-rich desktop GUI application for web reconnaissance, built in Python. ReconX bundles nine specialized modules — from subdomain enumeration and port scanning to Wayback Machine URL harvesting and vulnerability checks — into a single cyberpunk-themed interface designed for bug bounty hunters and penetration testers.

> **For authorized use only.** Only test targets you own or have explicit written permission to assess.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Screenshots](#screenshots)
- [Installation](#installation)
- [Usage](#usage)
- [Modules](#modules)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Changelog](#changelog)
- [References](#references)
- [License](#license)

---

## Overview

ReconX gives bug bounty hunters a unified dashboard to run the most common reconnaissance tasks without switching between tools. Enter a target domain, pick a module, and results stream live into a color-coded console and a structured results table — all exportable to CSV or JSON.

**Why ReconX:**
- No CLI flags to memorize — everything is point-and-click
- All nine modules share the same target input and results table
- Full Recon mode chains all modules automatically, one after another
- Live statistics panel tracks subdomains, open ports, directories, and vulnerabilities found per session
- Customizable threading, timeouts, port ranges, and wordlist paths via the Settings panel

---

## Features

| Module | What It Does |
|--------|-------------|
| **Subdomain Enumeration** | Wordlist bruteforce + crt.sh certificate transparency logs + DNS zone transfer |
| **Port Scanner** | TCP connect scan with banner grabbing across configurable port ranges |
| **Directory Bruteforce** | HTTP path discovery using a bundled wordlist |
| **Technology Detection** | Fingerprints CMS, frameworks, CDNs, and server software from headers, HTML, and cookies |
| **DNS Enumeration** | Resolves A, AAAA, MX, NS, TXT, SOA, CNAME, PTR, SRV, CAA records + DNSSEC check |
| **Header Analysis** | Audits HTTP security headers against best-practice recommendations |
| **Wayback Machine** | Harvests archived URLs, sensitive file paths, JS files, and API endpoints |
| **WHOIS Lookup** | Domain registration, registrar, nameservers, and contact info |
| **Vulnerability Scanner** | SSL/TLS checks, security misconfigurations, CORS, open redirects, and more |
| **Full Recon** | Runs all modules sequentially on the target |

**GUI Highlights:**
- Animated color-cycling ASCII banner
- Custom buttons with neon glow hover effects
- Live output console with per-message color coding (info / success / warning / error)
- Results table with right-click context menu: Copy, Open in Browser, Export
- Import a target list from a `.txt` file for multi-target sessions
- Export results to CSV or JSON at any time
- Live statistics counters: subdomains, open ports, directories, vulnerabilities

---

## Screenshots

> _Add screenshots of the application here._

---

## Installation

### Requirements

- Python 3.8 or higher
- Tkinter (included with most Python distributions)

### Steps

```bash
# Clone the repository
git clone https://github.com/rohit-1006/ReconX.git
cd ReconX

# Create and activate a virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows

# Install dependencies
pip install -r requirements.txt

# Launch the application
python3 reconx.py
```

### Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `requests` | >=2.28.0 | HTTP requests |
| `dnspython` | >=2.3.0 | DNS resolution and enumeration |
| `python-whois` | >=0.8.0 | WHOIS lookups |
| `urllib3` | >=1.26.0 | URL handling |
| `beautifulsoup4` | >=4.11.0 | HTML parsing |
| `lxml` | >=4.9.0 | XML/HTML parser backend |

### Tkinter Not Found?

```bash
# Debian / Ubuntu
sudo apt install python3-tk

# Fedora / RHEL
sudo dnf install python3-tkinter

# Arch Linux
sudo pacman -S tk
```

On macOS, Tkinter ships with the official Python installer from python.org. If installed via Homebrew, run `brew install python-tk`.

---

## Usage

### Starting the Application

```bash
python3 reconx.py
```

### Basic Workflow

**1. Enter a target**
Type a domain name (e.g. `example.com`) into the target field at the top of the window.

**2. Click a module card**
Each module appears as a clickable card in the left panel. Click one to start that scan immediately. Results stream into the console in real time and are added to the Results table.

**3. Review and export**
Switch to the **Results** tab to see all findings in a structured table. Right-click any row to copy a value, open it in the browser, or export the full table to CSV or JSON.

**4. Run everything at once**
Click the **Full Recon** card to chain all modules sequentially. The application runs each module in order and aggregates all findings into the same results table.

**5. Stop a scan**
Click the **STOP** button at any time to halt the running scan.

### Importing Multiple Targets

Go to **File → Import Targets** and select a `.txt` file with one domain per line.

### Exporting Results

- Console output: click **EXPORT** in the console header to save as `.txt`
- Results table: right-click → Export All, or go to **File → Export Results** for CSV or JSON

---

## Modules

### Subdomain Enumeration

Combines three techniques:

1. **Wordlist bruteforce** — DNS-resolves prefixes from `wordlists/subdomains.txt` (107 entries) concurrently using a thread pool
2. **Certificate Transparency** — Queries `crt.sh` for issued certificates matching `*.domain`, extracting all subject alternative names
3. **DNS Zone Transfer** — Attempts AXFR against each nameserver; records all names if the server allows it

### Port Scanner

TCP connect scan with optional banner grabbing. Resolves the target hostname before scanning, distributes port checks across a configurable thread pool, and maps 22 well-known ports to service names:

FTP (21), SSH (22), Telnet (23), SMTP (25), DNS (53), HTTP (80), POP3 (110), IMAP (143), HTTPS (443), SMB (445), MSSQL (1433), MySQL (3306), RDP (3389), PostgreSQL (5432), VNC (5900), Redis (6379), HTTP-Proxy (8080), HTTPS-Alt (8443), MongoDB (27017), Elasticsearch (9200), and more.

Accepts a port range (`1-1000`), comma-separated list (`80,443,8080`), or single port.

### Directory Bruteforce

HTTP path discovery using `wordlists/directories.txt` (106 entries). Reports status code and response size for each found path. Color-codes results: green for 200, yellow for redirects and other codes.

### Technology Detection

Fingerprints the target across three signal types — HTTP response headers, HTML body patterns, and cookies — for the following technologies:

**CMS:** WordPress, Drupal, Joomla, Magento, Shopify
**Frameworks:** React, Angular, Vue.js
**Additional signatures** are checked for server software, CDNs, and analytics platforms.

### DNS Enumeration

Resolves all standard record types for the target domain: `A`, `AAAA`, `MX`, `NS`, `TXT`, `SOA`, `CNAME`, `PTR`, `SRV`, `CAA`. Also resolves nameserver IPs, checks for DNSSEC (`DNSKEY`), and performs reverse PTR lookups on discovered A records.

### Header Analysis

Fetches the target's HTTP headers and audits against 9 security headers with recommendations:

| Header | Purpose |
|--------|---------|
| `Strict-Transport-Security` | Enforces HTTPS |
| `Content-Security-Policy` | Prevents XSS |
| `X-Frame-Options` | Prevents clickjacking |
| `X-Content-Type-Options` | Prevents MIME sniffing |
| `Referrer-Policy` | Controls referrer leakage |
| `Permissions-Policy` | Restricts browser features |
| `Cross-Origin-Opener-Policy` | Isolates browsing context |
| `Cross-Origin-Resource-Policy` | Controls resource loading |
| `Cross-Origin-Embedder-Policy` | Controls embedding |

Missing required headers are flagged as **missing** (red), present headers are verified for recommended values.

### Wayback Machine

Queries the Internet Archive CDX API for all archived URLs under `*.domain/*`. Results are deduplicated and streamed live. Capped at 100 URLs in the GUI to prevent overflow. The underlying module also supports filtering for:

- **Interesting/sensitive paths** — `.php`, `.env`, `.git`, `.sql`, `.bak`, `admin`, `login`, `backup`, `debug`, `.config`, `phpinfo`, `password`, `secret`
- **JavaScript files** — any URL ending in `.js` or containing `/js/`
- **API endpoints** — paths containing `api/`, `/v1/`, `/v2/`, `/graphql`, `/rest/`

### WHOIS Lookup

Retrieves domain registration data including registrar, creation and expiry dates, nameservers, and registrant contact information.

### Vulnerability Scanner

Runs seven passive and semi-active checks:

| Check | Severity |
|-------|---------|
| Weak TLS version (TLS 1.0 / 1.1 detected) | High |
| SSL certificate expiring within 30 days | Medium |
| SSL certificate expired | High |
| Directory listing enabled | Medium |
| Sensitive file exposure (`.env`, `.git`, `backup`, etc.) | Variable |
| CORS misconfiguration | Medium–High |
| Open redirect | Medium |
| Clickjacking (missing `X-Frame-Options`) | Medium |
| Information disclosure (server banners, error messages) | Low |

---

## Configuration

Open **Tools → Settings** to adjust:

| Setting | Default | Description |
|---------|---------|-------------|
| Threads | 10 | Concurrent workers for bruteforce modules |
| Timeout | 5s | Per-request timeout |
| Rate Limit | 100 | Max requests per second |
| Port Range | `1-1000` | Range for port scanner |
| Scan Type | TCP | Protocol for port scanning |
| Subdomain Wordlist | `wordlists/subdomains.txt` | Path to subdomain wordlist |
| Directory Wordlist | `wordlists/directories.txt` | Path to directory wordlist |

Custom wordlists can be dropped into the `wordlists/` directory and pointed to via Settings. Any text file with one entry per line is supported.

---

## Project Structure

```
ReconX/
├── reconx.py                  # Main application and GUI
│   ├── CyberTheme             # Color palette and font configuration
│   ├── AnimatedBanner         # Animated ASCII logo (color-cycling)
│   ├── ModernButton           # Custom canvas button with glow effect
│   ├── StatusBar              # Progress bar and live clock
│   ├── OutputConsole          # Color-coded scrollable log with export
│   ├── TargetInput            # Target domain input widget
│   ├── ModuleCard             # Clickable module tile
│   ├── ResultsTable           # Treeview results table (CSV/JSON export)
│   ├── SettingsPanel          # Configuration dialog
│   └── ReconX                 # Main application window
│
├── modules/
│   ├── subdomain.py           # Subdomain enumeration (bruteforce + crt.sh + zone transfer)
│   ├── portscan.py            # TCP port scanner with banner grabbing
│   ├── dirbrute.py            # Directory bruteforcer
│   ├── techdetect.py          # Technology fingerprinting
│   ├── dnsenum.py             # DNS record enumeration
│   ├── headers.py             # HTTP security header analysis
│   ├── wayback.py             # Wayback Machine URL harvesting
│   ├── whois_lookup.py        # WHOIS data retrieval
│   └── vulnscan.py            # Basic vulnerability checks
│
├── wordlists/
│   ├── subdomains.txt         # 107 common subdomain prefixes
│   └── directories.txt        # 106 common web paths
│
├── requirements.txt
└── README.md
```

---

## Troubleshooting

**Application does not open**
Verify Tkinter is available:
```bash
python3 -c "import tkinter; tkinter.Tk().mainloop()"
```
If this fails, install Tkinter for your OS (see [Installation](#installation)).

**`ModuleNotFoundError` for `dns.resolver` or `whois`**
Ensure your virtual environment is active and dependencies are installed:
```bash
source venv/bin/activate
pip install -r requirements.txt
```

**Subdomain enumeration returns no results**
- Confirm the target domain is reachable and resolves
- Try increasing the thread count or timeout in Settings
- The crt.sh query requires internet access; check your connection

**Port scanner is slow**
Increase the thread count in Settings (default is 10) or narrow the port range. For a quick check, use common ports only: `21,22,80,443,3306,8080,8443`.

**Wayback returns no URLs**
The Wayback CDX API has rate limits. Wait a moment and try again. Very new domains may have no archived records.

**`A scan is already running` message**
Only one module can run at a time. Wait for the current scan to complete or click **STOP** to cancel it before starting a new one.

---

## Contributing

Contributions are welcome. To get started:

1. Fork the repository and create a feature branch: `git checkout -b feature/your-feature`
2. Keep changes focused — one feature, fix, or module per pull request
3. Test your changes against a live or local target before submitting
4. Open a pull request against `main` with a clear description of the change

For new modules, open an issue first to discuss the approach.

### Reporting Bugs

Open a GitHub issue with:
- Python version and OS
- Steps to reproduce the issue
- Expected vs actual behavior
- Any error output from the terminal

### Roadmap

- [ ] Custom HTTP headers and cookie injection for authenticated recon
- [ ] Larger bundled wordlists (SecLists integration)
- [ ] Screenshot capture for discovered URLs
- [ ] Multi-target batch mode
- [ ] Session save and restore (persist results between runs)
- [ ] Proxy support (route traffic through Burp Suite)
- [ ] API key integration for Shodan, VirusTotal, SecurityTrails

---

## Changelog

### v2.0.0

- Full GUI rewrite with cyberpunk theme and animated banner
- Custom `ModernButton` widget with neon glow hover effect
- Live statistics panel (subdomains, ports, directories, vulnerabilities)
- Results table with CSV and JSON export
- Right-click context menu in results (Copy, Open in Browser, Export)
- Settings panel for threading, timeouts, and wordlist paths
- Import targets from file
- Full Recon mode chaining all modules

### v1.0.0

- Initial release with CLI interface
- Subdomain enumeration, port scanning, and directory bruteforce

---

## References

| Resource | URL |
|----------|-----|
| OWASP Testing Guide | https://owasp.org/www-project-web-security-testing-guide/ |
| Bug Bounty Methodology | https://github.com/jhaddix/tbhm |
| crt.sh Certificate Search | https://crt.sh |
| Wayback CDX API | https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server |
| SecLists Wordlists | https://github.com/danielmiessler/SecLists |
| HackTricks Recon | https://book.hacktricks.xyz/pentesting-web/web-vulnerabilities-methodology |

---

## License

This project is licensed under the [MIT License](LICENSE).
