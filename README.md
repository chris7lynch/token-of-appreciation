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

Before the demo, adjust these VS Code settings so Copilot stops spending tokens on your behalf. 

### Global vs. Per-Repository Settings

VS Code settings can be applied globally to all your projects (**User Settings**) or to a specific repository (**Workspace Settings**). 

- **In this Lab (Workspace Settings):** This repository ships these efficiency settings in the `[.vscode/settings.json](.vscode/settings.json)` file. Simply opening this folder in VS Code applies them automatically to this project without affecting your other work.
- **Globally (User Settings):** If you want these token-saving settings applied to *all* your projects, open the Command Palette (`Cmd+Shift+P`) → *Preferences: Open User Settings (JSON)* and add them there.

Here are the specific keys we added:

```jsonc
{
  // Automatically diagnosing and fixing code problems after an edit triggers new, 
  // billed requests that re-send the whole conversation. false = you decide when to fix.
  "github.copilot.chat.agent.autoFix": false,

  // One chat turn can fan out into many tool calls. The default is 25. Lowering 
  // it to 5 forces a runaway agent to stop before racking up high 25-step costs.
  "chat.agent.maxRequests": 5,

  // Collapses common terminal outputs (like install progress or lockfile noise) 
  // before sending, shrinking input volume.
  "chat.tools.compressOutput.enabled": true,

  // When long sessions fill the context window, Copilot summarizes earlier history 
  // rather than verbatim re-sending it. This keeps costs flatter as a thread grows.
  "github.copilot.chat.summarizeAgentConversationHistory.enabled": true
}
```

> **Inline completions are free.** Code completions (ghost text) are *not* billed in AI credits and are already enabled by default. Lean on them: they do real work for zero credits and reduce how often you reach for billed chat.

> Verify your IDE is on the latest version and the Copilot extension is current — older versions can display incorrect model pricing and usage numbers.

### Managing the Active Chat Context

The single biggest lever on input tokens is **what you put in the context window**. Copilot only needs the files and lines relevant to the task — everything else is wasted input.

- **Add only what you need:** Click the **paperclip / "Add Context"** button (or type `#`) to attach a *specific* file (`#file`), symbol (`#sym`), or folder. Prefer attaching one file over letting Copilot guess from open tabs.
- **Attach a selection, not the whole file:** Highlight just the relevant lines in the editor, then use **"Add Selection to Chat"** (`Cmd+L`). This sends only those lines instead of the entire file.
- **Exclude open editors:** In the context pill area above the prompt box, remove the auto-added "open editors" / current-file pills you don't need by hovering and clicking the **×**.
- **Start fresh between tasks:** Click **New Chat** (`Cmd+N` in the chat view) when you switch tasks. A stale thread keeps re-sending old turns as input on every new question. **/clear** also works here.
- **Use `#codebase` sparingly:** It performs a workspace search and can pull in a lot of input tokens. Reach for a targeted `#file` first.

---

## Module 2: The "Random Chatting" Approach (Inefficient)

We will build a **Product Proxy** API twice and compare the AI credit cost. The app wraps the free, public [Fake Store API](https://fakestoreapi.com/) (`https://fakestoreapi.com`), which is stable and repeatable for demos:

- `GET /products` → proxies `https://fakestoreapi.com/products` and returns the list of products.
- `GET /highly-rated-items` → fetches products from the Fake Store API and returns only items with more than 100 ratings and a rating higher than 3.0.
- Exposes a Swagger endpoint to view the API information.
- Includes simple tests for these APIs.

> **Before you start:** Open the Copilot **usage / AI credits view** (or your org's usage dashboard) and note your current credit total. You'll check it again after this run.

*Stay on the `main` branch. Use a **Powerful** model. Send these as separate messages, accepting each result before the next.*

1. `Create a Python API. It should return Fake Store API products from https://fakestoreapi.com/products`
2. `Add a second endpoint called highly-rated-items that returns only products with more than 100 ratings and a higher rating than 3.0.`
3. `Create tests for those APIs.`
4. `Finally, I want to ensure there is a swagger endpoint exposed to view my new API information.`

**Record your AI credit total now.** Note how each turn re-sent the growing conversation as input, and several replies included long explanations (output).

---

## Module 3: The "Solid Prompt" Approach & Model Comparison

Now let's see how much we can save when we provide Copilot with detailed instructions and compare models.

**Lab Action:** Swap to the `detailed-prompts` branch.

```bash
git checkout detailed-prompts
```

On this branch, you will find a `.github/copilot-instructions.md` file that sets the baseline context, and a dedicated prompt file that removes ambiguity.

### Model Choices and Associated Costs

Not all models are created equal. Copilot lets you swap your model from the dropdown in the chat input box. Reference the live [Models and pricing](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/copilot-billing/models-and-pricing) table for current per-token rates.

* **Lightweight Models:** Fast and use significantly fewer AI credits per token. Great for boilerplate, scaffolding, regex, and well-specified tasks.
* **Powerful Models:** Cost more AI credits per token but excel at ambiguous problems, deep refactoring, and subtle bugs.

**Lab Action:** Open the model dropdown, look at the pricing shown per model, and switch to a **Lightweight** model. You will use it for Run 2 to prove that a well-specified task does not need a frontier model.

### Run 2 — The Efficient Run

Start a **New Chat**, ensure you are using the **Lightweight** model, and use the prompt file provided on the `detailed-prompts` branch to generate the entire application at once. 

**Record your AI credit total again.** The detailed instructions and solid prompt remove ambiguity so the **Lightweight** model produces the exact same working app the Powerful model did — for a fraction of the credits.

### Compare the Results

Put the two outcomes side by side:

1. **AI credits spent** — Run 1 (multi-turn, Powerful) vs. Run 2 (single-shot, Lightweight). This is the headline number for the talk.
2. **The generated code** — Diff the generated files. They should be functionally equivalent: same two routes, same Fake Store API calls, same filtering logic. The point: precise requirements, not a bigger model, are what guarantee the result.
3. **Run it to prove parity** — for either version:
   ```bash
   pip install -r requirements.txt
   uvicorn main:app --reload
   ```
   Open `http://127.0.0.1:8000/docs`, call `GET /products`, then `GET /highly-rated-items`.
   You can also run the tests using `pytest`.