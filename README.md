# Coder Crew – Multi‑Agent AI Code‑Generation Framework

> **TL;DR** – A ready‑to‑run CrewAI project that lets you describe an assignment in plain English, and a fleet of AI agents will write, run, and validate Python code inside an isolated Docker sandbox.

---

## What This Project Does

- **Prompt‑to‑code**: Accept a natural‑language *assignment* (e.g. "calculate the first 1 000 000 terms of the Leibniz series and multiply by 4").
- **CrewAI orchestrator**: A *Crew* with a single **Coder** agent that uses the **sandbox_tools** toolkit.
- **Docker‑sandbox execution**: All generated Python files are stored in `sandbox/` and run inside a temporary `python:3.13‑slim` container, guaranteeing no side‑effects on the host.
- **Result aggregation**: The agent returns a markdown summary and the final script output, written to `output/solution.md`.

The whole flow is **self‑contained** – you only need a virtual‑environment with the dependencies listed in `pyproject.toml`.

---

## How It Works (Diagram)

```mermaid
flowchart TD
    A[User runs `coder run`] --> B[main.py loads assignment]
    B --> C[Coder Crew instantiated]
    C --> D[Agent "coder" reads assignment]
    D --> E[Agent calls sandbox_tools:
        - write_sandbox_file()
        - run_sandbox_python()
        - read_sandbox_file()]
    E --> F[Docker container runs the script]
    F --> G[Agent receives stdout
        and writes summary]
    G --> H[Output written to `output/solution.md`]
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style H fill:#9f9,stroke:#333,stroke-width:2px
```

---

## Core Components

| Component | Role | Key Files |
|-----------|------|-----------|
| **Crew definition** | Sets up the agent and the task pipeline | `src/coder/crew.py` |
| **Agent configuration** | Describes the coder’s purpose, goal and LLM | `src/coder/config/agents.yaml` |
| **Task configuration** | Defines the *coding_task* that the agent must perform | `src/coder/config/tasks.yaml` |
| **Sandbox toolkit** | Exposes safe file‑IO and Docker‑run helpers to the agent | `src/coder/tools/sandbox_tools.py` |
| **Entry point** | Parses the assignment and kicks off the crew | `src/coder/main.py` |
| **Docker sandbox** | Isolated runtime for generated scripts (`sandbox/`) | `src/coder/tools/sandbox_tools.py` (Docker command) |

---

## Getting Started

1. **Create a virtual environment** (the project already contains a helper script):
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
2. **Install the project in editable mode** (pulls CrewAI and all extras):
   ```bash
   pip install -e .
   ```
3. **Run the crew** – the default assignment is baked into `main.py`:
   ```bash
   coder   # or: python -m coder.main run
   ```
   The agent will write `sandbox/solution.py`, execute it in Docker, and dump a nicely formatted report to `output/solution.md`.

### Custom Assignments

Edit the `assignment` variable in `src/coder/main.py` or pass a JSON payload via the *trigger* helper:
```bash
python -m coder.main run_with_trigger '{"assignment": "Scrape the titles of the latest 5 posts on Hacker News"}'
```
The crew will automatically adapt to the new request.

---

## Extending the Project

- **Add more agents**: Create additional entries in `agents.yaml` and corresponding tasks in `tasks.yaml`. Use the same `sandbox_tools` or write new tool modules.
- **Custom tools**: Follow the pattern in `sandbox_tools.py`. Remember to set `cache_function = lambda *_, **__: False` for any state‑changing tool.
- **Testing**: Run `crewai test 5 gpt-4o-mini` to execute the crew multiple times with a cheaper model and spot regressions.
- **Docker image tweaks**: Change the base image in `run_sandbox_python()` if you need extra libraries (e.g. `python:3.13‑slim‑bullseye`).

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `ModuleNotFoundError: crewai` | Dependencies not installed or virtual‑env not activated | `source .venv/bin/activate && pip install -e .` |
| Docker command hangs | Docker not installed or current user lacks permission to run Docker | Install Docker Engine and add the user to the `docker` group |
| Sandbox file missing | Agent failed to write file (tool error) | Verify `write_sandbox_file` returns a success message; check the agent’s LLM response for malformed filenames |
| Empty `output/solution.md` | Agent never reached the *summary* step | Enable `verbose=True` in `crew.py` to see detailed agent logs |

---


## Want a Visual Demo?

Run the crew and then open the generated `output/solution.md` in the Antigravity markdown preview – it will render the code blocks, the Docker stdout, and the summary side‑by‑side.

Enjoy building AI‑driven coding crews! 🎉
