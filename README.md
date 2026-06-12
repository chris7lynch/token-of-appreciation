# GitHub Copilot Token Efficiency Lab

Welcome to the GitHub Copilot Token Efficiency Lab! This repository is designed for GitHub Copilot Business and Enterprise users to understand how to optimize token consumption under the new usage-based billing model.

## The New Billing Model

GitHub Copilot has shifted to **Usage-Based Billing** utilizing **GitHub AI Credits**. Your usage depends on the specific model selected and the number of tokens consumed.

Code completions (ghost text) are not billed in AI credits and remain unlimited.

**Important Links:**
- [Usage-based billing for organizations and enterprises](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)
- [Models and pricing for GitHub Copilot](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/copilot-billing/models-and-pricing)

### Understanding Token Types

You are billed per token, but not all tokens are priced the same. Every interaction is composed of three kinds:

| Token Type | What it is | Cost impact |
| :--- | :--- | :--- |
| **Input** | Everything sent *to* the model: your prompt, attached files, open tabs, and the entire prior chat history. | Grows with every follow-up turn and every file you attach. |
| **Output** | Everything the model *generates*: code, plus any explanations, apologies, and pleasantries. | Verbose answers and re-generations cost more. |
| **Cached** | Context the model reuses across turns instead of reprocessing from scratch. | Billed at a steep discount vs. fresh input — keeping a focused, stable context lets the cache work for you. |

> **Takeaway:** A long, meandering chat re-sends its whole history as *input* on every turn and produces verbose *output* on every reply. Tight context plus a tight prompt keeps all three columns small.

---

## Module 1: IDE Setup & Optimizations

Before the demo, adjust these VS Code settings so Copilot stops spending tokens on your behalf. Open **Settings (JSON)** with `Cmd+Shift+P` → *Preferences: Open User Settings (JSON)* and add the following:

```jsonc
{
  // Don't auto-diagnose and re-fix generated code after a turn. Each auto-fix
  // is another billed request that re-sends the conversation as input tokens.
  "github.copilot.chat.agent.autoFix": false,

  // Cap how many tool-call iterations the agent runs per turn (default 25).
  // A runaway agent can burn a lot of credits before it stops on its own.
  "chat.agent.maxRequests": 5,

  // Compress large terminal output before it's sent to the model, instead of
  // shipping the entire log as input tokens. (Preview)
  "chat.tools.compressOutput.enabled": true,

  // When the context window fills, summarize the history instead of re-sending
  // every prior turn verbatim. Keep this on.
  "github.copilot.chat.summarizeAgentConversationHistory.enabled": true
}
```

### What each setting does

- **`github.copilot.chat.agent.autoFix` → `false`**: By default (`true`), after the agent edits code it will automatically try to diagnose and fix any problems it detects. Each of those passes is another *billed* request that re-sends the whole conversation. Turning it off means you decide when a fix is worth spending on.
- **`chat.agent.maxRequests` → `5`**: One chat turn can fan out into many tool calls (read file, edit, run terminal, retry…). The default ceiling is **25**. Lowering it forces the agent to stop and check in with you before it racks up 25 billed steps chasing a problem.
- **`chat.tools.compressOutput.enabled` → `true`**: When the agent runs a command, the full terminal output is normally sent back to the model as input tokens. This (preview) setting collapses unchanged diffs, drops lockfile noise, and strips install progress first — directly shrinking input tokens.
- **`github.copilot.chat.summarizeAgentConversationHistory.enabled` → `true`**: Long sessions eventually overflow the context window. With this on, Copilot summarizes the earlier history instead of dropping it or re-sending it in full every turn — keeping input cost flatter as a conversation grows.

> **Inline completions are free.** Code completions (ghost text) are *not* billed in AI credits and are already enabled by default for code — no setting needed. Lean on them: they do real work for zero credits and reduce how often you reach for billed chat.

> **Shortcut:** This repo ships these settings in [.vscode/settings.json](.vscode/settings.json), so simply opening the folder in VS Code applies them at the workspace level — no manual editing required. Use the User Settings approach above if you'd rather make the change global to all your projects.

> Verify your IDE is on the latest version and the Copilot extension is current — older versions can display incorrect model pricing and usage numbers.

### Managing the Active Chat Context

The single biggest lever on input tokens is **what you put in the context window**. Copilot only needs the files and lines relevant to the task — everything else is wasted input.

