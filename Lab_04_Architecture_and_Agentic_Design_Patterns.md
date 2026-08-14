# Lab 04 - Software Architecture and Design Patterns for Agentic AI Systems

**Course:** Advanced Software Development with Agentic AI (ASD) \
**Theme:** Architecture and Agentic Decision Patterns \
**Primary IDE:** VS Code \
**AI Runtime:** Ollama \
**Duration:** 60 Minutes

## 1. Overview

<details>
<summary>Goal</summary>

Transform the Lab 03 monolithic Student Enrolment App into a containerized microservices architecture with clear UI-mode and AI-mode behavior, and factorize the Python backend from monolith to modular services consisting of:

- frontend-service (modular HTMX frontend service)
- enrolment-service (modular python-flask backend service)
- database-service
- agentic loop (modular interactive agentic loop)

Students will design the architecture, define service boundaries, create architecture artifacts, and validate architecture quality using evidence-driven review.

</details>

<details>
<summary>Workflow</summary>

PLAN → OBSERVE → IMPLEMENT → REVIEW → ADAPT -> IMPROVE

</details>

<details>
<summary>Expected Results</summary>

By the end of this lab, students should have:

- A completed Architecture Decision Record (ADR)
- A validated UI-mode and AI-mode flow design for the microservices app
- A service architecture for frontend-service, enrolment-service, and database-service
- A modularized Python backend structure factored from the Lab 03 monolith
- Architecture analysis activities completed with agentic loop evidence
- A working Docker Compose architecture for the three-service design
- A clear three-service design with defined responsibilities and ownership
- A containerized project structure for the app-root workflow
- An evidence log containing review notes, ADR feedback, and adaptation steps

</details>


---

## 2. Prerequisites and Configuration

<details>
<summary>Prerequisites</summary>

To start this lab, students should have:

Complete:

- Lab 01
- Lab 02
- Lab 03

Required:

- Docker Desktop
- Ollama

Verify:

```bash
docker --version
docker-compose --version
ollama list
```

If `docker` is not found on Ubuntu:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
docker-compose --version
```

Windows PowerShell (Docker Desktop):

Install Docker Desktop (run PowerShell as Administrator):

```powershell
winget install -e --id Docker.DockerDesktop
```

```powershell
docker --version
docker-compose --version
```

After installation, start Docker Desktop, then restart PowerShell and verify again.
</details>

---

## 3. Scenario

<details>
<summary>Student enrolment app</summary>

The Lab 03 application currently exists as a single deployment unit.

The business now requires:

- Independent frontend deployment
- Independent backend deployment
- Independent database lifecycle
- Future delivery automation
- Future cloud deployment

A microservices architecture is a suitable design.

</details>

<details>
<summary>Microservices Architecture</summary>

```text
+-------------------+
| frontend-service  |
+-------------------+
          |
          v
+-------------------+
| enrolment-service |
+-------------------+
          |
          v
+-------------------+
| database-service  |
+-------------------+
```

Responsibilities:

Frontend:

- HTMX
- HTML
- CSS

Backend:

- REST APIs
- Business Logic
- Prompt Loading
- Agent Integration

Database:

- SQLite or PostgreSQL
- Persistence

</details>

<details>
<summary>Project Structure</summary>

```text
enrolment-app-open-ai/
│
├── docker-compose.yml
├── agentic_loop/
│   ├── main.py                         # entrypoint + interactive menu
│   ├── config/
│   │   └── review_config.py           # mode->prompt roots, files, models
│   ├── core/
│   │   ├── orchestrator.py            # Observe -> Implement -> Review -> Summary
│   │   ├── prompt_registry.py         # strict prompt resolution + validation
│   │   ├── ai_runner.py               # OpenAI/Ollama chat runner
│   │   └── reporter.py                # console + optional evidence output
│   ├── collectors/
│   │   ├── db_collector.py            # DB evidence checks
│   │   ├── endpoints_collector.py     # endpoint/route evidence checks
│   │   └── architecture_collector.py  # compose/service boundary evidence
│   └── pipelines/
│       ├── db_pipeline.py             # DB mode flow
│       ├── endpoints_pipeline.py      # Endpoints mode flow
│       └── architecture_pipeline.py   # Architecture mode flow
│
├── frontend-service/
│   ├── Dockerfile
│   ├── css/
│   │   └── styles.css
│   └── templates/
│       ├── index.html
│       └── tabs/
│           ├── normal.html
│           └── ai-mode.html
│
├── enrolment-service/
│   ├── app.py
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── routes/
│   │   ├── ai_mode.py
│   │   ├── normal_ui.py
│   │   └── __init__.py
│   ├── services/
│   │   ├── database_api.py
│   │   ├── llm_client.py
│   │   ├── prompt_loader.py
│   │   └── __init__.py
│   └── views/
│       ├── html_formatters.py
│       └── __init__.py
│
├── database-service/
│   ├── app.py
│   ├── Dockerfile
│   ├── init_db.py
│   ├── requirements.txt
│   └── data/
│
├── prompts/
│   ├── service/
│   │   ├── implementation/
│   │   │   ├── system_prompt.txt
│   │   │   ├── task_prompt.txt
│   │   │   └── context_prompt.txt
│   │   └── review/
│   │
│   └── lab4/
│       ├── implementation/
│       │   ├── architecture_system_prompt.txt
│       │   ├── architecture_task_prompt.txt
│       │   ├── pattern_selection_prompt.txt
│       │   ├── adr_generation_prompt.txt
│       │   └── agent_task_prompt.txt
│       └── review/
│           ├── adr_review_prompt.txt
│           └── agent_review_prompt.txt
│
└── legacy-lab3/
    ├── app.py
    ├── init_db.py
    ├── requirements.txt
    ├── css/
    │   └── styles.css
    ├── templates/
    │   └── index.html
    └── enrolment.db
```

</details>

<details>
<summary>Create Project Workspace</summary>

Use the existing `enrolment-app-open-ai` created in Labs 01-03.

```bash
# 1) Go to app root
cd enrolment-app-open-ai

# 2) Root
touch docker-compose.yml

# 3) Frontend Service
mkdir -p frontend-service/templates frontend-service/templates/tabs frontend-service/css
touch frontend-service/Dockerfile \
      frontend-service/templates/index.html \
      frontend-service/templates/tabs/normal.html \
      frontend-service/templates/tabs/ai-mode.html \
      frontend-service/css/styles.css

# 4) Enrolment Service
mkdir -p enrolment-service/routes enrolment-service/services enrolment-service/views
touch enrolment-service/app.py \
      enrolment-service/Dockerfile \
      enrolment-service/requirements.txt \
      enrolment-service/routes/ai_mode.py \
      enrolment-service/routes/normal_ui.py \
      enrolment-service/routes/__init__.py \
      enrolment-service/services/database_api.py \
      enrolment-service/services/llm_client.py \
      enrolment-service/services/prompt_loader.py \
      enrolment-service/services/__init__.py \
      enrolment-service/views/html_formatters.py \
      enrolment-service/views/__init__.py

# 5) Database Service
mkdir -p database-service/data
touch database-service/Dockerfile \
      database-service/app.py \
      database-service/requirements.txt \
      database-service/init_db.py

# 6) Agentic Loop
mkdir -p agentic_loop/config agentic_loop/core agentic_loop/collectors agentic_loop/pipelines
touch agentic_loop.py \
      agentic_loop/main.py \
      agentic_loop/__init__.py \
      agentic_loop/config/review_config.py \
      agentic_loop/config/__init__.py \
      agentic_loop/core/orchestrator.py \
      agentic_loop/core/prompt_registry.py \
      agentic_loop/core/ai_runner.py \
      agentic_loop/core/reporter.py \
      agentic_loop/core/__init__.py \
      agentic_loop/collectors/db_collector.py \
      agentic_loop/collectors/endpoints_collector.py \
      agentic_loop/collectors/architecture_collector.py \
      agentic_loop/collectors/__init__.py \
      agentic_loop/pipelines/db_pipeline.py \
      agentic_loop/pipelines/endpoints_pipeline.py \
      agentic_loop/pipelines/architecture_pipeline.py \
      agentic_loop/pipelines/__init__.py
