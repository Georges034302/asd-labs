# Lab 03 - Prompt Engineering, Specifications, and Context Management

**Course:** Advanced Software Development with Agentic AI (ASD)  
**Theme:** Prompt Engineering, Specifications, and Context Management  
**Primary IDE:** VS Code (Optional IDE: AWS Kiro)  
**AI Runtime:** Ollama  
**Implementation Agent:** Qwen 2.5 0.5B  
**Review Agent:** Llama 3.1 8B  
**Duration:** 60 minutes

---

## 1. Overview

<details>
<summary>Goal</summary>

Extend Lab 02 by externalizing prompt text into files and adding context-based response flow.

</details>

<details>
<summary>Agentic Workflow</summary>

```text
PLAN -> ACT -> OBSERVE -> PROMPT AGENT -> REVIEW AGENT -> HUMAN REVIEW -> ADAPT
```

</details>

<details>
<summary>Expected Results</summary>

By the end of this lab, students should have:

- Reused the Lab 02 application
- Created prompt files under prompts/
- Added and tested POST /ask-with-context
- Updated the HTMX frontend with a context form
- Loaded prompts from files in app.py
- Executed the prompt and review loop in agentic_loop.py
- Validated live endpoint behavior and NFR timing
- Recorded evidence

</details>

---

## 2. Prerequisites and Configuration

<details>
<summary>Prerequisites</summary>

Complete first:

- Lab 01
- Lab 02
- AI Agent Configuration Guide: [AI Agent Configuration Guide](../docs/AI_Agent_Configuration_Guide.md)

Required models:

- qwen2.5:0.5b
- llama3.1:8b

Verify:

```bash
ollama list
```

</details>

---

## 3. Scenario Setup

<details>
<summary>Project Structure</summary>

```text
enrolment-app-open-ai/
├── prompts/
│   ├── implementation_system_prompt.txt
│   ├── implementation_task_prompt.txt
│   ├── context_qa_task_prompt.txt
│   ├── review_system_prompt.txt
│   └── review_task_prompt.txt
├── app.py
├── agentic_loop.py
└── templates/
    └── index.html
```

</details>

<details>
<summary>Create Prompt Folder and Files in the App Folder</summary>

Linux/macOS:

```bash
cd enrolment-app-open-ai
mkdir -p prompts
touch prompts/implementation_system_prompt.txt prompts/implementation_task_prompt.txt prompts/context_qa_task_prompt.txt
touch prompts/review_system_prompt.txt prompts/review_task_prompt.txt
```

Windows PowerShell:

```powershell
cd enrolment-app-open-ai
mkdir prompts
New-Item prompts/implementation_system_prompt.txt -ItemType File
New-Item prompts/implementation_task_prompt.txt -ItemType File
New-Item prompts/context_qa_task_prompt.txt -ItemType File
New-Item prompts/review_system_prompt.txt -ItemType File
New-Item prompts/review_task_prompt.txt -ItemType File
```

</details>

---

## 4. Application Setup and Development

<details>
<summary>Use Lab 02 App</summary>

Confirm these exist:

- requirements.txt
- .env
- .gitignore
- enrolment.db
- css/styles.css
- templates/index.html

Reuse Lab 02 application. Do not create a new app.

</details>

<details>
<summary>Prompt Specification Files</summary>

prompts/implementation_system_prompt.txt

```text
You are the IMPLEMENTATION AGENT for a Flask Student Enrolment App.

Use only supplied evidence or context. Follow the rules exactly. Do not invent requirements.

Rules:
- Do not invent new database fields.
- Do not invent new endpoints.
- Do not invent functionality.
- Do not modify endpoint contracts.
- Do not suggest new application features.
- Do not recommend a unique constraint on subject_code.
- Focus only on validation, error handling, response formatting, or testing.

If information is unavailable in the supplied context, say:

Information unavailable in supplied context.
```

prompts/implementation_task_prompt.txt

