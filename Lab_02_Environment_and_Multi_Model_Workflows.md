# Lab 02 - Environment and Multi-Model Workflows

**Course:** Advanced Software Development with Agentic AI  
**Theme:** Environment Readiness and Multi-Model Workflows  \
**Primary IDE:** VS Code  \
**AI Agent Runtime:** Ollama  
**Implementation Model:** Qwen 2.5 0.5B  
**Review Model:** Llama 3.1 8B  
**Duration:** 60 minutes  

---

## 1. Overview

<details>
<summary>Goal</summary>

Extend the Lab 01 **Student Enrolment App** by adding **student search by subject code**, then validate the change using a local multi-model agentic workflow.

</details>

<details>
<summary>Agentic Workflow</summary>

```text
PLAN → ACT → OBSERVE → IMPLEMENTATION AGENT → REVIEW AGENT → HUMAN REVIEW → ADAPT
```

using:

```text
Qwen 2.5 0.5B → Implementation Agent
Llama 3.1 8B → Review Agent
Human → Final Decision Maker
```

</details>

<details>
<summary> Expected Results</summary>

By the end of this lab, students should have:

- Reused the Lab 01 Student Enrolment App
- Added a new `/students/by-subject` endpoint
- Updated the HTMX frontend with a subject-code search form
- Configured Qwen as the implementation agent
- Configured Llama as the review agent
- Executed a local multi-model agentic loop
- Completed endpoint validation
- Completed NFR validation
- Recorded evidence
- Completed a short reflection

</details>

---

## 2. Prerequisites and Configuration

<details>

<summary>Required Tools</summary>

For setup details, see [AI Agent Configuration Guide](../docs/AI_Agent_Configuration_Guide.md).

- Linux, macOS, or Windows
- VS Code (or optional AWS Kiro)
- Python 3.12+
- Ollama
- Qwen 2.5 0.5B
- Llama 3.1 8B
- Flask
- HTMX
- SQLite
- Completed Lab 01 Student Enrolment App

</details>

<details>
<summary>Update <code>.env</code>:</summary>

```env
OLLAMA_BASE_URL=http://localhost:11434/v1
OLLAMA_MODEL=qwen2.5:0.5b
OLLAMA_REVIEW_MODEL=llama3.1:8b
```

</details>

<details>
<summary>Model Roles</summary>

| Model | Role | Purpose |
|---|---|---|
| `qwen2.5:0.5b` | Implementation Agent | Suggest scoped implementation improvements |
| `llama3.1:8b` | Review Agent | Review the recommendation and identify evidence-backed risks |
| Human | Final Decision Maker | Accept, partially accept, or reject recommendations |

</details>

---

## 3. Scenario Setup

<details>
<summary>Enrolment App Upgrades</summary>

Lab 01 provides a Student Enrolment App that can:

- View all enrolled students
- Search for a student by ID
- Ask a local AI agent to explain the application
- Validate the application through an agentic loop

Lab 02 adds one behavior:

```text
Search students by subject code.
```

Example subject codes:

```text
ASD101
WEB201
DBS101
NET201
SEC301
```

Users should be able to return all students enrolled in a selected subject.

</details>

<details>
<summary>Project Structure</summary>

Use the existing `Lab 01` app:

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
<summary>Files modified in <code>Lab 02</code></summary>

```text
app.py
templates/index.html
css/styles.css
agentic_loop.py
.env
```

No new application should be created from scratch.

</details>

---

## 4. Application Setup and Development

<details>
<summary>Create <code> requirements.txt</code></summary>

```text
flask
openai
python-dotenv
```
</details>

<details>
<summary>Install the app packages</summary>

**Linux/macOS:**

```bash
.venv/bin/python -m pip install -r requirements.txt
```

**Windows PowerShell:**

```powershell
.venv\Scripts\python -m pip install -r requirements.txt
```
</details>

<details>
<summary>Update <code>.env</code> content</summary>

