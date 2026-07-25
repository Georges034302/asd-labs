# Lab 01 - DevOps and Agentic AI Foundations

**Course:** Advanced Software Development with Agentic AI  
**Theme:** DevOps and Agentic AI Foundations  
**Primary IDE:** VS Code  (Optional IDE: AWS Kiro).  
**AI Agent Runtime:** Ollama  
**Recommended Open-Source Model:** Qwen2.5 0.5B  
**Duration**: 60 minutes

---

## 1. Overview

<details>
<summary>Goal</summary>

Build a local **Student Enrolment App** using Flask, HTMX, SQLite, Ollama, and an open-source Qwen model.

</details>

<details>
<summary>Agentic Workflow</summary>

```text
PLAN → ACT → OBSERVE → ADAPT
```

</details>

<details>
<summary> Expected Results</summary>

By the end of this lab, students should have:

- Flask app running locally
- SQLite database with 10 students
- Local Ollama AI agent running
- Qwen model call working from Flask
- Agentic loop script completed
- Endpoint checks completed
- NFR validation completed
- Evidence log completed
- Reflection completed

</details>

<details>
<summary>Required Tools</summary>

For setup details, see [AI Agent Configuration Guide](./AI_Agent_Configuration_Guide.md).

- Linux, macOS, or Windows
- VS Code
- Python 3.12+
- Ollama
- Qwen2.5 0.5B model
- Flask
- HTMX
- SQLite

Optional:

- AWS Kiro as an IDE only

</details>

---

## 2. AI Agent Configuration

<details>
<summary>Reference</summary>

Use the approved ASD AI configuration guide: [AI Agent Configuration Guide](./AI_Agent_Configuration_Guide.md)

This guide contains the standard instructions for:

- Environment management
- Python virtual environment setup
- Ollama runtime installation
- Ollama runtime management
- Ollama runtime testing
- AI model installation and testing
- Environment configuration
- Recommended model selection

</details>

<details>
<summary>AI Agent Usage</summary>

This lab does **not** repeat the AI runtime installation steps.

Before starting Lab 01, complete the configuration guide and verify that `ollama` runtime works

```bash
ollama --version
```

Verify that the selected model is available:

```bash
ollama list
```

For this lab, ensure that `.env` environment configuration file includes:

```env
OLLAMA_BASE_URL=http://localhost:11434/v1
OLLAMA_MODEL=qwen2.5:0.5b
```

</details>

---

## 3. Scenario Setup

> Build the initial Student Enrolment App with Flask, HTMX, SQLite, and supporting files.

<details>
<summary>System Architecture</summary>

```text
Browser
   ↓
HTMX
   ↓
Flask
 ├── SQLite
 └── Ollama
       ↓
   Qwen2.5 0.5B
```

</details>

<details>
<summary>Student Enrolment System Scenario</summary>

Build a local web application called:

```text
Student Enrolment App
```

The application allows users to:

- View all enrolled students
- Search for a student by ID
- Ask a local AI agent to explain the application
- Validate the application through an agentic loop

Follow a DevOps feedback cycle:

- Plan the system behaviour
- Build the application
- Observe application outputs
- Adapt the system based on evidence

</details>

<details>
<summary>Python Application Structure</summary>

**Project Tree**

```text
enrolment-app-open-ai/
├── requirements.txt
├── app.py
├── init_db.py
├── agentic_loop.py
├── .env
├── .gitignore
├── css/
│   └── styles.css
└── templates/
    └── index.html
```

</details>

<details>
<summary>Create Project Structure</summary>

Use vscode (Or other IDE of your choice) terminal

```bash
mkdir enrolment-app-open-ai
cd enrolment-app-open-ai
mkdir templates
mkdir css
```
</details>

<details>
<summary>Create Files Using Linux CLI</summary>

```bash
mkdir templates css

touch requirements.txt
touch app.py
touch init_db.py
touch agentic_loop.py
touch .env
touch .gitignore
touch css/styles.css
touch templates/index.html
```
</details>

<details>
<summary>Create Files using Windows PowerShell</summary>