```text
Application Name:
Student Enrolment App

Database Fields:
- student_id
- student_name
- subject_code

Important domain rule:
- subject_code is NOT unique.
- Multiple students may enrol in the same subject.
- Never recommend a unique constraint on subject_code.

Endpoints:
- GET /students
- GET /students/{student_id}
- GET /students/by-id
- GET /students/by-subject
- POST /ask
- POST /ask-with-context

These are the ONLY endpoints that exist. Do not invent or recommend create, update, or delete operations for students.

Validation Evidence:
{{VALIDATION_EVIDENCE}}

Task:
Review ONLY the existing subject-code search feature using the validation evidence above.

Base your answer only on the Validation Evidence above — it already reflects real HTTP requests made to the running Flask app and real database checks. Do not claim to have made requests yourself, and do not rely on assumptions beyond that evidence. If the evidence indicates the app is not responding, say: Unable to verify live endpoint behavior.

Output rules:
- If the evidence does not support an improvement, write: No evidence-backed improvement identified.
- Return exactly two bullet points, or the no-evidence sentence.
```

prompts/context_qa_task_prompt.txt

```text
Application Name:
Student Enrolment App

Database Fields:
- student_id
- student_name
- subject_code

Endpoints:
- GET /students
- GET /students/{student_id}
- GET /students/by-id
- GET /students/by-subject
- POST /ask
- POST /ask-with-context

These are the ONLY features that exist. Do not mention create, add, update, delete, or edit operations for students — they are not implemented.

Task:
Explain the Student Enrolment App using only the supplied context above. Maximum 60 words.
```

prompts/review_system_prompt.txt

```text
You are a concise software review assistant.

Follow the output format exactly. Do not invent requirements.

Rules:
- Do not invent new database fields.
- Do not invent new endpoints.
- Do not suggest new features.
- Identify only evidence-backed risks or corrections.
```

prompts/review_task_prompt.txt

```text
Application Scope:
- database fields: student_id, student_name, subject_code
- endpoints: /students, /students/<student_id>, /students/by-id, /students/by-subject, /ask, /ask-with-context

Important domain rule:
- subject_code is NOT unique.
- Multiple students may enrol in the same subject.
- Any recommendation to make subject_code unique is invalid.

Implementation Recommendation:
{{IMPLEMENTATION_RECOMMENDATION}}

Validation Evidence:
{{VALIDATION_EVIDENCE}}

Task:
Before reviewing the recommendation, use the validation evidence provided. Review only the implementation-agent recommendation and identify evidence-backed risks or corrections. If no evidence-backed risk exists, say so.

Format:
Risk: <one short sentence>
Correction: <one short sentence>
Retest: <one short sentence>

If no risk is supported by the evidence, use:
Risk: No evidence-backed risk identified.
Correction: No correction required.
Retest: Repeat validation after future changes.

Output rules:
- Return exactly three lines.
- Maximum 35 words total.
- Do not explain reasoning.
```

</details>

<details>
<summary>Backend Flask API</summary>

```python
from flask import Flask, render_template, request, send_from_directory
from dotenv import load_dotenv
from openai import OpenAI
from pathlib import Path
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

PROMPT_DIR = Path(__file__).with_name("prompts")


def load_prompt(filename):
    prompt_path = PROMPT_DIR / filename
    return prompt_path.read_text(encoding="utf-8").strip()


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


@app.route("/ask-with-context", methods=["POST"])
def ask_with_context():
    question = request.form.get("question", "").strip()

    if not question:
        return "<p>Question is required.</p>", 400

    try:
        system_prompt = load_prompt("implementation_system_prompt.txt")
        task_prompt = load_prompt("context_qa_task_prompt.txt")

        final_prompt = f"""
{task_prompt}

User Question:

{question}
"""

        response = client.chat.completions.create(
            model=OLLAMA_MODEL,
            messages=[
                {
                    "role": "system",
                    "content": system_prompt
                },
                {
                    "role": "user",
                    "content": final_prompt
                }
            ],
            max_tokens=300,
            temperature=0,
        )

        answer = response.choices[0].message.content
        return f"<p>{answer}</p>"

    except Exception as exc:
        return (
            "<p>Context-aware request failed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


if __name__ == "__main__":
    app.run(debug=True)
```

</details>