```env
OLLAMA_BASE_URL=http://localhost:11434/v1
OLLAMA_MODEL=qwen2.5:0.5b
OLLAMA_REVIEW_MODEL=llama3.1:8b
```

</details>

<details>
<summary>Confirm <code>.gitignore</code> content</summary>

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
<summary>Database</summary>

Use the existing db initialized using `init_db.py`. The existing `students` table remains:

```text
student_id
student_name
subject_code
```

If needed, re-run the database initialization.

**Linux/macOS:**

```bash
.venv/bin/python init_db.py
```

**Windows PowerShell:**

```powershell
.venv\Scripts\python init_db.py
```

**Expected output:**

```text
Database initialized with 10 students.
```

</details>

<details>
<summary>CSS</summary>

Update `css/styles.css`

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
<summary>HTMX Frontend</summary>

Update with `templates/index.html`

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

            <h2>Find Students by Subject Code</h2>

            <form hx-get="/students/by-subject" hx-target="#subject-result">
                <label for="subject_code">Subject Code</label>
                <input id="subject_code" name="subject_code" type="text" placeholder="Example: ASD101">
                <button type="submit">Find Students</button>
            </form>

            <div id="subject-result" class="panel"></div>
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
<summary>Flask Backend</summary>

Update `app.py` to include the new `/students/by-subject`

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


@app.route("/students/by-subject")
def get_students_by_subject():
    subject_code = request.args.get("subject_code", "").strip().upper()

    if not subject_code:
        return "<p>Subject code is required.</p>", 400

    conn = get_db_connection()
    students = conn.execute(
        "SELECT student_id, student_name, subject_code FROM students WHERE subject_code = ?",
        (subject_code,)
    ).fetchall()
    conn.close()

    if not students:
        return f"<p>No students found for subject code {subject_code}.</p>", 404

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
<summary>Start <code>Ollama</code> Engine</summary>

**On Linux:**

```bash
ollama serve
```

**On Win PS:**

```powershell
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" serve
```

**Confirm Ollama is running.**

</details>

<details>
<summary>Run Flask App</summary>

**Linux/macOS:**

```bash
.venv/bin/python app.py
```

The app opens `http://127.0.0.1:5000` in Chrome automatically on startup.

**Windows PowerShell:**

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

**Open:**

```text
http://127.0.0.1:5000
```

</details>

<details>
<summary>Endpoints Testing</summary>

| Function | Browser Validation | API Validation |
|---|---|---|
| Home Page | `http://127.0.0.1:5000` | N/A |
| View Students | `http://127.0.0.1:5000/students` | `curl http://127.0.0.1:5000/students` |
| Get Student | `http://127.0.0.1:5000/students/1` | `curl http://127.0.0.1:5000/students/1` |
| Get Student by ID | `http://127.0.0.1:5000/students/by-id?student_id=1` | `curl "http://127.0.0.1:5000/students/by-id?student_id=1"` |
| Student Not Found | `http://127.0.0.1:5000/students/99` | `curl http://127.0.0.1:5000/students/99` |
| Search by Subject | `http://127.0.0.1:5000/students/by-subject?subject_code=ASD101` | `curl "http://127.0.0.1:5000/students/by-subject?subject_code=ASD101"` |
| Subject Not Found | `http://127.0.0.1:5000/students/by-subject?subject_code=ABC999` | `curl "http://127.0.0.1:5000/students/by-subject?subject_code=ABC999"` |
| AI Assistant | Submit the form on the home page | `curl -X POST http://127.0.0.1:5000/ask -d "question=Test"` |

Expected outcome:

```text
All functions and endpoints return valid responses.
```

</details>

<details>
<summary>NFR Validation </summary>

The non-functional requirement is:

```text
GET /students/by-subject?subject_code=ASD101 returns within 500 ms.
```

Pass condition:

```text
At least 19 out of 20 requests complete in <= 0.500 seconds.
```

**Linux/macOS:**

```bash
for i in $(seq 1 20); do
    curl -s -o /dev/null -w "%{time_total}\n" \
    "http://127.0.0.1:5000/students/by-subject?subject_code=ASD101"
done
```