```

</details>

<details>
<summary>Prompts Setup</summary>

Lab 4 uses two prompt hierarchies:

1. **Service prompts** (`prompts/service/`) for the HTMX + Flask service AI flows.
2. **Lab 4 prompts** (`prompts/lab4/`) for architecture and ADR analysis.

```bash
# Run from app root: enrolment-app-open-ai

# Service prompts used by HTMX + Flask AI flows
mkdir -p prompts/service/implementation
mkdir -p prompts/service/review

touch prompts/service/implementation/system_prompt.txt
touch prompts/service/implementation/task_prompt.txt
touch prompts/service/implementation/context_prompt.txt

# Lab 4 implementation prompts
mkdir -p prompts/lab4/implementation

touch prompts/lab4/implementation/architecture_system_prompt.txt
touch prompts/lab4/implementation/architecture_task_prompt.txt
touch prompts/lab4/implementation/pattern_selection_prompt.txt
touch prompts/lab4/implementation/adr_generation_prompt.txt
touch prompts/lab4/implementation/agent_task_prompt.txt

# Lab 4 review prompts
mkdir -p prompts/lab4/review

touch prompts/lab4/review/adr_review_prompt.txt
touch prompts/lab4/review/agent_review_prompt.txt
```

Populate the prompt files with the following content.

**Service prompts:**

`prompts/service/implementation/system_prompt.txt`
```text
You are a PRECISION IMPLEMENTATION AGENT for the Flask Student Enrolment App.

Your role: Review live application behavior using real evidence from HTTP requests and database checks.

Use only supplied evidence from the running application. Follow the rules exactly. Do not invent requirements.

Rules:
- Do not invent new database fields.
- Do not invent new endpoints.
- Do not invent functionality.
- Do not modify endpoint contracts.
- Do not suggest new application features.
- Do not recommend a unique constraint on subject_code.
- Focus strictly on validation, error handling, response formatting, or testing.

Evidence source: Real HTTP requests to running Flask app + Real database queries.

If information is unavailable in the supplied evidence, say:

Information unavailable in supplied evidence.
```

`prompts/service/implementation/task_prompt.txt`
```text
Review Target: {{REVIEW_TARGET}}

The Flask app must be running. The evidence below comes from live HTTP requests and database validation.

Live Validation Evidence:
{{VALIDATION_EVIDENCE}}

Task:
Analyze the evidence above. Identify ONE precise, evidence-backed improvement for:
- Input validation
- Error handling
- Response formatting
- Edge case testing

If the evidence shows the app is not responding: Say "Unable to verify live behavior."
If no improvement is supported by the evidence: Say "No evidence-backed improvement identified."

Output Format:
- Return exactly two bullet points (Issue + Recommendation), or the no-evidence sentence.
- Maximum 60 words total.
- No explanations, no reasoning.
```

`prompts/service/implementation/context_prompt.txt`
```text
Application Name:
Student Enrolment App

Database Schema:
- student_id (integer, primary key)
- student_name (text, required)
- subject_code (text, required, NOT UNIQUE)

Important Domain Rule:
- subject_code is NOT unique.
- Multiple students may enrol in the same subject.
- Never recommend a unique constraint on subject_code.

Endpoints (ONLY these exist):

GET Endpoints (Read-Only):
- GET /students
- GET /students/{student_id}
- GET /students/by-id
- GET /students/by-subject

POST Endpoints (AI Queries):
- POST /ask
- POST /ask-with-context

Forbidden Operations:
Do not mention or suggest create, add, update, delete, or edit operations for students — they are not implemented in this application.
```

**Lab 4 implementation prompts:**

`prompts/lab4/implementation/architecture_system_prompt.txt`
```text
You are a cautious software architecture review assistant.
Use only the evidence and prompts provided. Do not invent services, APIs, ownership, or dependencies.
Focus on service boundaries, API ownership, database ownership, deployment independence, and ADR quality.
If the evidence is missing or ambiguous, say so explicitly and label the finding as incomplete rather than speculative.
Prefer concise, evidence-based observations over broad architectural opinions.
Reply in at most 30 words.
```

`prompts/lab4/implementation/architecture_task_prompt.txt`
```text
Review the proposed microservices architecture for frontend-service, enrolment-service, and database-service.
Use the evidence below to identify:
- responsibilities per service
- ownership boundaries
- dependencies and coupling
- missing or overlapping responsibilities
Distinguish clearly between observed evidence and assumptions.
Provide a concise review and at most one or two practical recommendations.
If something cannot be assessed from the evidence, mention that limitation explicitly.
Reply in at most 30 words.
```

`prompts/lab4/implementation/pattern_selection_prompt.txt`
```text
Compare the following architecture styles:

- Monolith
- Layered Architecture
- Microservices

Evaluate each using:

- scalability
- maintainability
- deployment complexity
- operational complexity
- team ownership
- future growth

Recommend one architecture style for the Student Enrolment System.

Return exactly:

Architecture:
Reason:
Trade-Offs:
```

`prompts/lab4/implementation/adr_generation_prompt.txt`
```text
You are a careful Architecture Decision Record (ADR) writer.
Using the architecture review evidence and the selected architecture choice, create a concise ADR with these sections:
- Context
- Decision
- Alternatives
- Trade-offs
- Consequences
Ground every section in the supplied evidence. Do not invent requirements, constraints, or future outcomes.
If evidence is incomplete, state that clearly and avoid over-claiming.
Keep the ADR architecture-focused, practical, and easy to review.
Reply in at most 30 words.
```

`prompts/lab4/implementation/agent_task_prompt.txt`
```text
You are a cautious software implementation assistant.
Use only the evidence provided. Do not invent missing facts or assume unverified behavior.

Based on the validation evidence, recommend exactly one implementation improvement that improves reliability, maintainability, or user experience.
State why it is needed, what evidence supports it, and what risk or trade-off it may introduce.
Do not suggest multiple unrelated changes. If the evidence is insufficient, say so explicitly and propose the minimum next check.
Keep the answer concise and focused on the most actionable improvement.
Reply in at most 30 words.
```

**Lab 4 review prompts:**

`prompts/lab4/review/adr_review_prompt.txt`
```text
You are a concise ADR reviewer.
Review the ADR using only the supplied evidence.
Flag one weakness or missing point.
Suggest one short improvement.
Reply in at most 30 words.
```

`prompts/lab4/review/agent_review_prompt.txt`
```text
You are a critical review assistant.
Review the implementation recommendation against the evidence and provide a short critique or improvement suggestion.
Be strict about evidence quality. If the recommendation is unsupported, say so clearly.
Highlight any assumptions, missing evidence, or risks.
Keep the review concise and actionable.
Reply in at most 30 words.
```

</details>

## 4. Application Setup and Development

### Frontend Service (HTMX)

<details>
<summary>frontend-service description</summary>

The frontend service is a static Nginx container responsible for:

* HTML
* modular tab composition (iframe-based tab shell)
* HTMX in tab pages
* CSS
* User interaction

All requests are sent to the `enrolment-service`.

</details>

<details>
<summary>frontend-service/Dockerfile</summary>

```dockerfile
FROM nginx:alpine

COPY templates/ /usr/share/nginx/html/
COPY css/ /usr/share/nginx/html/css/

EXPOSE 80
```

</details>

<details>
<summary>frontend-service/templates/index.html</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Student Enrolment App</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
<main class="app-shell">
    <header class="app-header">
        <h1>Student Enrolment App</h1>
        <p>HTMX UI: Normal UI and AI Mode</p>
    </header>

    <section class="tab-shell card">
        <nav class="tab-nav" aria-label="Application tabs">
            <button class="tab-btn is-active" data-tab="normal" type="button">Normal UI</button>
            <button class="tab-btn" data-tab="ai-mode" type="button">AI Mode</button>
        </nav>

        <div class="tab-content">
            <iframe id="tab-frame-normal" class="tab-frame is-active" src="tabs/normal.html" title="Normal UI tab"></iframe>
            <iframe id="tab-frame-ai-mode" class="tab-frame" src="tabs/ai-mode.html" title="AI Mode tab"></iframe>
        </div>
    </section>
</main>

<script>
const tabButtons = Array.from(document.querySelectorAll(".tab-btn"));
const tabFrames = {
    normal: document.getElementById("tab-frame-normal"),
    "ai-mode": document.getElementById("tab-frame-ai-mode"),
};

function activateTab(tabName) {
    tabButtons.forEach((button) => {
        button.classList.toggle("is-active", button.dataset.tab === tabName);
    });

    Object.entries(tabFrames).forEach(([name, frame]) => {
        frame.classList.toggle("is-active", name === tabName);
    });

    window.location.hash = tabName;
}

tabButtons.forEach((button) => {
    if (button.disabled) {
        return;
    }

    button.addEventListener("click", () => {
        activateTab(button.dataset.tab);
    });
});

const hashTab = window.location.hash.replace("#", "");
if (hashTab && tabFrames[hashTab]) {
    activateTab(hashTab);
} else {
    activateTab("normal");
}
</script>

</body>
</html>
```