```powershell
New-Item requirements.txt -ItemType File
New-Item app.py -ItemType File
New-Item init_db.py -ItemType File
New-Item agentic_loop.py -ItemType File
New-Item .env -ItemType File
New-Item .gitignore -ItemType File
New-Item css/styles.css -ItemType File
New-Item templates/index.html -ItemType File
```
</details>

---

## 4. Application Setup and Development

<details>
<summary>Configure the app requirements</summary>

Add the following configuration to `requirements.txt`

```text
flask
openai
python-dotenv
```
</details>

<details>
<summary>Install the app packages</summary>

**Using Linux CLI**

```bash
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install -r requirements.txt
```


**UsingOn Windows PowerShell**

```powershell
.venv\Scripts\python -m pip install --upgrade pip
.venv\Scripts\python -m pip install -r requirements.txt
```

**Verify imports on Linux:**

```bash
.venv/bin/python -c "import flask, openai, dotenv"
```

**Verify On Windows PowerShell:**

```powershell
.venv\Scripts\python -c "import flask, openai, dotenv"
```
</details>

<details>
<summary>Create <code>.env</code></summary>

Add the following content to `.env`

```env
OLLAMA_BASE_URL=http://localhost:11434/v1
OLLAMA_MODEL=qwen2.5:0.5b
```

</details>

<details>
<summary>Create<code>.gitignore</code></summary>

Add the following content to `.gitignore`

```text
.env
.venv/
venv
.vscode
vscode
__pycache__/
*.db
```
</details>

<details>
<summary>Create SQLite Database</summary>

Add the following code to `init_db.py`

```python
import sqlite3

DATABASE_NAME = "enrolment.db"

students = [
    (1, "John Smith", "ASD101"),
    (2, "Sarah Jones", "ASD101"),
    (3, "Michael Lee", "WEB201"),
    (4, "Emma Brown", "WEB201"),
    (5, "James Wilson", "DBS101"),
    (6, "Olivia White", "DBS101"),
    (7, "Daniel Green", "NET201"),
    (8, "Sophia Hall", "NET201"),
    (9, "Liam King", "SEC301"),
    (10, "Chloe Young", "SEC301")
]

conn = sqlite3.connect(DATABASE_NAME)
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS students (
    student_id INTEGER PRIMARY KEY,
    student_name TEXT NOT NULL,
    subject_code TEXT NOT NULL
)
""")

cursor.execute("DELETE FROM students")

cursor.executemany(
    "INSERT INTO students (student_id, student_name, subject_code) VALUES (?, ?, ?)",
    students
)

conn.commit()
conn.close()

print("Database initialized with 10 students.")
```
</details>

<details>
<summary>Create CSS</summary>

Add the following code to `css/styles.css`

```css
:root {
    --bg-0: #0b1020;
    --bg-1: #111a31;
    --bg-2: #1a2645;
    --surface: #0f172b;
    --surface-alt: #15213d;
    --text: #e8edf8;
    --muted: #9eb0d3;
    --accent: #45c9ff;
    --accent-strong: #1eb1ee;
    --border: #2b3a63;
    --radius: 12px;
}

* {
    box-sizing: border-box;
}

body {
    margin: 0;
    min-height: 100vh;
    font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.5;
    color: var(--text);
    background:
        radial-gradient(1100px 540px at 10% -10%, #1e3263 0%, transparent 60%),
        radial-gradient(900px 480px at 100% 0%, #1a4e74 0%, transparent 58%),
        linear-gradient(160deg, var(--bg-0) 0%, var(--bg-1) 46%, var(--bg-2) 100%);
}

.app-shell {
    max-width: 1100px;
    margin: 0 auto;
    padding: 1.25rem;
}

.app-header {
    margin-bottom: 1rem;
}

.app-header h1 {
    margin: 0;
    font-size: 1.9rem;
}

.app-header p {
    margin: 0.4rem 0 0;
    color: var(--muted);
}

.layout-grid {
    display: grid;
    grid-template-columns: 1.4fr 1fr;
    gap: 1rem;
}

.card {
    background: linear-gradient(180deg, rgba(20, 31, 56, 0.95), rgba(13, 22, 41, 0.95));
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 1rem;
    box-shadow: 0 18px 40px rgba(0, 0, 0, 0.35);
}

h2 {
    margin: 0 0 0.6rem;
    font-size: 1.1rem;
}

label {
    display: block;
    margin-bottom: 0.3rem;
    color: var(--muted);
}

button {
    margin-top: 0.5rem;
    padding: 0.5rem 0.9rem;
    border: 1px solid transparent;
    border-radius: 10px;
    font-weight: 600;
    color: #06202c;
    background: var(--accent);
    cursor: pointer;
}

button:hover {
    background: var(--accent-strong);
}

input,
textarea {
    width: 100%;
    padding: 0.55rem 0.7rem;
    border: 1px solid var(--border);
    border-radius: 10px;
    color: var(--text);
    background: var(--surface-alt);
}

.panel {
    margin-top: 0.7rem;
    padding: 0.75rem;
    border: 1px solid var(--border);
    border-radius: 10px;
    background: var(--surface);
    min-height: 2.5rem;
}

.is-hidden {
    display: none;
}

@media (max-width: 900px) {
    .layout-grid {
        grid-template-columns: 1fr;
    }
}
```
</details>