**Windows PowerShell**

```powershell
1..20 | ForEach-Object {
    curl.exe -s -o NUL -w "%{time_total}`n" `
    "http://127.0.0.1:5000/students/by-subject?subject_code=ASD101"
}
```

Record the result in the evidence log.

</details>

---

## 6. Agentic Loop

<details>
<summary>Python updated Agentic Loop <code>agentic_loop.py</code></summary>
Replace the Lab 01 `agentic_loop.py` with the following Lab 02 version.

This version uses:

```text
Qwen 2.5 0.5B as Implementation Agent
Llama 3.1 8B as Review Agent
Human as Final Decision Maker
```

```python
import os
import sqlite3
from pathlib import Path

from dotenv import load_dotenv
from openai import OpenAI

ENV_PATH = Path(__file__).with_name(".env")
load_dotenv(dotenv_path=ENV_PATH)

PLAN = {
    "goal": "Validate Student Enrolment App behavior using a local multi-agent workflow",
    "checks": [
        "/students",
        "/students/{student_id}",
        "/students/by-id",
        "/students/by-subject",
        "/ask"
    ]
}

DATABASE_NAME = Path(__file__).with_name("enrolment.db")

OLLAMA_BASE_URL = os.getenv(
    "OLLAMA_BASE_URL",
    "http://localhost:11434/v1"
)

IMPLEMENTATION_MODEL = os.getenv(
    "OLLAMA_MODEL",
    "qwen2.5:0.5b"
)

REVIEW_MODEL = os.getenv(
    "OLLAMA_REVIEW_MODEL",
    "llama3.1:8b"
)


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
        """
        SELECT
            student_id,
            student_name,
            subject_code
        FROM students
        """
    ).fetchall()

    conn.close()

    if len(students) != 10:
        return False, "Expected 10 students"

    for student in students:
        ok, msg = validate_student(student)

        if not ok:
            return False, msg

    return True, "Data validation passed"


def observe_subject_search(subject_code):
    conn = sqlite3.connect(DATABASE_NAME)
    cursor = conn.cursor()

    students = cursor.execute(
        """
        SELECT
            student_id,
            student_name,
            subject_code
        FROM students
        WHERE subject_code = ?
        """,
        (subject_code,)
    ).fetchall()

    conn.close()

    if not students:
        return False, (
            f"No students found for subject code {subject_code}"
        )

    for student in students:
        if student[2] != subject_code:
            return False, (
                f"Unexpected subject code found: {student[2]}"
            )

    return True, (
        f"Subject search validation passed for {subject_code}"
    )


def call_model(
    model_name,
    system_prompt,
    user_prompt,
    max_tokens=120
):
    try:
        client = OpenAI(
            base_url=OLLAMA_BASE_URL,
            api_key="ollama",
            timeout=180.0
        )

        response = client.chat.completions.create(
            model=model_name,
            messages=[
                {
                    "role": "system",
                    "content": system_prompt
                },
                {
                    "role": "user",
                    "content": user_prompt
                }
            ],
            max_tokens=max_tokens,
            temperature=0.1
        )

        content = response.choices[0].message.content

        if content and content.strip():
            return content.strip(), None

        return "No response generated.", None

    except Exception as exc:
        return None, (
            f"{model_name} unavailable or timed out ({exc})"
        )


