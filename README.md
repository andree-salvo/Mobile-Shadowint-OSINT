# SHADOWINT — All-in-One OSINT Framework

![Status](https://img.shields.io/badge/Status-Local%20Dashboard%20Running-success) ![Platform](https://img.shields.io/badge/Platform-iPhone%20%7C%20iSH-blue) ![Python](https://img.shields.io/badge/Backend-Python%20%7C%20Flask-informational) ![Version](https://img.shields.io/badge/Version-1.0-lightgrey)

> A local-first, mobile-friendly open-source intelligence (OSINT) investigation dashboard built and deployed on an iPhone using iSH Linux, Python, Flask, and SQLite.

## 0. Executive summary

SHADOWINT is a mobile OSINT framework for organizing passive, public-source cybersecurity investigations in one web interface. The project demonstrates how a lightweight Linux environment on an iPhone can host a Python web application with multiple research modules, case storage, and report export features.

**Confirmed milestone:** The Flask dashboard was successfully launched and accessed using **`http://127.0.0.1:5050`**.

**Verification boundary:** Installation and local dashboard accessibility were confirmed in the build session. Successful end-to-end execution of all OSINT modules, SQLite case persistence, report downloads, and GitHub publication were **not yet confirmed**.

## 1. Project objectives

- Run a practical cybersecurity investigation application on an iPhone without jailbreaking.
- Provide a centralized interface for passive domain, IP, email, username, URL, and threat reputation research.
- Store investigation records locally using SQLite.
- Support structured exports for analyst notes and portfolio documentation.
- Practice Linux command-line operations, Python application deployment, application configuration, and Git version control.

## 2. Technology stack

| Component | Technology | Purpose |
|---|---|---|
| Device | iPhone | Mobile development and access |
| Linux environment | iSH / Alpine Linux | Terminal and local application runtime |
| Backend | Python 3, Flask | Web server and investigation endpoints |
| Frontend | HTML, CSS, JavaScript | Mobile-responsive dashboard |
| Storage | SQLite | Local case history |
| Network tools | curl, DNS and public-source APIs | Connectivity checks and OSINT lookups |
| Source control | Git / GitHub | Version control and publication |

## 3. Architecture

```text
Safari on iPhone
       |
       | HTTP http://127.0.0.1:5050
       v
iSH / Alpine Linux
       |
       v
Python Flask application (app.py)
       |-------------------------------|
       v                               v
Passive OSINT modules          SQLite case database
       |                               |
       v                               v
Public DNS / RDAP / APIs       JSON / Markdown exports
```

The Flask server is bound to loopback (`127.0.0.1`) rather than exposed to the local network.

## 4. Source-code structure

```text
shadowint/
├── app.py                 # Flask app, module handlers, API routes
├── requirements.txt       # Python dependencies
├── README.md              # Original project readme
├── .gitignore             # Excluded local files
├── templates/
│   └── index.html         # Dashboard page
└── static/
    ├── style.css          # Interface styling
    └── app.js             # Browser interactions
```

SQLite creates `shadowint.db` when the application initializes. The database should not be committed to GitHub.

## 5. Build and installation process

### Phase A — Prepare iSH

Installed the required Alpine packages and verified the Python/Flask runtime:

```sh
apk update && apk upgrade
apk add python3 py3-pip py3-flask git curl unzip sqlite ca-certificates
python3 --version
python3 -c "import flask; print('SHADOWINT READY')"
```

**Observed result:** The environment printed `SHADOWINT READY` and installation Step 1 was completed.

### Phase B — Install project files

Transferred the SHADOWINT source ZIP through the iPhone Files app into the iSH `root` directory, then extracted and checked the project:

```sh
cd /root
ls -lh SHADOWINT-v1.0.zip
unzip SHADOWINT-v1.0.zip
cd /root/shadowint
ls -la
python3 -m py_compile app.py
```

**Observed result:** The project files were installed and Step 2 was completed.

### Phase C — Configure and launch dashboard

The deployed SHADOWINT instance uses port **5050**. Ensure the last line of `app.py` configures Flask with this port:

```python
app.run(host='127.0.0.1', port=5050, debug=False)
```

Launch the application:

```sh
cd /root/shadowint
python3 app.py
```

Open Safari at **http://127.0.0.1:5050**.

**Confirmed result:** The SHADOWINT dashboard loaded successfully on the iPhone, with the investigation console, module selector, and case history visible.

## 7. Implemented module capabilities (source-code scope)

| Module | Intended functionality | Suggested test input |
|---|---|---|
| Domain Intelligence | DNS, RDAP, certificate transparency | `example.com` |
| IP Intelligence | Public network information, reverse DNS, approximate location | `8.8.8.8` |
| Username OSINT | Public profile research links; no identity verification | `octocat` |
| Email Security | MX, SPF, DMARC checks | `security@example.com` |
| URL Investigator | Passive URL parsing and public DNS resolution; does not fetch target page | `https://example.com` |
| Threat Reputation | Optional AbuseIPDB reputation lookup | `8.8.8.8` |

**Important:** These features are present in the project source, but the build record does not establish that every external lookup succeeded on the iPhone.

## 8. Test plan and evidence checklist

Run these commands against the deployed port:

```sh
curl -v --max-time 10 http://127.0.0.1:5050/health

curl -X POST http://127.0.0.1:5050/api/investigate \
  -H 'Content-Type: application/json' \
  -d '{"module":"username","target":"octocat"}'

sqlite3 /root/shadowint/shadowint.db \
  'SELECT id, module, target, created FROM cases;'
```

Capture the following screenshots and save them under `screenshots/`:

1. `01-ish-environment.png` — Python/Flask verification and `SHADOWINT READY`.
2. `02-project-files.png` — Project directory structure (`ls -la`).
3. `03-port-5050-server.png` — Successful Flask startup on port 5050.
4. `04-shadowint-dashboard.png` — Dashboard open in Safari.
5. `05-domain-investigation.png` — Completed domain lookup (pending verification).
6. `06-ip-investigation.png` — Completed IP lookup (pending verification).
7. `07-saved-cases.png` — SQLite case history (pending verification).
8. `08-exported-report.png` — Downloaded report (pending verification).

Add images to this README once available, for example:

```md
![SHADOWINT dashboard running on iPhone](screenshots/06-shadowint-dashboard.png)
```

Only publish screenshots of completed tests; redact tokens, API keys, personal data, and sensitive targets.

## 9. Security considerations and limitations

- Conduct only authorized investigations using public information.
- Do not treat username matches as proof of identity.
- Do not treat IP geolocation as precise location evidence.
- External sources may rate-limit requests or return incomplete results.
- The URL module performs passive analysis; it does not crawl or visit the submitted URL.
- Keep API keys in environment variables, not committed source files.
- Keep local investigation databases and private reports out of public repositories.
- Flask's built-in development server is appropriate for local testing, not production internet exposure.
- iOS may suspend iSH when backgrounded, affecting local service availability.

## 10. GitHub publication checklist

Before publishing:

```sh
cd /root/shadowint
cat .gitignore
# Ensure shadowint.db, *.db, .env, and local backups are ignored.
git init
git status
git add .
git status
git commit -m 'Initial SHADOWINT OSINT framework'
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/SHADOWINT.git
git push -u origin main
```

Create the empty GitHub repository first. Authenticate with an appropriate personal access token or SSH key. Ensure the published `app.py` and README both specify the intended port (5050 for this deployment).

## 11. Skills demonstrated

- Linux package management and terminal operations
- Python and Flask application deployment
- Loopback networking and HTTP API testing
- Passive OSINT workflow design
- SQLite-based investigation storage
- Cybersecurity technical documentation
- Git/GitHub version control workflow

## 12. Project status

| Milestone | Status |
|---|---|
| iSH dependencies installed | Confirmed |
| Source files installed | Confirmed |
| Dashboard accessible on port 5050 | Confirmed |
| All six OSINT modules tested successfully | Pending confirmation |
| SQLite case persistence verified | Pending confirmation |
| JSON/Markdown exports verified | Pending confirmation |
| GitHub repository published | Pending confirmation |