<details>
<summary>HTMX Frontend</summary>

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

            <h2>Ask With Context</h2>

            <form hx-post="/ask-with-context" hx-target="#context-result">
                <label for="context-question">Question</label>
                <textarea id="context-question" name="question" rows="4">Explain the Student Enrolment App.</textarea>
                <button type="submit">Ask With Context</button>
            </form>

            <div id="context-result" class="panel"></div>
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

---

## 5. Application Testing

<details>
<summary>Run the Python-HTMX app <code>app.py</code></summary>

```bash
.venv/bin/python app.py
```

Open `http://127.0.0.1:5000` in a web-browser.

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
| AI Assistant | Submit home page form | `curl -X POST http://127.0.0.1:5000/ask -d "question=Test"` |
| Context Assistant | Submit context form | `curl -X POST http://127.0.0.1:5000/ask-with-context -d "question=What endpoints exist?"` |

Expected:

```text
All endpoints return valid responses.
```

</details>

<details>
<summary>NFR Validation</summary>

Requirement:

```text
GET /students/by-subject?subject_code=ASD101 returns within 500 ms.
```

Pass condition:

```text
At least 19 out of 20 requests complete in <= 0.500 seconds.
```

Linux/macOS:

```bash
for i in $(seq 1 20); do
    curl -s -o /dev/null -w "%{time_total}\n" \
    "http://127.0.0.1:5000/students/by-subject?subject_code=ASD101"
done
```

Windows PowerShell:

```powershell
1..20 | ForEach-Object {
    curl.exe -s -o NUL -w "%{time_total}`n" `
    "http://127.0.0.1:5000/students/by-subject?subject_code=ASD101"
}
```

Record results in the evidence log.

</details>

---

## 6. Agentic Loop

<details>
<summary>Python Agentic Loop</summary>

Update `agentic_loop.py`

```python
import os
import sqlite3
from pathlib import Path

import requests
from dotenv import load_dotenv
from openai import OpenAI

ENV_PATH = Path(__file__).with_name(".env")
load_dotenv(dotenv_path=ENV_PATH)

PROMPT_DIR = Path(__file__).with_name("prompts")


def load_prompt(filename):
    prompt_path = PROMPT_DIR / filename
    return prompt_path.read_text(encoding="utf-8").strip()


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


def get_sample_student():
    conn = sqlite3.connect(DATABASE_NAME)
    cursor = conn.cursor()

    row = cursor.execute(
        """
        SELECT student_id, subject_code
        FROM students
        LIMIT 1
        """
    ).fetchone()

    conn.close()

    return row


def observe_live_endpoints(sample_student):
    results = []

    student_id, subject_code = (
        sample_student if sample_student else (None, None)
    )

    def check(label, method, url, **kwargs):
        try:
            response = requests.request(
                method, url, timeout=5, **kwargs
            )
            content_ok = bool(response.text and response.text.strip())
            results.append(
                f"{label} -> HTTP {response.status_code}, "
                f"content_ok={content_ok}"
            )
        except Exception as exc:
            results.append(f"{label} -> error: {exc}")

    check("/students", "GET", "http://127.0.0.1:5000/students")

    if student_id is not None:
        check(
            "/students/<student_id>",
            "GET",
            f"http://127.0.0.1:5000/students/{student_id}"
        )
        check(
            "/students/by-id",
            "GET",
            f"http://127.0.0.1:5000/students/by-id?student_id={student_id}"
        )
    else:
        results.append("/students/<student_id> -> skipped: no sample student found")
        results.append("/students/by-id -> skipped: no sample student found")

    if subject_code is not None:
        check(
            "/students/by-subject",
            "GET",
            f"http://127.0.0.1:5000/students/by-subject?subject_code={subject_code}"
        )
    else:
        results.append("/students/by-subject -> skipped: no sample student found")

    check(
        "/ask",
        "POST",
        "http://127.0.0.1:5000/ask",
        data={"question": "What does this app do?"}
    )

    return results


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
    task_prompt = load_prompt("implementation_task_prompt.txt").replace(
        "{{VALIDATION_EVIDENCE}}", observe_message
    )

    return call_model(
        IMPLEMENTATION_MODEL,
        load_prompt("implementation_system_prompt.txt"),
        task_prompt,
        max_tokens=120
    )