def get_implementation_agent_advice(observe_message):
    prompt = (
        "You are the IMPLEMENTATION AGENT for a Flask "
        "Student Enrolment App.\n\n"

        "Current database fields:\n"
        "- student_id\n"
        "- student_name\n"
        "- subject_code\n\n"

        "Important domain rule:\n"
        "- subject_code is NOT unique.\n"
        "- Multiple students may enrol in the same subject.\n"
        "- Never recommend a unique constraint on subject_code.\n\n"

        "Current endpoints:\n"
        "- GET /students\n"
        "- GET /students/<student_id>\n"
        "- GET /students/by-id\n"
        "- GET /students/by-subject\n"
        "- POST /ask\n\n"

        f"Validation Evidence:\n{observe_message}\n\n"

        "Task:\n"
        "Review ONLY the existing subject-code search feature.\n\n"

        "Rules:\n"
        "- Do not invent new database fields.\n"
        "- Do not invent new endpoints.\n"
        "- Do not modify endpoint contracts.\n"
        "- Do not suggest new application features.\n"
        "- Do not recommend subject_code uniqueness.\n"
        "- Focus only on validation, error handling, "
        "response formatting, or testing.\n"
        "- If the evidence does not support an improvement, "
        "write: No evidence-backed improvement identified.\n"
        "- Return exactly two bullet points, or the no-evidence sentence.\n"
    )

    return call_model(
        IMPLEMENTATION_MODEL,
        (
            "You are a concise implementation assistant. "
            "Follow the rules exactly. "
            "Do not invent requirements."
        ),
        prompt,
        max_tokens=120
    )


def get_review_agent_advice(
    implementation_message,
    observe_message
):
    prompt = (
        "Review ONLY the implementation-agent "
        "recommendation.\n\n"

        f"Implementation Recommendation:\n"
        f"{implementation_message}\n\n"

        f"Validation Evidence:\n"
        f"{observe_message}\n\n"

        "Application Scope:\n"
        "- database fields: student_id, "
        "student_name, subject_code\n"
        "- endpoints: /students, "
        "/students/<student_id>, "
        "/students/by-id, "
        "/students/by-subject, "
        "/ask\n\n"

        "Important domain rule:\n"
        "- subject_code is NOT unique.\n"
        "- Multiple students may enrol in the same subject.\n"
        "- Any recommendation to make subject_code unique is invalid.\n\n"

        "Rules:\n"
        "- Do not invent new database fields.\n"
        "- Do not invent new endpoints.\n"
        "- Do not suggest new features.\n"
        "- Identify only evidence-backed risks "
        "or corrections.\n"
        "- If no evidence-backed risk exists, say so.\n"
        "- Return exactly three lines.\n\n"

        "Format:\n"
        "Risk: <one short sentence>\n"
        "Correction: <one short sentence>\n"
        "Retest: <one short sentence>\n\n"

        "If no risk is supported by the evidence, use:\n"
        "Risk: No evidence-backed risk identified.\n"
        "Correction: No correction required.\n"
        "Retest: Repeat validation after future changes.\n\n"

        "- Maximum 35 words total.\n"
        "- Do not explain reasoning.\n"
    )

    return call_model(
        REVIEW_MODEL,
        (
            "You are a concise software review assistant. "
            "Follow the output format exactly. "
            "Do not invent requirements."
        ),
        prompt,
        max_tokens=100
    )


def human_review():
    print()
    print("HUMAN REVIEW")
    print("1 - Accept")
    print("2 - Partially Accept")
    print("3 - Reject")

    decision = input("Decision: ").strip()

    if decision == "1":
        return "Accept"

    if decision == "2":
        return "Partially Accept"

    return "Reject"


def adapt(decision):
    print()

    if decision == "Accept":
        print(
            "ADAPT: Apply recommendation and rerun validation."
        )

    elif decision == "Partially Accept":
        print(
            "ADAPT: Apply selected recommendations and "
            "rerun validation."
        )

    else:
        print(
            "ADAPT: Keep current implementation and "
            "document rationale."
        )


