# SHADOWINT — All-in-One OSINT Framework

![Status](https://img.shields.io/badge/Status-Local%20Dashboard%20Running-success) ![Platform](https://img.shields.io/badge/Platform-iPhone%20%7C%20iSH-blue) ![Python](https://img.shields.io/badge/Backend-Python%20%7C%20Flask-informational) ![Version](https://img.shields.io/badge/Version-1.0-lightgrey)

> A local-first, mobile-friendly open-source intelligence (OSINT) investigation dashboard built and deployed on an iPhone using iSH Linux, Python, Flask, and SQLite.

## 1. Executive summary

SHADOWINT is a mobile OSINT framework for organizing passive, public-source cybersecurity investigations in one web interface. The project demonstrates how a lightweight Linux environment on an iPhone can host a Python web application with multiple research modules, case storage, and report export features.

**Confirmed milestone:** The Flask dashboard was successfully launched and accessed using **`http://127.0.0.1:5050`** after troubleshooting a port conflict on the original port 5000.

**Verification boundary:** Installation and local dashboard accessibility were confirmed in the build session. Successful end-to-end execution of all OSINT modules, SQLite case persistence, report downloads, and GitHub publication were **not yet confirmed**.

## 2. Project objectives

- Run a practical cybersecurity investigation application on an iPhone without jailbreaking.
- Provide a centralized interface for passive domain, IP, email, username, URL, and threat reputation research.
- Store investigation records locally using SQLite.
- Support structured exports for analyst notes and portfolio documentation.
- Practice Linux command-line operations, Python application deployment, debugging, and Git version control.

## 3. Technology stack

| Component | Technology | Purpose |
|---|---|---|
| Device | iPhone | Mobile development and access |
| Linux environment | iSH / Alpine Linux | Terminal and local application runtime |
| Backend | Python 3, Flask | Web server and investigation endpoints |
| Frontend | HTML, CSS, JavaScript | Mobile-responsive dashboard |
| Storage | SQLite | Local case history |
| Network tools | curl, DNS and public-source APIs | Connectivity checks and OSINT lookups |
| Source control | Git / GitHub | Version control and publication |

## 4. Architecture

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

## 5. Source-code structure

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

## 6. Build and installation process

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

### Phase C — Launch dashboard

```sh
cd /root/shadowint
python3 app.py
```

The initial launch encountered a port conflict. See the troubleshooting record below.

### Phase D — Resolve port conflict and launch

After reviewing the Flask log, updated the local development port from 5000 to 5050:

```sh
cd /root/shadowint
cp app.py app.py.backup
sed -i 's/port=5000/port=5050/' app.py
grep 'app.run' app.py
python3 app.py
```

**Confirmed result:** The dashboard worked on `http://127.0.0.1:5050`.

> Note: The original distributed ZIP/README uses port 5000. The deployed iSH copy was changed to 5050. Keep these consistent when updating the GitHub repository.

## 7. Troubleshooting case study: Flask port collision

### Symptom

Safari could not connect to the original dashboard address:

```text
http://127.0.0.1:5000
```

A local curl check returned:

```text
curl: (7) Failed to connect to 127.0.0.1 port 5000
Connection refused
```

### Diagnostic procedure

Started the application with stdout/stderr redirected to a log and inspected it:

```sh
python3 app.py > /tmp/shadowint.log 2>&1 &
cat /tmp/shadowint.log
```

### Root-cause evidence

The Python traceback ended with:

```text
OSError: [Errno 98] Address in use
```

This indicated the attempted Flask listener could not bind to port 5000 because the address/port was already in use. The precise process occupying that port was not identified in the recorded session.

### Remediation

Backed up `app.py`, changed the Flask port to 5050, restarted the application, and opened the new local URL.

### Outcome and lesson learned

**Outcome:** SHADOWINT became accessible on port 5050.

**Lesson:** Check application logs and port bindings before assuming a browser/networking issue. An `Address in use` error is a server bind failure; switching ports is a practical workaround, while identifying and stopping the conflicting process is an alternative.

## 8. Implemented module capabilities (source-code scope)

| Module | Intended functionality | Suggested test input |
|---|---|---|
| Domain Intelligence | DNS, RDAP, certificate transparency | `example.com` |
| IP Intelligence | Public network information, reverse DNS, approximate location | `8.8.8.8` |
| Username OSINT | Public profile research links; no identity verification | `octocat` |
| Email Security | MX, SPF, DMARC checks | `security@example.com` |
| URL Investigator | Passive URL parsing and public DNS resolution; does not fetch target page | `https://example.com` |
| Threat Reputation | Optional AbuseIPDB reputation lookup | `8.8.8.8` |

**Important:** These features are present in the project source, but the build record does not establish that every external lookup succeeded on the iPhone.

## 9. Test plan and evidence checklist

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
3. `03-port-5000-error.png` — Curl connection refused.
4. `04-address-in-use-traceback.png` — Flask log showing `OSError: [Errno 98] Address in use`.
5. `05-port-5050-server.png` — Successful Flask startup on port 5050.
6. `06-shadowint-dashboard.png` — Dashboard open in Safari.
7. `07-domain-investigation.png` — Completed domain lookup (pending verification).
8. `08-ip-investigation.png` — Completed IP lookup (pending verification).
9. `09-saved-cases.png` — SQLite case history (pending verification).
10. `10-exported-report.png` — Downloaded report (pending verification).

Add images to this README once available, for example:

```md
![SHADOWINT dashboard running on iPhone](screenshots/06-shadowint-dashboard.png)
```

Only publish screenshots of completed tests; redact tokens, API keys, personal data, and sensitive targets.

## 10. Security considerations and limitations

- Conduct only authorized investigations using public information.
- Do not treat username matches as proof of identity.
- Do not treat IP geolocation as precise location evidence.
- External sources may rate-limit requests or return incomplete results.
- The URL module performs passive analysis; it does not crawl or visit the submitted URL.
- Keep API keys in environment variables, not committed source files.
- Keep local investigation databases and private reports out of public repositories.
- Flask's built-in development server is appropriate for local testing, not production internet exposure.
- iOS may suspend iSH when backgrounded, affecting local service availability.

## 11. GitHub publication checklist

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

Create the empty GitHub repository first. Authenticate with an appropriate personal access token or SSH key. Update the original README to mention port 5050 if that is the intended default for the published version.

## 12. Skills demonstrated

- Linux package management and terminal operations
- Python and Flask application deployment
- Loopback networking and HTTP API testing
- Reading stack traces and application logs
- Diagnosing TCP port conflicts
- Passive OSINT workflow design
- SQLite-based investigation storage
- Cybersecurity technical documentation
- Git/GitHub version control workflow

## 13. Project status

| Milestone | Status |
|---|---|
| iSH dependencies installed | Confirmed |
| Source files installed | Confirmed |
| Dashboard accessible on port 5050 | Confirmed |
| All six OSINT modules tested successfully | Pending confirmation |
| SQLite case persistence verified | Pending confirmation |
| JSON/Markdown exports verified | Pending confirmation |
| GitHub repository published | Pending confirmation |

## 14. Resume-ready description

**SHADOWINT — Mobile OSINT Framework | Personal Cybersecurity Project**

- Deployed a Python/Flask-based OSINT investigation dashboard within Alpine Linux on an iPhone using iSH.
- Integrated source-code modules for passive domain, IP, email, username, URL, and threat reputation research, with SQLite case management and report export functionality.
- Diagnosed a Flask server startup failure using curl and Python traceback logs, identifying an address-in-use conflict and restoring dashboard access by migrating the local service to port 5050.

---

**Project version:** 1.0  
**Deployment environment:** iPhone / iSH / Alpine Linux  
**Documented status:** Dashboard running; module-by-module validation in progress.
