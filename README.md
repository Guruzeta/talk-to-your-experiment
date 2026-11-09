# Talk to Code

**A voice research partner that checks your experiments by running real code, and explains what it finds on a whiteboard as it talks.**

Built in one afternoon at the [SDxSDSU Voice Agent Hackathon](https://luma.com/uan97kyo) (Nov 2026, Open track).

## The problem

LLMs now write and run research code faster than people can read it. A single afternoon of
LLM-assisted experiments leaves a pile of scripts, logs, CSVs and plots that take hours to piece
together. Producing research has become cheap, but understanding it hasn't.

Docs and chat answers don't fix this. They are more text to parse. A good colleague does something
different: they look at the data, check claims before making them, draw a diagram, and slow down
when you're lost.

## What it does

You open a project and ask out loud, *"Why did run 6 explode if Verlet is supposed to be the stable one?"*
Sam, the agent:

1. **Checks before answering.** It writes analysis code and runs it in a Daytona cloud sandbox
   against your actual data, then returns numbers and plots.
2. **Explains while drawing.** It builds a hand-drawn diagram on a shared whiteboard step by step
   while it speaks, and writes key equations and numbers on the board.
3. **Adapts to you.** You can interrupt at any time. It keeps a profile of your background and preferred
   explanation level and style, and adjusts how it explains.
4. **Saves the finding.** Results go into a markdown report with figures, the project's to-do list
   is updated, and key decisions are saved to project memory for the next session.

**User:** a researcher with a folder of LLM-generated experiment runs they don't have time to read.
**Finish line:** a verified finding, explained at their level, saved as a report in the repo.

### Two conversation modes

- **To-do list:** work through your open questions (`TODO.md`) one by one. Items are checked off live.
- **Walkthrough:** Sam presents the project like a grad student and you probe like an advisor.

### Point it at any repo

Paste a GitHub URL into the Repo bar. The backend clones it into its own Daytona sandbox, indexes it
(file tree, README, notebook outlines, scripts, data columns) and keeps the sandbox warm. Sam can then
read files, search the code and run analysis inside the repo itself.

## Architecture

```
Browser (Vite)                          ElevenLabs Agent (voice, LLM, turn-taking)
  ├─ @elevenlabs/client  <── WebRTC ──>   tools the agent calls:
  ├─ hand-drawn whiteboard (Mermaid)        run_analysis, read_file, search_repo → Daytona sandbox
  ├─ tab recorder → explainer .webm         sketch, write_on_board, show_agenda  → whiteboard
  └─ FastAPI backend                        update_todo, save_report, remember   → project files
                                            set_explanation_style, update_profile → learner profile
```

| Piece | Role |
|---|---|
| **ElevenLabs Agents** | Real-time voice, interruptions and tuned turn-taking. Agent, prompt and 14 tools are created from code (`backend/setup_agent.py`) |
| **Daytona** | One named, persistent sandbox per project. All analysis code the agent writes runs there, never on the host |
| **FastAPI** (`backend/`) | Session auth, sandbox management, repo indexing, to-dos, reports, learner profile |
| **Frontend** (`frontend/`) | Voice client, whiteboard, live to-do list, and a recorder that turns a conversation into an explainer video |

## Demo project

`demo_experiment/` is a small physics study: a frictionless mass on a spring, simulated with four
integrators (Euler, symplectic Euler, Verlet, RK4) across seven runs. The notes are deliberately
messy and the data has real stories in it:

- Euler's energy drifts upward even though there is no friction.
- Verlet explodes at `dt = 2.1`, past its stability limit of `dt < 2/ω`.
- A stiffer spring (`k = 4`) oscillates twice as fast, as theory predicts.

## Run it

Requires Python with [uv](https://docs.astral.sh/uv/), Node.js, an ElevenLabs API key and a Daytona API key.

```bash
cp .env.example .env            # add ELEVENLABS_API_KEY and DAYTONA_API_KEY
cd backend && uv sync && uv run python setup_agent.py   # creates the agent, writes its id to .env
uv run uvicorn app:app --port 8000 --reload
cd ../frontend && npm install && npm run dev            # open http://localhost:5173
```

Re-run `setup_agent.py` after editing `agent/prompt.md` or the tool definitions to update the agent in place.

## Why voice and not text

Narration plus a diagram is easier to absorb than on-screen text plus a diagram, because text and the
diagram compete for your eyes (Mayer's *modality principle*). Voice also makes it nearly free to say
"wait, I'm lost," and that feedback is what lets the explanation adapt to the listener.