def main():
    print("=" * 60)
    print("ASD LAB 02 AGENTIC LOOP")
    print("=" * 60)

    print()
    print("PLAN")
    print(PLAN)

    print()
    print("ACT")
    print("Check local database records")

    ok_data, msg_data = observe_data_quality()

    print()
    print("OBSERVE")
    print(msg_data)

    ok_subject, msg_subject = observe_subject_search(
        "ASD101"
    )

    print(msg_subject)

    observe_message = (
        f"{msg_data}. "
        f"{msg_subject}."
    )

    print()
    print("IMPLEMENTATION AGENT")
    print(f"Model: {IMPLEMENTATION_MODEL}")

    implementation_advice, implementation_error = (
        get_implementation_agent_advice(
            observe_message
        )
    )

    if implementation_advice:
        print()
        print(implementation_advice)
    else:
        print()
        print(implementation_error)
        implementation_advice = (
            "Implementation agent unavailable."
        )

    print()
    print("REVIEW AGENT")
    print(f"Model: {REVIEW_MODEL}")

    review_advice, review_error = (
        get_review_agent_advice(
            implementation_advice,
            observe_message
        )
    )

    if review_advice:
        print()
        print(review_advice)
    else:
        print()
        print(review_error)

    print()
    print("HUMAN DECISION")

    decision = human_review()

    print()
    print(f"Decision: {decision}")

    adapt(decision)

    print()
    print("LOOP COMPLETE")


if __name__ == "__main__":
    main()
```
</details>

<details>
<summary>Run Agentic Loop</summary>

Open a second terminal.

**Linux/macOS:**

```bash
.venv/bin/python agentic_loop.py
```

**Windows PowerShell:**

```powershell
.venv\Scripts\python agentic_loop.py
```

**Expected flow:**

```text
PLAN
ACT
OBSERVE
IMPLEMENTATION AGENT
REVIEW AGENT
HUMAN REVIEW
ADAPT
LOOP COMPLETE
```

</details>

---

## 7. Improvement Cycle

<details>
<summary>Improve and Record</summary>

Apply one improvement based on evidence.

Choose one:

```text
Improve subject-code validation
Improve empty-result message
Improve HTML output
Improve review-agent prompt
Improve validation in agentic_loop.py
```

After applying the improvement:

1. Rerun the agentic loop.
2. Rerun endpoint tests.
3. Rerun NFR validation if the endpoint changed.
4. Record the before/after result.

Record:

```text
Improvement applied:
Before:
After:
Evidence:
Human decision:
```

</details>

---

## 8. Evidence Log

<details>
<summary>Record Evidence</summary>

| Check | Expected Result | Actual Result | Pass/Fail |
|---|---|---|---|
| AI configuration guide completed | Qwen and Llama setup completed | | |
| Ollama installed | `ollama --version` works | | |
| Qwen model installed | `ollama list` shows `qwen2.5:0.5b` | | |
| Llama model installed | `ollama list` shows `llama3.1:8b` | | |
| Database created | `enrolment.db` exists | | |
| Seed data | 10 students | | |
| Flask app runs | `http://127.0.0.1:5000` opens | | |
| Baseline `/students` | 10 students returned | | |
| Baseline `/students/1` | Student returned | | |
| New `/students/by-subject?subject_code=ASD101` | Matching students returned | | |
| New `/students/by-subject?subject_code=ABC999` | No students found message returned | | |
| HTMX subject search | Browser form returns matching students | | |
| Implementation agent | Qwen recommendation returned or unavailable message shown | | |
| Review agent | Llama review returned or unavailable message shown | | |
| Human review | AI recommendation assessed | | |
| NFR | 19/20 subject-search requests <= 0.500s | | |
| Adapt | One improvement applied | | |

</details>

---

## 9. Reflection

<details>
<summary>Answer Briefly:</summary>

1. What worked?
2. What failed?
3. What did Qwen help with?
4. What did Llama help with?
5. Which AI recommendation was accepted, partially accepted, or rejected?
6. What evidence most strongly validated correctness?
7. What would you automate next?

</details>

---

## 10. Key Learning Point

<details>
<summary>Learning Outcome</summary>
This lab demonstrates using multiple local models with different responsibilities.

Lab 01 used:

```text
Human
+
Single Local AI Agent
+
Software System
+
Feedback
+
Adaptation
```

Lab 02 workflow:

```text
Human
+
Implementation Agent
+
Review Agent
+
Software System
+
Evidence
+
Human Decision
```

**Process focus:**

```text
One agent may implement.
Another agent may review.
Evidence validates.
Humans decide.
```

Treat recommendations as input, not proof. Validate before implementation.
</details>