</details>

<details>
<summary>frontend-service/templates/tabs/normal.html</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Normal UI</title>
    <link rel="stylesheet" href="/css/styles.css">
    <style>
        body { margin: 0; min-height: auto; background: transparent; }
        .app-shell { max-width: none; padding: 0; }
    </style>
</head>
<body>
<main class="app-shell">
    <section class="card">
        <h2>Students</h2>

        <button id="toggle-students-btn" type="button">
            Show All Students
        </button>

        <div id="students-result" class="panel is-hidden"></div>

        <h2>Find Student by ID</h2>

        <form id="student-by-id-form">
            <label for="student_id">Student ID</label>
            <input id="student_id" name="student_id" type="number" min="1" placeholder="Enter student ID">
            <button type="submit">Get Student</button>
        </form>

        <div id="student-result" class="panel"></div>

        <h2>Find Students by Subject Code</h2>

        <form id="students-by-subject-form">
            <label for="subject_code">Subject Code</label>
            <input id="subject_code" name="subject_code" type="text" placeholder="Example: ASD101">
            <button type="submit">Find Students</button>
        </form>

        <div id="subject-result" class="panel"></div>
    </section>
</main>

<script>
const toggleStudentsBtn = document.getElementById("toggle-students-btn");
const studentsPanel = document.getElementById("students-result");
const studentByIdForm = document.getElementById("student-by-id-form");
const studentsBySubjectForm = document.getElementById("students-by-subject-form");

async function renderIntoPanel(panelId, url, options = {}) {
    const panel = document.getElementById(panelId);

    try {
        const response = await fetch(url, options);
        const body = await response.text();
        panel.innerHTML = body;
    } catch (error) {
        panel.innerHTML = `<p>Request failed.</p><pre>${error}</pre>`;
    }
}

toggleStudentsBtn.addEventListener("click", () => {
    const isHidden = studentsPanel.classList.contains("is-hidden");

    if (isHidden) {
        studentsPanel.classList.remove("is-hidden");
        toggleStudentsBtn.textContent = "Hide All Students";

        if (!studentsPanel.dataset.loaded) {
            renderIntoPanel("students-result", "http://localhost:5001/students");
            studentsPanel.dataset.loaded = "true";
        }
    } else {
        studentsPanel.classList.add("is-hidden");
        toggleStudentsBtn.textContent = "Show All Students";
    }
});

studentByIdForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const formData = new FormData(studentByIdForm);
    const query = new URLSearchParams(formData).toString();
    renderIntoPanel("student-result", `http://localhost:5001/students/by-id?${query}`);
});

studentsBySubjectForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const formData = new FormData(studentsBySubjectForm);
    const query = new URLSearchParams(formData).toString();
    renderIntoPanel("subject-result", `http://localhost:5001/students/by-subject?${query}`);
});
</script>
</body>
</html>
```

</details>

<details>
<summary>frontend-service/templates/tabs/ai-mode.html</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>AI Mode</title>
    <link rel="stylesheet" href="/css/styles.css">
    <style>
        body { margin: 0; min-height: auto; background: transparent; }
        .app-shell { max-width: none; padding: 0; }
    </style>
</head>
<body>
<main class="app-shell">
    <section class="card">
        <h2>Ask Local AI Agent</h2>

        <form id="ask-form">
            <label for="question">Question</label>
            <textarea id="question" name="question" rows="5">Explain what this Student Enrolment App does in one short paragraph.</textarea>
            <button type="submit">Ask Local Agent</button>
        </form>

        <div id="agent-result" class="panel"></div>

        <h2>Ask With Context</h2>

        <form id="ask-with-context-form">
            <label for="context-question">Question</label>
            <textarea id="context-question" name="question" rows="4">Explain the Student Enrolment App.</textarea>
            <button type="submit">Ask With Context</button>
        </form>

        <div id="context-result" class="panel"></div>

        <h2>Architecture Review</h2>

        <form id="architecture-review-form">
            <label for="architecture_request">Request</label>
            <textarea id="architecture_request" name="architecture_request" rows="4">Review service boundaries for frontend-service, enrolment-service, and database-service.</textarea>
            <button type="submit">Run Architecture Review</button>
        </form>

        <div id="architecture-result" class="panel"></div>
    </section>
</main>

<script>
const askForm = document.getElementById("ask-form");
const askWithContextForm = document.getElementById("ask-with-context-form");
const architectureReviewForm = document.getElementById("architecture-review-form");

async function renderIntoPanel(panelId, url, options = {}) {
    const panel = document.getElementById(panelId);

    try {
        const response = await fetch(url, options);
        const body = await response.text();
        panel.innerHTML = body;
    } catch (error) {
        panel.innerHTML = `<p>Request failed.</p><pre>${error}</pre>`;
    }
}

askForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const formData = new URLSearchParams(new FormData(askForm));
    renderIntoPanel("agent-result", "http://localhost:5001/ask", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: formData,
    });
});

askWithContextForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const formData = new URLSearchParams(new FormData(askWithContextForm));
    renderIntoPanel("context-result", "http://localhost:5001/ask-with-context", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: formData,
    });
});

architectureReviewForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const formData = new URLSearchParams(new FormData(architectureReviewForm));
    renderIntoPanel("architecture-result", "http://localhost:5001/architecture-review", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: formData,
    });
});
</script>
</body>
</html>
```

</details>

<details>
<summary>frontend-service/css/styles.css</summary>

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
<summary>Notes</summary>

```text
Frontend Service Port:
80

Container Name:
frontend-service

Backend Service:
enrolment-service

Backend Port:
5001
```

The frontend service contains presentation only.

Business logic, AI integration, prompt loading, and architecture review processing remain inside the `enrolment-service`.

</details>

</details>


### Backend Service (Flask)

<details>
<summary>backend-service description</summary>

The `enrolment-service` is the backend API layer.

Responsibilities:

- Expose UI-mode routes through the `normal_ui` blueprint
- Expose AI-mode routes through the `ai_mode` blueprint
- Keep business logic and integrations in the service layer
- Call `database-service` through backend service modules
- Load service prompt files for context-aware requests
- Call Ollama for AI and ask-with-context tasks
- Return HTML fragments through the view layer

</details>

<details>
<summary>enrolment-service/Dockerfile</summary>

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY enrolment-service/requirements.txt ./requirements.txt

RUN pip install --no-cache-dir -r requirements.txt

COPY enrolment-service ./enrolment-service
COPY prompts/service ./prompts/service
COPY prompts/lab4 ./prompts/lab4

EXPOSE 5001

CMD ["python", "enrolment-service/app.py"]
```

</details>


<details>
<summary>enrolment-service/requirements.txt</summary>

```text
flask==3.0.3
flask-cors==4.0.1
requests==2.32.3
openai
python-dotenv
```

</details>

<details>
<summary>enrolment-service/app.py</summary>

```python
from pathlib import Path
import sys

from flask import Flask
from flask_cors import CORS


BASE_DIR = Path(__file__).resolve().parent
if str(BASE_DIR) not in sys.path:
    sys.path.insert(0, str(BASE_DIR))

from routes.ai_mode import ai_mode_bp
from routes.normal_ui import normal_ui_bp


def create_app():
    app = Flask(__name__)
    CORS(app)

    app.register_blueprint(normal_ui_bp)
    app.register_blueprint(ai_mode_bp)

    return app


app = create_app()


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5001, debug=True)
```

</details>

<details>
<summary>enrolment-service/routes/ai_mode.py</summary>

```python
from flask import Blueprint, request