<details>
<summary>Create HTMX Frontend</summary>

Add the following code to `templates/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Student Enrolment App</title>
    <script src="https://unpkg.com/htmx.org@2.0.4"></script>
    <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
<main class="app-shell">
    <header class="app-header">
        <h1>Student Enrolment App</h1>
        <p>Use HTMX to query students and ask the local model.</p>
    </header>

    <div class="layout-grid">
        <section class="card card-left">
            <h2>Students</h2>

            <button id="toggle-students-btn" type="button">
                Show All Students
            </button>

            <div id="students-result" class="panel is-hidden"></div>

            <h2>Find Student by ID</h2>

            <form hx-get="/students/by-id" hx-target="#student-result">
                <label for="student_id">Student ID</label>
                <input id="student_id" name="student_id" type="number" min="1" placeholder="Enter student ID">
                <button type="submit">Get Student</button>
            </form>

            <div id="student-result" class="panel"></div>
        </section>

        <section class="card card-right">
            <h2>Ask Local AI Agent</h2>

            <form hx-post="/ask" hx-target="#agent-result">
                <label for="question">Question</label>
                <textarea id="question" name="question" rows="7">Explain what this Student Enrolment App does in one short paragraph.</textarea>
                <button type="submit">Ask Local Agent</button>
            </form>

            <div id="agent-result" class="panel"></div>
        </section>
    </div>
</main>

<script>
const toggleStudentsBtn = document.getElementById("toggle-students-btn");
const studentsPanel = document.getElementById("students-result");

toggleStudentsBtn.addEventListener("click", () => {
    const isHidden = studentsPanel.classList.contains("is-hidden");

    if (isHidden) {
        studentsPanel.classList.remove("is-hidden");
        toggleStudentsBtn.textContent = "Hide All Students";

        if (!studentsPanel.dataset.loaded) {
            htmx.ajax("GET", "/students", {
                target: "#students-result",
                swap: "innerHTML"
            });
            studentsPanel.dataset.loaded = "true";
        }
    } else {
        studentsPanel.classList.add("is-hidden");
        toggleStudentsBtn.textContent = "Show All Students";
    }
});
</script>

</body>
</html>
```

</details>

<details>
<summary>Create Flask Backend</summary>

Add the following code to `app.py`