- **Add only what you need:** Click the **paperclip / "Add Context"** button (or type `#`) to attach a *specific* file (`#file`), symbol (`#sym`), or folder. Prefer attaching one file over letting Copilot guess from open tabs.
- **Attach a selection, not the whole file:** Highlight just the relevant lines in the editor, then use **"Add Selection to Chat"** (`Cmd+L`). This sends only those lines instead of the entire file.
- **Exclude open editors:** In the context pill area above the prompt box, remove the auto-added "open editors" / current-file pills you don't need by hovering and clicking the **×**.
- **Start fresh between tasks:** Click **New Chat** (`Cmd+N` in the chat view) when you switch tasks. A stale thread keeps re-sending old turns as input on every new question. **/clear** also works here.
- **Use `#codebase` sparingly:** It performs a workspace search and can pull in a lot of input tokens. Reach for a targeted `#file` first.

---

## Module 2: Model Choices and Associated Costs

Not all models are created equal. Copilot lets you swap your model from the dropdown in the chat input box. Reference the live [Models and pricing](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/copilot-billing/models-and-pricing) table for current per-token rates.

* **Lightweight Models:** Fast and use significantly fewer AI credits per token. Great for boilerplate, scaffolding, regex, and well-specified tasks.
* **Powerful Models:** Cost more AI credits per token but excel at ambiguous problems, deep refactoring, and subtle bugs.

**Lab Action:** Open the model dropdown, look at the pricing shown per model, and switch to a **Lightweight** model. You'll use it in Module 3 to prove that a well-specified task does not need a frontier model.

---

## Module 3: Same App, Two Approaches (The Challenge)

We will build the **"Token of Appreciation"** API twice and compare the AI credit cost. The app wraps the free, public [Fake Store API](https://fakestoreapi.com/) (`https://fakestoreapi.com`), which is stable and repeatable for demos:

- `GET /users` → proxies `https://fakestoreapi.com/users` and returns the list of users.
- `POST /appreciation` → accepts a `user_id` and a `message`, looks the user up in the Fake Store API, and returns a confirmation payload.

> **Before you start:** Open the Copilot **usage / AI credits view** (or your org's usage dashboard) and note your current credit total. You'll check it again after each run.

### Run 1 — The "Random Chatting" Approach (inefficient)
*Stay on the `main` branch. Use a **Powerful** model. Send these as separate messages, accepting each result before the next.*

1. `Create a Python web API.`
2. `Make it return a list of users.`
3. `Actually use FastAPI so I get Swagger docs.`
4. `The users should come from an external API, not hardcoded.`
5. `Use the Fake Store API for that.`
6. `Now add a route so I can send someone an appreciation message.`
7. `It should validate the input. Can you also explain how it all works?`

**Record your AI credit total now.** Note how each turn re-sent the growing conversation as input, and several replies included long explanations (output).

### Run 2 — The "Solid Prompt" Approach (efficient)
*Stay on `main` for this comparison run. Switch the model dropdown to a **Lightweight** model. Send this as a single message:*

> Build a single-file Python FastAPI app named `main.py`.
> Requirements:
> - `GET /users`: fetch `https://fakestoreapi.com/users` with `httpx` and return the JSON list as-is.
> - `POST /appreciation`: accept a JSON body with `user_id: int` and `message: str` validated by a Pydantic `BaseModel`. Fetch `https://fakestoreapi.com/users/{user_id}`; if found, return `{ "to": "<firstname lastname>", "message": message, "status": "delivered" }`; if the upstream returns empty/404, return HTTP 404.
> - Use `async` route handlers and a module-level `httpx.AsyncClient`.
> - Include a `requirements.txt` listing `fastapi`, `uvicorn`, and `httpx`.
> Output only the code for `main.py` and `requirements.txt` in separate fenced blocks. Do not explain the code.

**Record your AI credit total again.** The detailed prompt removes ambiguity so the **Lightweight** model produces the same working app the Powerful model did — for a fraction of the credits.

### Compare the Results

Put the two outcomes side by side:

1. **AI credits spent** — Run 1 (multi-turn, Powerful) vs. Run 2 (single-shot, Lightweight). This is the headline number for the talk.
2. **The generated code** — Diff the two `main.py` files. They should be functionally equivalent: same two routes, same Fake Store API calls, same Pydantic validation. The point: precise requirements, not a bigger model, are what guarantee the result.
3. **Run it to prove parity** — for either version:
   ```bash
   pip install -r requirements.txt
   uvicorn main:app --reload
   ```
   Open `http://127.0.0.1:8000/docs`, call `GET /users`, then `POST /appreciation` with a body like `{ "user_id": 1, "message": "Thanks for the great work!" }`.

> **Note:** This `main` branch contains lab instructions only — no source code. The fully built, efficient version lives on the `efficient-prompts` branch, which also adds a `.github/copilot-instructions.md` so the whole team gets these savings automatically. We'll create that branch later in the session.