from services.llm_client import OLLAMA_MODEL, call_architecture_agent, create_chat_completion
from services.prompt_loader import load_prompt


ai_mode_bp = Blueprint("ai_mode", __name__)


@ai_mode_bp.post("/ask")
def ask_local_agent():
    question = request.form.get("question", "").strip()

    if not question:
        return "<p>Question is required.</p>", 400

    try:
        answer = create_chat_completion(
            [
                {
                    "role": "system",
                    "content": (
                        "You are a concise software engineering assistant. "
                        "Answer in one short paragraph unless asked otherwise."
                    ),
                },
                {"role": "user", "content": question},
            ],
            max_tokens=200,
            temperature=0.2,
            model=OLLAMA_MODEL,
        )
        return f"<p>{answer}</p>", 200
    except Exception as exc:
        return (
            "<p>Local AI agent request failed. "
            "Check that Ollama is running and that qwen2.5:0.5b is installed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


@ai_mode_bp.post("/ask-with-context")
def ask_with_context():
    question = request.form.get("question", "").strip()

    if not question:
        return "<p>Question is required.</p>", 400

    try:
        system_prompt = load_prompt("service/implementation/system_prompt.txt")
        task_prompt = load_prompt("service/implementation/task_prompt.txt")
        context_prompt = load_prompt("service/implementation/context_prompt.txt")

        final_prompt = f"""
{task_prompt}

{context_prompt}

User Question:

{question}
"""

        answer = create_chat_completion(
            [
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": final_prompt},
            ],
            max_tokens=300,
            temperature=0.2,
            model=OLLAMA_MODEL,
        )
        return f"<p>{answer}</p>", 200
    except Exception as exc:
        return (
            "<p>Context-aware request failed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


@ai_mode_bp.post("/pattern-selection")
def pattern_selection():
    architecture_request = request.form.get("architecture_request", "").strip()

    if not architecture_request:
        return "<p>Architecture request is required.</p>", 400

    try:
        answer = call_architecture_agent(
            "architecture_system_prompt.txt",
            "pattern_selection_prompt.txt",
            architecture_request,
        )
        return f"<pre>{answer}</pre>", 200
    except Exception as exc:
        return (
            "<p>Pattern selection request failed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


@ai_mode_bp.post("/architecture-review")
def architecture_review():
    architecture_request = request.form.get("architecture_request", "").strip()

    if not architecture_request:
        return "<p>Architecture request is required.</p>", 400

    try:
        answer = call_architecture_agent(
            "architecture_system_prompt.txt",
            "architecture_task_prompt.txt",
            architecture_request,
        )
        return f"<pre>{answer}</pre>", 200
    except Exception as exc:
        return (
            "<p>Architecture review request failed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


@ai_mode_bp.post("/adr-review")
def adr_review():
    architecture_request = request.form.get("architecture_request", "").strip()

    if not architecture_request:
        return "<p>ADR text is required.</p>", 400

    try:
        answer = call_architecture_agent(
            "architecture_system_prompt.txt",
            "adr_review_prompt.txt",
            architecture_request,
        )
        return f"<pre>{answer}</pre>", 200
    except Exception as exc:
        return (
            "<p>ADR review request failed.</p>"
            f"<pre>{exc}</pre>",
            503,
        )
```

</details>

<details>
<summary>enrolment-service/routes/normal_ui.py</summary>

```python
from flask import Blueprint, request
import requests

from services.database_api import get_student_by_id_response, get_students, get_students_by_subject_response
from views.html_formatters import format_student_html, format_students_html


normal_ui_bp = Blueprint("normal_ui", __name__)


@normal_ui_bp.get("/")
def health():
    return "<p>enrolment-service running</p>", 200


@normal_ui_bp.get("/students")
def get_students_route():
    try:
        return format_students_html(get_students()), 200
    except requests.RequestException as exc:
        return (
            "<p>Failed to retrieve students from database-service.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


@normal_ui_bp.get("/students/by-id")
def get_student_by_id():
    student_id = request.args.get("student_id", "").strip()

    if not student_id:
        return "<p>Student ID is required.</p>", 400

    try:
        response = get_student_by_id_response(student_id)

        if response.status_code == 404:
            return "<p>Student not found.</p>", 404
        if response.status_code == 400:
            return "<p>Student ID must be valid.</p>", 400

        response.raise_for_status()
        return format_student_html(response.json()), 200
    except requests.RequestException as exc:
        return (
            "<p>Failed to retrieve student from database-service.</p>"
            f"<pre>{exc}</pre>",
            503,
        )


@normal_ui_bp.get("/students/by-subject")
def get_students_by_subject():
    subject_code = request.args.get("subject_code", "").strip().upper()

    if not subject_code:
        return "<p>Subject code is required.</p>", 400

    try:
        response = get_students_by_subject_response(subject_code)

        if response.status_code == 404:
            return f"<p>No students found for {subject_code}.</p>", 404

        response.raise_for_status()
        return format_students_html(response.json()), 200
    except requests.RequestException as exc:
        return (
            "<p>Failed to retrieve subject results from database-service.</p>"
            f"<pre>{exc}</pre>",
            503,
        )
```

</details>

<details>
<summary>enrolment-service/services/database_api.py</summary>

```python
import os

import requests


DATABASE_SERVICE_URL = os.getenv("DATABASE_SERVICE_URL", "http://database-service:5002")


def get_students():
    response = requests.get(f"{DATABASE_SERVICE_URL}/students", timeout=5)
    response.raise_for_status()
    return response.json()


def get_student_by_id_response(student_id):
    return requests.get(f"{DATABASE_SERVICE_URL}/students/{student_id}", timeout=5)


def get_students_by_subject_response(subject_code):
    return requests.get(
        f"{DATABASE_SERVICE_URL}/students/by-subject",
        params={"subject_code": subject_code},
        timeout=5,
    )
```

</details>

<details>
<summary>enrolment-service/services/llm_client.py</summary>

```python
import os

from openai import OpenAI

from services.prompt_loader import load_lab4_prompt


OLLAMA_BASE_URL = os.getenv("OLLAMA_BASE_URL", "http://host.docker.internal:11434/v1")
OLLAMA_MODEL = os.getenv("OLLAMA_MODEL", "qwen2.5:0.5b")

client = OpenAI(base_url=OLLAMA_BASE_URL, api_key="ollama")


def create_chat_completion(messages, max_tokens=300, temperature=0.2, model=None):
    response = client.chat.completions.create(
        model=model or OLLAMA_MODEL,
        messages=messages,
        max_tokens=max_tokens,
        temperature=temperature,
    )
    return response.choices[0].message.content


def call_architecture_agent(system_prompt_file, task_prompt_file, user_input, max_tokens=300):
    system_prompt = load_lab4_prompt(system_prompt_file)
    task_prompt = load_lab4_prompt(task_prompt_file)

    final_prompt = f"""
{task_prompt}

User Input:

{user_input}
"""

    answer = create_chat_completion(
        [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": final_prompt},
        ],
        max_tokens=max_tokens,
        temperature=0.2,
    )
    return answer.strip()
```

</details>

<details>
<summary>enrolment-service/services/prompt_loader.py</summary>

```python
from pathlib import Path


BASE_DIR = Path(__file__).resolve().parent.parent
APP_DIR = BASE_DIR.parent
PROMPT_DIR = APP_DIR / "prompts"
LAB4_IMPLEMENTATION_PROMPT_DIR = PROMPT_DIR / "lab4" / "implementation"
LAB4_REVIEW_PROMPT_DIR = PROMPT_DIR / "lab4" / "review"


def load_prompt(filename):
    return (PROMPT_DIR / filename).read_text(encoding="utf-8").strip()


def load_lab4_prompt(filename):
    prompt_dirs = [LAB4_IMPLEMENTATION_PROMPT_DIR, LAB4_REVIEW_PROMPT_DIR]

    for prompt_dir in prompt_dirs:
        candidate = prompt_dir / filename
        if candidate.exists():
            return candidate.read_text(encoding="utf-8").strip()

    return (LAB4_IMPLEMENTATION_PROMPT_DIR / filename).read_text(encoding="utf-8").strip()
```

</details>

<details>
<summary>enrolment-service/views/html_formatters.py</summary>

```python
def format_students_html(students):
    if not students:
        return "<p>No students found.</p>"

    html = "<ul>"
    for student in students:
        html += (
            f"<li>{student['student_id']} - "
            f"{student['student_name']} - {student['subject_code']}</li>"
        )
    html += "</ul>"
    return html


def format_student_html(student):
    return (
        f"<p>ID: {student['student_id']}<br>"
        f"Name: {student['student_name']}<br>"
        f"Subject: {student['subject_code']}</p>"
    )
```

</details>

<details>
<summary>Notes</summary>

```text
Backend Service Port:
5001

Container Name:
enrolment-service

Database Service:
database-service

Database Port:
5002
```

The backend service contains business logic, AI integration, prompt loading, context-aware review, and ADR architecture review through backend routes using prompt assets packaged in the backend container app.

</details>

### Database Service

<details>
<summary>database-service description</summary>

The database service owns all student data.

Responsibilities:

- Database creation
- Database seeding
- Data persistence
- Student data APIs

The database service does not:

- Render HTML
- Load prompts
- Call AI models
- Implement business logic

</details>

<details>
<summary>database-service/Dockerfile</summary>

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .
COPY init_db.py .

RUN python init_db.py

EXPOSE 5002

CMD ["python", "app.py"]
```

</details>

<details>
<summary>database-service/requirements.txt</summary>

```text
flask==3.0.3
```

</details>


<details>
<summary>database-service/init_db.py</summary>

```python
import os
import sqlite3

DATA_DIR = "/app/data"
DATABASE_NAME = os.path.join(DATA_DIR, "enrolment.db")

os.makedirs(DATA_DIR, exist_ok=True)

conn = sqlite3.connect(
    DATABASE_NAME
)

cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS students (
    student_id INTEGER PRIMARY KEY,
    student_name TEXT NOT NULL,
    subject_code TEXT NOT NULL
)
""")

cursor.execute(
    "DELETE FROM students"
)

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
    (10, "Chloe Young", "SEC301"),
]

cursor.executemany(
    """
    INSERT INTO students (
        student_id,
        student_name,
        subject_code
    )
    VALUES (?, ?, ?)
    """,
    students
)

conn.commit()
conn.close()

print(
    "Database initialized with 10 students."
)
```

</details>


<details>
<summary>database-service/app.py</summary>

```python
from flask import Flask, jsonify, request
import sqlite3

app = Flask(__name__)

DATABASE_NAME = "/app/data/enrolment.db"

def get_db_connection():
    conn = sqlite3.connect(DATABASE_NAME)
    conn.row_factory = sqlite3.Row
    return conn

@app.get("/")
def health():
    return jsonify({"service": "database-service", "status": "running"})

@app.get("/students")
def get_students():
    conn = get_db_connection()
    students = conn.execute(
        "SELECT student_id, student_name, subject_code FROM students"
    ).fetchall()
    conn.close()
    return jsonify([dict(row) for row in students])

@app.get("/students/<int:student_id>")
def get_student(student_id):
    conn = get_db_connection()
    student = conn.execute(
        "SELECT student_id, student_name, subject_code FROM students WHERE student_id = ?",
        (student_id,),
    ).fetchone()
    conn.close()

    if student is None:
        return jsonify({"error": "Student not found"}), 404

    return jsonify(dict(student))

@app.get("/students/by-subject")
def get_students_by_subject():
    subject_code = request.args.get("subject_code", "").strip().upper()

    if not subject_code:
        return jsonify({"error": "subject_code required"}), 400

    conn = get_db_connection()
    students = conn.execute(
        "SELECT student_id, student_name, subject_code FROM students WHERE subject_code = ?",
        (subject_code,),
    ).fetchall()
    conn.close()

    if not students:
        return jsonify({"error": "No students found"}), 404

    return jsonify([dict(row) for row in students])

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5002, debug=True)
```

</details>


###  Docker Compose Architecture

<details>
<summary>docker-compose.yml</summary>

```yaml
services:
    frontend-service:
        build:
            context: ./frontend-service
        container_name: frontend-service
        ports:
            - "8080:80"
        depends_on:
            - enrolment-service
        networks:
            - enrolment-network
        restart: unless-stopped

    enrolment-service:
        build:
            context: .
            dockerfile: enrolment-service/Dockerfile
        container_name: enrolment-service
        ports:
            - "5001:5001"
        environment:
            DATABASE_SERVICE_URL: http://database-service:5002
            OLLAMA_BASE_URL: http://host.docker.internal:11434/v1
            OLLAMA_MODEL: qwen2.5:0.5b
        extra_hosts:
            - "host.docker.internal:host-gateway"
        depends_on:
            - database-service
        networks:
            - enrolment-network
        restart: unless-stopped

    database-service:
        build:
            context: ./database-service
        container_name: database-service
        ports:
            - "5002:5002"
        volumes:
            - database_data:/app/data
        networks:
            enrolment-network:
                aliases:
                    - database-service
        restart: unless-stopped

volumes:
    database_data:

networks:
    enrolment-network:
        driver: bridge
```

</details>

<details>
<summary>Architecture Flow</summary>

```text
Browser
    │
    ▼
frontend-service
(Nginx)
Port 8080
    │
    ▼
enrolment-service
(Flask + Ollama)
Port 5001
    │
    ▼
database-service
(Flask + SQLite)
Port 5002
```
</details>

<details>
<summary>Service Communication</summary>

```text
frontend-service
    ↓ HTTP
enrolment-service

enrolment-service
    ↓ HTTP
database-service

database-service
    ↓
SQLite Database
```
</details>

<details>
<summary>Run Steps</summary>

1. Install

Linux (Ubuntu):

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

Windows PowerShell (Admin):

```powershell
winget install -e --id Docker.DockerDesktop
```

2. Build and run

Linux and Windows PowerShell:

```bash
docker-compose up --build
```

3. Check status

Linux and Windows PowerShell:

```bash
docker-compose ps
```

4. Open app

Linux:

```bash
xdg-open http://localhost:8080
```

Windows PowerShell:

```powershell
Start-Process http://localhost:8080
```

5. Enable AI connection to containers

Linux:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d && printf "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0:11434\"\n" | sudo tee /etc/systemd/system/ollama.service.d/override.conf >/dev/null && sudo systemctl daemon-reload && sudo systemctl restart ollama
```

Windows PowerShell:

```powershell
setx OLLAMA_HOST "0.0.0.0:11434"
```

Close and reopen Ollama after `setx`.

6. Verify container access to Ollama

Linux and Windows PowerShell:

```bash
docker-compose exec enrolment-service python -c "import requests; print(requests.get('http://host.docker.internal:11434/api/tags', timeout=5).status_code)"
```

Expected: `200`

7. Test AI endpoint

Linux and Windows PowerShell:

```bash
curl -X POST http://localhost:5001/ask -d "question=Say hello in one sentence"
```

8. Stop services

Linux and Windows PowerShell:

```bash
docker-compose down
```

9. Remove stopped containers

Linux and Windows PowerShell:

```bash
docker-compose rm -f
```

10. View logs

Linux and Windows PowerShell:

```bash
docker-compose logs -f
```

11. Rebuild containers

Linux and Windows PowerShell:

```bash
docker-compose up --build --force-recreate
```
</details>

---

## 5. Application Testing

<details>
<summary>Endpoints Testing</summary>

| Function | Browser Validation | API Validation |
|---|---|---|
| Home Page | `http://localhost:8080` | N/A |
| View Students | Use "Show All Students" on `http://localhost:8080` | `curl http://localhost:5001/students` |
| Get Student by ID | Use "Find Student by ID" on `http://localhost:8080` | `curl "http://localhost:5001/students/by-id?student_id=1"` |
| Student Not Found | Use "Find Student by ID" with invalid ID on `http://localhost:8080` | `curl "http://localhost:5001/students/by-id?student_id=9999"` |
| Search by Subject | Use "Find Students by Subject Code" on `http://localhost:8080` | `curl "http://localhost:5001/students/by-subject?subject_code=ASD101"` |
| Subject Not Found | Use "Find Students by Subject Code" with unknown code on `http://localhost:8080` | `curl "http://localhost:5001/students/by-subject?subject_code=ABC999"` |
| AI Assistant | Use "Ask Local AI Agent" on `http://localhost:8080` | `curl -X POST http://localhost:5001/ask -d "question=Test"` |
| Context Assistant | Use "Ask With Context" on `http://localhost:8080` | `curl -X POST http://localhost:5001/ask-with-context -d "question=What endpoints exist?"` |
| Architecture Review | Use "Run Architecture Review" in the AI-mode tab on `http://localhost:8080` | `curl -X POST http://localhost:5001/architecture-review -d "architecture_request=Review service boundaries for frontend-service, enrolment-service, and database-service."` |
| Pattern Selection | API validation only (no dedicated UI button in Lab 04 page) | `curl -X POST http://localhost:5001/pattern-selection -d "architecture_request=Select the most suitable architecture pattern for the Student Enrolment System."` |
| ADR Review | API validation only (no dedicated UI button in Lab 04 page) | `curl -X POST http://localhost:5001/adr-review -d "architecture_request=Review this ADR draft for quality and missing points."` |

Expected: all listed browser and API checks return valid responses.

</details>

<details>
<summary>NFR Validation</summary>

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
    "http://localhost:5001/students/by-subject?subject_code=ASD101"
done
```

**Windows PowerShell**

```powershell
1..20 | ForEach-Object {
    curl.exe -s -o NUL -w "%{time_total}`n" `
    "http://localhost:5001/students/by-subject?subject_code=ASD101"
}
```

Record the result in the evidence log.

</details>


---

## 6. Architecture Analysis Activities

<details>
<summary>Configure Environment Variables</summary>

Update the `.env` file in the app root to include the Flask base URL for live endpoint testing.

**Location:** `enrolment-app-open-ai/.env`

Add the following line:

```bash
FLASK_BASE_URL=http://localhost:5001
```

**Complete .env file:**

```bash
OLLAMA_BASE_URL=http://localhost:11434/v1
OLLAMA_MODEL=qwen2.5:0.5b
OLLAMA_REVIEW_MODEL=llama3.1:8b
FLASK_BASE_URL=http://localhost:5001
```

**Purpose:**

- `FLASK_BASE_URL`: Base URL for the enrolment-service Flask app
- Used by `endpoints_collector.py` to make live HTTP requests to running endpoints
- Defaults to `http://localhost:5001` if not set

**Note:** The enrolment-service runs on port 5001 by default. If you change the port in `enrolment-service/app.py`, update this URL accordingly.

</details>

<details>
<summary>Update Agentic loop</summary>

Use the app-root modular agentic loop with an interactive menu for:

- DB
- Endpoints
- Architecture
- Run All
- Exit

The loop must load prompts from app-root prompt folders only:

- `enrolment-app-open-ai/prompts/service`
- `enrolment-app-open-ai/prompts/lab4`

### Source code per file

<details>
<summary>enrolment-app-open-ai/agentic_loop.py</summary>

```python
from pathlib import Path
import sys


ENGINE_DIR = Path(__file__).resolve().parent / "agentic_loop"
if str(ENGINE_DIR) not in sys.path:
    sys.path.insert(0, str(ENGINE_DIR))

from main import main


if __name__ == "__main__":
    main()
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/main.py</summary>

```python
from pathlib import Path

from dotenv import load_dotenv

from config.review_config import build_mode_config
from core.ai_runner import AIRunner
from core.orchestrator import run_mode
from core.prompt_registry import PromptRegistry
from core.reporter import print_menu, print_prompt_map, print_result


def _resolve_roots() -> tuple[Path, Path]:
    module_dir = Path(__file__).resolve().parent
    app_dir = module_dir.parent
    repo_root = app_dir.parent
    return app_dir, repo_root


def _menu_choice_to_key(choice: str) -> str | None:
    return {
        "1": "db",
        "2": "endpoints",
        "3": "architecture",
    }.get(choice)


def _print_mode_mapping(app_dir: Path) -> None:
    prompt_map = {
        "DB": app_dir / "prompts" / "service",
        "Endpoints": app_dir / "prompts" / "service",
        "Architecture": app_dir / "prompts" / "lab4",
    }
    print_prompt_map({key: str(path) for key, path in prompt_map.items()})


def main() -> None:
    app_dir, repo_root = _resolve_roots()
    load_dotenv(dotenv_path=app_dir / ".env")

    mode_config = build_mode_config()
    prompts = PromptRegistry(app_dir)
    ai = AIRunner()

    print("AGENTIC LOOP (MODULAR)")
    _print_mode_mapping(app_dir)

    while True:
        print_menu()
        choice = input("Choose a review target: ").strip()

        if choice == "0":
            print("Loop closed.")
            break

        if choice == "4":
            for key in ("db", "endpoints", "architecture"):
                result = run_mode(mode_config[key], app_dir, repo_root, prompts, ai)
                print_result(mode_config[key].label, result)
            continue

        mode_key = _menu_choice_to_key(choice)
        if not mode_key:
            print("Invalid choice. Select 0, 1, 2, 3, or 4.")
            continue

        result = run_mode(mode_config[mode_key], app_dir, repo_root, prompts, ai)
        print_result(mode_config[mode_key].label, result)
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/config/review_config.py</summary>

```python
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class ModeConfig:
    key: str
    label: str
    prompt_family: str
    implementation_prompts: tuple[str, ...]
    review_prompts: tuple[str, ...] = ()


def build_mode_config() -> dict[str, ModeConfig]:
    return {
        "db": ModeConfig(
            key="db",
            label="DB",
            prompt_family="service",
            implementation_prompts=(
                "implementation/system_prompt.txt",
                "implementation/task_prompt.txt",
                "implementation/context_prompt.txt",
            ),
        ),
        "endpoints": ModeConfig(
            key="endpoints",
            label="Endpoints",
            prompt_family="service",
            implementation_prompts=(
                "implementation/system_prompt.txt",
                "implementation/task_prompt.txt",
                "implementation/context_prompt.txt",
            ),
        ),
        "architecture": ModeConfig(
            key="architecture",
            label="Architecture",
            prompt_family="lab4",
            implementation_prompts=(
                "implementation/architecture_system_prompt.txt",
                "implementation/architecture_task_prompt.txt",
            ),
            review_prompts=("review/agent_review_prompt.txt",),
        ),
    }


def prompts_root(app_dir: Path) -> Path:
    return app_dir / "prompts"
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/core/orchestrator.py</summary>

```python
from pathlib import Path

from collectors import architecture_collector, db_collector, endpoints_collector
from config.review_config import ModeConfig
from core.ai_runner import AIRunner
from core.prompt_registry import PromptRegistry
from pipelines import architecture_pipeline, db_pipeline, endpoints_pipeline


COLLECTORS = {
    "db": db_collector.collect,
    "endpoints": endpoints_collector.collect,
    "architecture": architecture_collector.collect,
}


def _stage(mode_label: str, step: str, message: str) -> None:
    print(f"[{mode_label}][{step}] {message}")


def run_mode(mode: ModeConfig, app_dir: Path, repo_root: Path, prompts: PromptRegistry, ai: AIRunner) -> str:
    _stage(mode.label, "START", "Starting review flow")
    _stage(mode.label, "OBSERVE", "Collecting evidence")
    collector = COLLECTORS[mode.key]
    ok, evidence = collector(app_dir, repo_root)
    if not ok:
        _stage(mode.label, "OBSERVE", "Failed")
        return f"OBSERVE FAILED: {evidence}"
    _stage(mode.label, "OBSERVE", "Complete")

    if mode.key in {"db", "endpoints"}:
        _stage(mode.label, "PROMPTS", f"Loading prompt family: {mode.prompt_family}")
        system_prompt = prompts.read(mode.prompt_family, mode.implementation_prompts[0])
        task_prompt = prompts.read(mode.prompt_family, mode.implementation_prompts[1])
        context_prompt = prompts.read(mode.prompt_family, mode.implementation_prompts[2])
        _stage(mode.label, "PROMPTS", "Loaded implementation prompt set")

        if mode.key == "db":
            user_prompt = db_pipeline.build_user_prompt(task_prompt, context_prompt, evidence)
        else:
            user_prompt = endpoints_pipeline.build_user_prompt(task_prompt, context_prompt, evidence)

        _stage(mode.label, "LLM", "Running implementation model")
        output, err = ai.call(system_prompt, user_prompt, review=False)
        if err:
            _stage(mode.label, "LLM", "Failed")
            return f"MODEL FAILED: {err}"
        _stage(mode.label, "LLM", "Complete")
        _stage(mode.label, "DONE", "Review complete")
        return f"OBSERVE: {evidence}\n\nREVIEW: {output}"

    if mode.key == "architecture":
        _stage(mode.label, "PROMPTS", f"Loading prompt family: {mode.prompt_family}")
        system_prompt = prompts.read(mode.prompt_family, mode.implementation_prompts[0])
        task_prompt = prompts.read(mode.prompt_family, mode.implementation_prompts[1])
        implementation_user_prompt = architecture_pipeline.build_implementation_prompt(task_prompt, evidence)
        _stage(mode.label, "PROMPTS", "Loaded architecture implementation prompts")

        _stage(mode.label, "LLM", "Running architecture model")
        implementation_output, err = ai.call(system_prompt, implementation_user_prompt, review=False)
        if err:
            _stage(mode.label, "LLM", "Failed")
            return f"MODEL FAILED: {err}"
        _stage(mode.label, "LLM", "Architecture model complete")

        review_system_prompt = prompts.read(mode.prompt_family, mode.review_prompts[0])
        review_user_prompt = architecture_pipeline.build_review_prompt(implementation_output, evidence)
        _stage(mode.label, "PROMPTS", "Loaded architecture review prompt")
        _stage(mode.label, "LLM", "Running review model")
        review_output, review_err = ai.call(review_system_prompt, review_user_prompt, review=True)
        if review_err:
            review_output = review_err
            _stage(mode.label, "LLM", "Review model failed")
        else:
            _stage(mode.label, "LLM", "Review model complete")

        _stage(mode.label, "DONE", "Review complete")

        return (
            f"OBSERVE: {evidence}\n\n"
            f"ARCHITECTURE: {implementation_output}\n"
            f"REVIEW: {review_output}"
        )

    return "Unknown mode."
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/core/prompt_registry.py</summary>

```python
from pathlib import Path


class PromptRegistry:
    def __init__(self, app_dir: Path):
        self.app_dir = app_dir
        self.root = app_dir / "prompts"

    def resolve(self, family: str, relative_file: str) -> Path:
        candidate = self.root / family / relative_file
        if not candidate.exists():
            rel = candidate.relative_to(self.app_dir)
            raise FileNotFoundError(f"Missing prompt file: {rel}")
        return candidate

    def read(self, family: str, relative_file: str) -> str:
        return self.resolve(family, relative_file).read_text(encoding="utf-8").strip()

    def family_path(self, family: str) -> Path:
        return self.root / family
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/core/ai_runner.py</summary>

```python
import os

from openai import OpenAI


def _truncate_words(text: str, limit: int = 45) -> str:
    words = " ".join(text.split()).split()
    if len(words) <= limit:
        return " ".join(words)
    return " ".join(words[:limit]) + " ..."


class AIRunner:
    def __init__(self):
        self.base_url = os.getenv("OLLAMA_BASE_URL", "http://localhost:11434/v1")
        self.implementation_model = os.getenv("OLLAMA_MODEL", "qwen2.5:0.5b")
        self.review_model = os.getenv("OLLAMA_REVIEW_MODEL", "llama3.1:8b")
        self.client = OpenAI(base_url=self.base_url, api_key="ollama", timeout=180.0)

    def call(self, system_prompt: str, user_prompt: str, *, review: bool = False, max_tokens: int = 180) -> tuple[str | None, str | None]:
        model_name = self.review_model if review else self.implementation_model
        try:
            response = self.client.chat.completions.create(
                model=model_name,
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt},
                ],
                max_tokens=max_tokens,
                temperature=0.1,
            )
            content = (response.choices[0].message.content or "").strip()
            if not content:
                return "No response generated.", None
            return _truncate_words(content), None
        except Exception as exc:
            return None, f"Model call failed ({model_name}): {exc}"
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/core/reporter.py</summary>

```python
def print_prompt_map(mapping: dict[str, str]):
    print("PROMPT PATH MAP")
    for label, path in mapping.items():
        print(f"- {label}: {path}")


def print_menu() -> None:
    print()
    print("=" * 70)
    print("AGENTIC REVIEW MENU")
    print("1 - DB")
    print("2 - Endpoints")
    print("3 - Architecture")
    print("4 - Run All")
    print("0 - Exit")
    print("=" * 70)


def print_result(title: str, text: str) -> None:
    print()
    print(f"RUNNING: {title}")
    print(text)
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/collectors/db_collector.py</summary>

```python
import sqlite3
from pathlib import Path


DATABASE_NAME = "enrolment.db"


def _validate_student(student: tuple[int, str, str]) -> tuple[bool, str]:
    student_id, student_name, subject_code = student
    if not isinstance(student_id, int):
        return False, "student_id must be integer"
    if not student_name:
        return False, "student_name required"
    if not subject_code:
        return False, "subject_code required"
    return True, "ok"


def collect(app_dir: Path, repo_root: Path) -> tuple[bool, str]:
    db_path = app_dir / "legacy-lab3" / DATABASE_NAME
    if not db_path.exists():
        return False, f"Missing local database file: {db_path.name}"

    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()

    students = cursor.execute(
        """
        SELECT student_id, student_name, subject_code
        FROM students
        """
    ).fetchall()
    count_asd101 = cursor.execute(
        "SELECT COUNT(*) FROM students WHERE subject_code = ?",
        ("ASD101",),
    ).fetchone()[0]
    conn.close()

    if len(students) != 10:
        return False, f"Expected 10 students, found {len(students)}"

    for student in students:
        ok, msg = _validate_student(student)
        if not ok:
            return False, msg

    return True, (
        "Database evidence: students table has 10 valid rows; "
        f"ASD101 rows count is {count_asd101}; fields are student_id, student_name, subject_code."
    )
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/collectors/endpoints_collector.py</summary>

```python
import os
import re
from pathlib import Path

import requests


ROUTE_PATTERN = re.compile(r"@\w+_bp\.(get|post)\(\"([^\"]+)\"\)")


def _test_endpoint(base_url: str, method: str, path: str) -> str:
    """Test a single endpoint and return evidence string."""
    url = f"{base_url}{path}"
    
    try:
        if method.upper() == "GET":
            response = requests.get(url, timeout=2)
        elif method.upper() == "POST":
            response = requests.post(url, data={"question": "test"}, timeout=2)
        else:
            return f"{method.upper()} {path} [UNSUPPORTED METHOD]"
        
        elapsed_ms = int(response.elapsed.total_seconds() * 1000)
        status = response.status_code
        
        if status == 200:
            return f"{method.upper()} {path} returned {status} in {elapsed_ms}ms"
        else:
            return f"{method.upper()} {path} returned {status} in {elapsed_ms}ms"
    
    except requests.exceptions.ConnectionError:
        return f"{method.upper()} {path} [CONNECTION REFUSED - app not running]"
    except requests.exceptions.Timeout:
        return f"{method.upper()} {path} [TIMEOUT]"
    except Exception as exc:
        return f"{method.upper()} {path} [ERROR: {type(exc).__name__}]"


def collect(app_dir: Path, repo_root: Path) -> tuple[bool, str]:
    """Collect live endpoint evidence by making real HTTP requests."""
    flask_base_url = os.getenv("FLASK_BASE_URL", "http://localhost:5001")
    
    route_files = [
        app_dir / "enrolment-service" / "routes" / "normal_ui.py",
        app_dir / "enrolment-service" / "routes" / "ai_mode.py",
    ]

    missing = [str(path.relative_to(app_dir)) for path in route_files if not path.exists()]
    if missing:
        return False, "Missing route files: " + ", ".join(missing)

    endpoints: list[tuple[str, str]] = []

    for route_file in route_files:
        content = route_file.read_text(encoding="utf-8")
        for method, route in ROUTE_PATTERN.findall(content):
            endpoints.append((method, route))

    if not endpoints:
        return False, "No Flask routes found in route files."

    # Test each endpoint with real HTTP requests
    evidence_parts = []
    connection_failures = 0
    
    for method, route in sorted(set(endpoints)):
        result = _test_endpoint(flask_base_url, method, route)
        evidence_parts.append(result)
        if "CONNECTION REFUSED" in result:
            connection_failures += 1
    
    evidence = "Live endpoint evidence: " + "; ".join(evidence_parts) + "."
    
    # If all endpoints failed to connect, warn that app isn't running
    if connection_failures == len(endpoints):
        return False, "Flask app not running. Start the app first, then run the agentic loop."
    
    return True, evidence
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/collectors/architecture_collector.py</summary>

```python
from pathlib import Path


REQUIRED_SERVICES = ["frontend-service", "enrolment-service", "database-service"]


def collect(app_dir: Path, repo_root: Path) -> tuple[bool, str]:
    required_paths = [
        app_dir / "frontend-service" / "templates" / "index.html",
        app_dir / "frontend-service" / "css" / "styles.css",
        app_dir / "enrolment-service" / "app.py",
        app_dir / "database-service" / "app.py",
        app_dir / "database-service" / "init_db.py",
        app_dir / "docker-compose.yml",
    ]

    missing = [str(path.relative_to(app_dir)) for path in required_paths if not path.exists()]
    if missing:
        return False, "Architecture evidence incomplete. Missing: " + ", ".join(missing)

    compose_text = (app_dir / "docker-compose.yml").read_text(encoding="utf-8")
    present = [name for name in REQUIRED_SERVICES if name in compose_text]
    if len(present) != len(REQUIRED_SERVICES):
        return False, "docker-compose does not define all required services."

    return True, (
        "Architecture evidence: frontend, backend, database service files and "
        "three-service compose topology are present."
    )
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/pipelines/db_pipeline.py</summary>

```python
def build_user_prompt(task_prompt: str, context_prompt: str, evidence: str) -> str:
    """Build user prompt for database review, injecting placeholders."""
    
    # Replace placeholders in task_prompt
    task_with_evidence = task_prompt.replace("{{REVIEW_TARGET}}", "Database")
    task_with_evidence = task_with_evidence.replace("{{VALIDATION_EVIDENCE}}", evidence)
    
    return f"""
{task_with_evidence}

Application Context:
{context_prompt}
""".strip()
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/pipelines/endpoints_pipeline.py</summary>

```python
def build_user_prompt(task_prompt: str, context_prompt: str, evidence: str) -> str:
    """Build user prompt for endpoints review, injecting placeholders."""
    
    # Replace placeholders in task_prompt
    task_with_evidence = task_prompt.replace("{{REVIEW_TARGET}}", "Endpoints")
    task_with_evidence = task_with_evidence.replace("{{VALIDATION_EVIDENCE}}", evidence)
    
    return f"""
{task_with_evidence}

Application Context:
{context_prompt}
""".strip()
```

</details>

<details>
<summary>enrolment-app-open-ai/agentic_loop/pipelines/architecture_pipeline.py</summary>

```python
def build_implementation_prompt(task_prompt: str, evidence: str) -> str:
    return f"""
{task_prompt}

Evidence:
{evidence}
""".strip()


def build_review_prompt(implementation_output: str, evidence: str) -> str:
    return f"""
Implementation Recommendation:
{implementation_output}

Evidence:
{evidence}
""".strip()
```

</details>

**Run instructions**

```bash
python agentic_loop.py
```

</details>


## 7. Agentic Workflow and Improvement Cycle

<details>
<summary>Agentic workflow</summary>

**Execution Steps:**

1. **Deploy:** `docker-compose up` (loop needs live endpoints)
2. **Run:** `python agentic_loop.py` from `enrolment-app-open-ai/`
3. **Choose:** 1=DB, 2=Endpoints, 3=Architecture
4. **Observe:** Evidence collection (DB state, HTTP requests, service files)
5. **Review:** Stage banners: `[START]` → `[OBSERVE]` → `[PROMPTS]` → `[LLM]` → `[DONE]`
6. **Capture:** Implementation output + review feedback
7. **Iterate:** Refine prompts/evidence if output unclear → rerun

**Document:**
- Evidence collected
- Implementation recommendations
- Review feedback (Architecture mode)

</details>

<details>
<summary>Improve and record</summary>

**Review Targets:**

1. **Database** — Verify records match expected student data
2. **Endpoints** — Test live HTTP requests and responses
3. **Architecture** — Validate service boundaries and dependencies

**Evidence Requirements:**

- Implementation agent: Uses live DB + endpoint evidence only, no assumptions
- Review agent: Validates implementation recommendations using same evidence
- Both agents: Must confirm Flask app is running

**Improvement Workflow:**

```
PLAN → OBSERVE (DB + Endpoints + Architecture) → IMPLEMENT → REVIEW → ADAPT
```

**Steps:**

1. Choose review target: DB, Endpoints, or Architecture
2. Run agentic loop and capture output
3. Select one prompt to improve:
   - `prompts/service/implementation/` (system, task, context)
   - `prompts/lab4/implementation/` (architecture system, task)
   - `prompts/lab4/review/` (review)
4. Apply prompt change
5. Rerun same review target
6. Record result:

```
Review Target: [DB/Endpoints/Architecture]
Prompt Changed: [filename]
Before: [original issue]
After: [improvement]
Evidence: [output comparison]
Decision: [Accept/Partially Accept/Reject]
```

**Repeat for each review target.**

</details>

---

## 8. Evidence Log

<details>
<summary>Record Evidence</summary>

| Check | Expected Result | Actual Result | Pass/Fail |
|---------|---------|---------|---------|
| **Environment Setup** |
| `.env` updated | `FLASK_BASE_URL=http://localhost:5001` added | | |
| `requirements.txt` includes requests | `requests` library listed | | |
| **Deployment** |
| Microservices running | `docker-compose up` successful | | |
| Flask app accessible | `http://localhost:5001/students` returns 200 | | |
| Database initialized | 10 student records exist | | |
| **Agentic Loop Execution** |
| Loop starts | `python agentic_loop.py` shows menu | | |
| Menu options visible | 1=DB, 2=Endpoints, 3=Architecture, 0=Exit | | |
| **DB Review Mode** |
| DB evidence collected | Returns DB state with 10 records | | |
| Stage banners shown | `[START]` → `[OBSERVE]` → `[PROMPTS]` → `[LLM]` → `[DONE]` | | |
| Implementation output | Recommendation based on DB evidence | | |
| **Endpoints Review Mode** |
| Live HTTP requests made | Evidence shows status codes + response times | | |
| Connection check works | If app down: "Flask app not running" error | | |
| Endpoint evidence valid | `GET /students returned 200 in XXms` format | | |
| Implementation output | Recommendation based on live endpoints | | |
| **Architecture Review Mode** |
| Service files checked | Frontend, enrolment, database services validated | | |
| docker-compose verified | Three-service topology confirmed | | |
| Implementation output | Architecture recommendation captured | | |
| Review output | Review agent feedback captured | | |
| **Prompts and Evidence** |
| Service prompts loaded | DB/Endpoints use `prompts/service` | | |
| Lab4 prompts loaded | Architecture uses `prompts/lab4` | | |
| Placeholders injected | `{{REVIEW_TARGET}}` and `{{VALIDATION_EVIDENCE}}` replaced | | |
| Evidence-based output | No assumptions, only observed data used | | |
| **Improvement Cycle** |
| Prompt refined | Changed one prompt file | | |
| Rerun comparison | Before/after outputs captured | | |
| Result documented | Review Target, Prompt, Before, After, Evidence, Decision recorded | | |

</details>

---

## 9. Reflection

<details>
<summary>Answer Briefly:</summary>

1. Why did microservices pattern suit UI mode vs AI mode separation?
2. Which service boundary most improved maintainability?
3. How did agentic loop evidence validate your architecture decisions?
4. What production-readiness change matters most for deployment?

</details>

---

## 10. Key Learning Point

<details>
<summary>Learning Outcome</summary>

**Architecture is evidence-driven:**
- UI mode and AI mode separated across three services (frontend, enrolment, database)
- Agentic loop validates DB, Endpoints, and Architecture using live evidence
- Stronger decisions require: observe → prompt → review → adapt

**Workflow:**
```
PLAN → OBSERVE → IMPLEMENT → REVIEW → ADAPT
```

</details>