```python
from flask import Flask, render_template, request, send_from_directory
from dotenv import load_dotenv
from openai import OpenAI
import sqlite3
import os

load_dotenv()

DATABASE_NAME = "enrolment.db"

OLLAMA_BASE_URL = os.getenv("OLLAMA_BASE_URL", "http://localhost:11434/v1")
OLLAMA_MODEL = os.getenv("OLLAMA_MODEL", "qwen2.5:0.5b")

app = Flask(__name__)

client = OpenAI(
    base_url=OLLAMA_BASE_URL,
    api_key="ollama"
)


def get_db_connection():
    conn = sqlite3.connect(DATABASE_NAME)
    conn.row_factory = sqlite3.Row
    return conn


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/css/<path:filename>")
def css(filename):
    return send_from_directory("css", filename)


@app.route("/students")
def get_students():
    conn = get_db_connection()
    students = conn.execute(
        "SELECT student_id, student_name, subject_code FROM students"
    ).fetchall()
    conn.close()

    html = "<ul>"
    for student in students:
        html += (
            f"<li>"
            f"{student['student_id']} - "
            f"{student['student_name']} - "
            f"{student['subject_code']}"
            f"</li>"
        )
    html += "</ul>"

    return html


@app.route("/students/<int:student_id>")
def get_student(student_id):
    conn = get_db_connection()
    student = conn.execute(
        "SELECT student_id, student_name, subject_code FROM students WHERE student_id = ?",
        (student_id,)
    ).fetchone()
    conn.close()

    if student is None:
        return "<p>Student not found.</p>", 404

    return (
        f"<p>"
        f"ID: {student['student_id']}<br>"
        f"Name: {student['student_name']}<br>"
        f"Subject: {student['subject_code']}"
        f"</p>"
    )


@app.route("/students/by-id")
def get_student_by_id():
    student_id_raw = request.args.get("student_id", "").strip()

    if not student_id_raw:
        return "<p>Student ID is required.</p>", 400

    if not student_id_raw.isdigit():
        return "<p>Student ID must be a positive integer.</p>", 400

    return get_student(int(student_id_raw))


@app.route("/ask", methods=["POST"])
def ask_local_agent():
    question = request.form.get("question", "").strip()

    if not question:
        return "<p>Question is required.</p>", 400

    try:
        response = client.chat.completions.create(
            model=OLLAMA_MODEL,
            messages=[
                {
                    "role": "system",
                    "content": (
                        "You are a concise software engineering assistant. "
                        "Answer in one short paragraph unless asked otherwise."
                    )
                },
                {
                    "role": "user",
                    "content": question
                }
            ],
            max_tokens=200,
            temperature=0.2,
        )

        answer = response.choices[0].message.content

        return f"<p>{answer}</p>"

    except Exception as exc:
        return (
            "<p>Local AI agent request failed. "
            "Check that Ollama is running and that qwen2.5:0.5b is installed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


if __name__ == "__main__":
    app.run(debug=True)
```

</details>

---

## 5. Application Testing

<details>
<summary>Run Database Initialization</summary>

**On Linux**
```bash
.venv/bin/python init_db.py
```

**On Windows PowerShell**

```powershell
.venv\Scripts\python init_db.py
```

**Expected Outcome**: 
```text
 Database initialized with 10 students.
```
</details>

<details>
<summary>Run AI Agent</summary>

Ensure the configured AI agent is running.

See the [AI Agent Configuration Guide](./AI_Agent_Configuration_Guide.md).

Verify that:
- Ollama runtime management
- Model installation and verification
- Runtime health checks
- API connectivity testing

</details>

<details>
<summary>Run Flask Application</summary>

**Start the Flask app:**

```bash
.venv/bin/python app.py
```

The app opens `http://127.0.0.1:5000` in Chrome automatically on startup.

**On Windows PowerShell:**

```powershell
.venv\Scripts\python app.py
```

To disable auto-open for one run:

```bash
FLASK_OPEN_CHROME=0 .venv/bin/python app.py
```

```powershell
$env:FLASK_OPEN_CHROME="0"
.venv\Scripts\python app.py
```

**Expected Outcome**:

```text
Running on http://127.0.0.1:5000
```

**Open in browser:**

```text
http://127.0.0.1:5000
```
</details>

<details>
<summary>Endpoints Testing</summary>

| Function | Browser Validation | API Validation |
|----------|-------------------|----------------|
| Home Page | `http://127.0.0.1:5000` | N/A |
| View Students | `http://127.0.0.1:5000/students` | `curl http://127.0.0.1:5000/students` |
| Get Student | `http://127.0.0.1:5000/students/1` | `curl http://127.0.0.1:5000/students/1` |
| Get Student by ID | `http://127.0.0.1:5000/students/by-id?student_id=1` | `curl "http://127.0.0.1:5000/students/by-id?student_id=1"` |
| Student Not Found | `http://127.0.0.1:5000/students/99` | `curl http://127.0.0.1:5000/students/99` |
| AI Assistant | Submit the form on `http://127.0.0.1:5000` | `curl -X POST http://127.0.0.1:5000/ask -d "question=Test"` |

**Expected Outcome:** All functions and endpoints return valid responses.

</details>

<details>
<summary>NFR Validation</summary>