def get_review_agent_advice(
    implementation_message,
    observe_message
):
    task_prompt = (
        load_prompt("review_task_prompt.txt")
        .replace("{{IMPLEMENTATION_RECOMMENDATION}}", implementation_message)
        .replace("{{VALIDATION_EVIDENCE}}", observe_message)
    )

    return call_model(
        REVIEW_MODEL,
        load_prompt("review_system_prompt.txt"),
        task_prompt,
        max_tokens=150
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

    sample_student = get_sample_student()
    sample_subject_code = sample_student[1] if sample_student else "ASD101"

    ok_subject, msg_subject = observe_subject_search(
        sample_subject_code
    )

    print(msg_subject)

    live_results = observe_live_endpoints(sample_student)

    observe_message = (
        f"{msg_data}. "
        f"{msg_subject}. "
        f"Live endpoint checks: " + "; ".join(live_results)
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
<summary>Run the Agentic Loop</summary>

The agentic loop makes live HTTP requests against the running Flask app, so start `app.py` first in a separate terminal (leave it running):

```bash
cd enrolment-app-open-ai
.venv/bin/python app.py
```

In a second terminal, run the agentic loop (no need to run `init_db.py` again if you already seeded the database earlier):

```bash
cd enrolment-app-open-ai
.venv/bin/python agentic_loop.py
```

Expected:

```text
PLAN
ACT
OBSERVE
PROMPT AGENT
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

Work in groups of 5. Refer to step 6 results.

**Focus:**

1. Database check — verify records match the expected student data.
2. Live endpoint check — send real HTTP requests and inspect the actual responses.
3. Review agent — must use evidence from both the database check and the live endpoint check.

**Implementation agent must:**

- Confirm the Flask app is running locally.
- Base its recommendation only on the supplied live-endpoint and database evidence, not assumptions.
- Propose one evidence-based improvement, scoped to the subject-code search feature.

**Review agent must:**

- Review the implementation agent's recommendation using that same evidence.
- Identify evidence-backed risks or corrections, or confirm none exist.

**Workflow:**

```text
PLAN -> OBSERVE (database + live endpoints) -> IMPLEMENTATION AGENT -> REVIEW AGENT -> HUMAN REVIEW -> ADAPT
```

**Steps:**

1. Choose one prompt file to improve: 
    - implementation system
    - implementation task
    - context QA task
    - review system
    - review task.
2. Apply the change.
3. Rerun the endpoint tests and the agentic loop.
4. Record the result below.

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
| Prompt folder created | Yes | | |
| Prompt files created | Yes | | |
| qwen2.5:0.5b installed | Yes | | |
| llama3.1:8b installed | Yes | | |
| Context form added to frontend | Yes | | |
| /ask-with-context works | Yes | | |
| NFR: by-subject <= 500ms (19/20 requests) | Pass | | |
| Agentic loop live endpoint checks | All HTTP 200 | | |
| Implementation agent output | Returned | | |
| Review agent output | Returned | | |
| Improvement cycle completed | Recorded | | |

</details>

---

## 9. Reflection

<details>
<summary>Answer Briefly</summary>

1. Which prompt change from the improvement cycle improved output quality most, and why?
2. What evidence-backed risk did the review agent identify from the database or live endpoint checks?
3. Which validation evidence (database check or live endpoint check) had the greatest impact on the agents' recommendations?
4. What should be automated next in the PLAN -> ACT -> OBSERVE -> IMPLEMENTATION AGENT -> REVIEW AGENT -> HUMAN REVIEW -> ADAPT loop?

</details>

---

## 10. Key Learning Point

<details>
<summary>Learning Outcome</summary>

Keep these concerns separate:

- Code (`app.py`, `agentic_loop.py`)
- Runtime configuration (`.env`, model names)
- Prompt assets (`prompts/*.txt`)
- Validation evidence (database check + live endpoint check)

Focus:

```text
PLAN -> ACT -> OBSERVE -> IMPLEMENTATION AGENT -> REVIEW AGENT -> HUMAN REVIEW -> ADAPT
```

</details>