The non-functional requirement is:

```text
GET /students returns within 500 ms.
```

**On Linux / macOS**

Run:

```bash
for i in $(seq 1 20); do
    curl -s -o /dev/null -w "%{time_total}\n" \
    http://127.0.0.1:5000/students
done
```

**Windows PowerShell**

Run:

```powershell
1..20 | ForEach-Object {
    curl.exe -s -o NUL -w "%{time_total}`n" `
    http://127.0.0.1:5000/students
}
```

**Expected Outcome:** At least 19 of 20 requests complete in ≤ 0.500 seconds.


**Pass condition:**

```text
At least 19 out of 20 requests are <= 0.500 seconds.
```

Record the result in the evidence log.

</details>

---

## 6. Agentic Loop

<details>
<summary>Agentic Loop Process</summary>

### PLAN

Record this before coding:

```text
Goal:
Build and validate a local Student Enrolment App using an open-source AI agent.

Pass condition:
All endpoint checks pass locally.

NFR:
GET /students returns within 500 ms.

AI condition:
The local Qwen model responds through the Flask /ask endpoint.

Stop condition:
Database, endpoints, local AI response, agentic loop, and evidence log are complete.
```

### ACT

The ACT stage consists of:

- Creating the Flask application
- Creating the database
- Creating the HTMX frontend
- Running the application locally
- Running deterministic validation checks
- Calling the local AI model through Ollama

### OBSERVE

The OBSERVE stage collects evidence from:

- Database initialization output
- Browser testing
- curl testing
- AI response output
- NFR response time measurements
- Agentic loop validation output

### ADAPT

The ADAPT stage requires one improvement based on observed results.

Choose one:

```text
Improve student-not-found message
Improve local AI prompt
Improve HTML output
Improve validation in agentic_loop.py
```

After applying the improvement, rerun the relevant checks.

</details>

<details>
<summary>Create Agentic Loop for DevOps Stages</summary>

Add the following code to `agentic_loop.py`

```python
import os
import sqlite3
from pathlib import Path

from dotenv import load_dotenv
from openai import OpenAI

ENV_PATH = Path(__file__).with_name(".env")
load_dotenv(dotenv_path=ENV_PATH)

PLAN = {
    "goal": "Validate Student Enrolment App behavior using a local open-source AI agent",
    "checks": [
        "/students",
        "/students/{student_id}",
        "/students/by-id",
        "/ask"
    ]
}

DATABASE_NAME = Path(__file__).with_name("enrolment.db")

OLLAMA_BASE_URL = os.getenv("OLLAMA_BASE_URL", "http://localhost:11434/v1")
OLLAMA_MODEL = os.getenv("OLLAMA_MODEL", "qwen2.5:0.5b")


def validate_student(student):
    student_id, student_name, subject_code = student

    if not isinstance(student_id, int):
        return False, "student_id must be an integer"

    if not student_name:
        return False, "student_name is required"

    if not subject_code:
        return False, "subject_code is required"

    return True, "ok"


def observe_data_quality():
    conn = sqlite3.connect(DATABASE_NAME)
    cursor = conn.cursor()

    students = cursor.execute(
        "SELECT student_id, student_name, subject_code FROM students"
    ).fetchall()

    conn.close()

    if len(students) != 10:
        return False, "Expected 10 students"

    for student in students:
        ok, msg = validate_student(student)
        if not ok:
            return False, msg

    return True, "Data validation passed"


def get_local_agent_advice(observe_message):
    prompt = (
        "You are reviewing an existing Flask Student Enrolment App.\n"
        "Current database fields: student_id, student_name, subject_code.\n"
        "Current endpoints:\n"
        "- GET /students\n"
        "- GET /students/<student_id>\n"
        "- GET /students/by-id\n"
        "- POST /ask\n\n"
        f"OBSERVE result: {observe_message}\n\n"
        "Rules:\n"
        "- Do not invent new database fields.\n"
        "- Do not invent new endpoints.\n"
        "- Do not recommend functionality that already exists.\n"
        "- Recommend one small improvement to validation, error handling, "
        "response formatting, or testing.\n"
        "- Return exactly two bullet points."
    )

    try:
        client = OpenAI(
            base_url=OLLAMA_BASE_URL,
            api_key="ollama"
        )

        response = client.chat.completions.create(
            model=OLLAMA_MODEL,
            messages=[
                {
                    "role": "system",
                    "content": "You are a concise software engineering reviewer."
                },
                {
                    "role": "user",
                    "content": prompt
                }
            ],
            max_tokens=220,
            temperature=0.2,
        )

        text = response.choices[0].message.content.strip()
        return text, None

    except Exception as exc:
        return None, f"Local AI agent unavailable ({exc})."


def main():
    print("PLAN:", PLAN)

    print("ACT: Check local database records")

    ok, msg = observe_data_quality()
    print("OBSERVE:", msg)

    if not ok:
        print("ADAPT: Fix database seed data and rerun validation")
    else:
        print("ADAPT: Proceed to endpoint checks")

    advice, advice_err = get_local_agent_advice(msg)
    if advice:
        print("ADAPT (Local AI suggestion):")
        print(advice)
    else:
        print("ADAPT (Local AI suggestion):", advice_err)


if __name__ == "__main__":
    main()
```

</details>

<details>
<summary>Execute Agentic Loop</summary>
 
- Make sure the database exists.
- Make sure Ollama is running.

**On Linux:**

```bash
.venv/bin/python agentic_loop.py
```

**On Windows PowerShell:**

```powershell
.venv\Scripts\python agentic_loop.py
```

**Expected Outcome:**

```text
PLAN: ...
ACT: Check local database records
OBSERVE: Data validation passed
ADAPT: Proceed to endpoint checks
ADAPT (Local AI suggestion):
- ...
- ...
```

If Ollama is not running, the script should still complete the deterministic validation and print a local AI unavailable message.

</details>

---

## 7. Improvement Cycle

<details>
<summary>Execute Agentic Loop</summary>

Apply one improvement.

Use the deterministic `OBSERVE` result as the source of truth.

If `ADAPT (Local AI suggestion)` is available, choose one concrete action from it.

If the local AI agent is unavailable, choose one local improvement manually.

Choose one:

```text
Improve student-not-found message
Improve local AI prompt
Improve HTML output
Improve validation in agentic_loop.py
```

Record what changed.

Re-run:

```bash
.venv/bin/python agentic_loop.py
```

Re-test:

```bash
curl http://127.0.0.1:5000/students
curl http://127.0.0.1:5000/students/1
```

Record:

```text
Improvement applied:
Before:
After:
Evidence:
```
</details>

---

## 8. Evidence Log

<details>
<summary>Record Evidence</summary>

| Check | Expected Result | Actual Result | Pass/Fail |
|---|---|---|---|
| AI configuration guide completed | `docs/AI_Agent_Configuration_Guide.md` steps completed | | |
| Ollama installed | `ollama --version` works | | |
| Qwen model installed | `ollama list` shows `qwen2.5:0.5b` | | |
| Local AI API test | `curl http://localhost:11434/api/tags` returns models | | |
| Database created | `enrolment.db` exists | | |
| Seed data | 10 students | | |
| Flask app runs | `http://127.0.0.1:5000` opens | | |
| HTMX student list | View All Students returns list | | |
| Student search | Student ID search works | | |
| Agentic loop | Data validation passed | | |
| Local agent advice | Local AI suggestion returned or unavailable message shown | | |
| GET /students | 10 students returned | | |
| GET /students/1 | Student returned | | |
| GET /students/by-id?student_id=1 | Student returned | | |
| GET /students/99 | Student not found | | |
| POST /ask | Local AI response returned | | |
| NFR | 19/20 requests <= 0.500s | | |
| Adapt | One improvement applied | | |

</details>

---

## 9. Reflection

<details>
<summary>Answer Briefly:</summary>

1. What worked?
2. What failed?
3. What did the local AI agent help with?
4. What did you improve?
5. What would you automate next?

</details>

## 10. Key Learning Point

<details>
<summary>Learning Outcome</summary>
This lab demonstrates local agent-assisted software development without a commercial subscription.

Core stack:

```text
VS Code
+
Flask
+
SQLite
+
Ollama
+
Qwen2.5 0.5B
```

**Process focus:**

```text
Human → Agent → Software System → Feedback → Adaptation
```
</details>