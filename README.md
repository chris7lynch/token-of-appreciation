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

The core takeaway for the lab: tight context and precise prompts lower credits without sacrificing output quality.

## Repository Contents

- [README.md](README.md) - Lab flow, setup instructions, and runbook.
- [.github/copilot-instructions.md](.github/copilot-instructions.md) - Full use-case requirements and implementation constraints.
- [.github/prompts/build-product-proxy-api.prompt.md](.github/prompts/build-product-proxy-api.prompt.md) - Single-shot prompt for generating the same use case.

## Lab Workflow

### Step 1) Use this repository in the browser only

Do not clone this repository for the demo run. Open this README in your browser and copy/paste only when instructed.

### Step 2) Create a blank app repository for the baseline run

Use a unique repository name each time so every run starts from a true blank canvas. Example:

```bash
mkdir token-of-appreciation-simple
cd token-of-appreciation-simple
git init
code .
```

### Step 3) Capture starting usage

Before any generation, open Copilot usage and record your current AI credits.

### Step 4) Baseline run with raw chat first (multi-turn)

In the blank app repo:

1. Start a new chat.
2. Use a powerful model.
3. Do not create or paste prompt/instruction files yet.
4. Build the use case through multiple chat messages.
  - `Create a Python API. It should return Fake Store API products from https://fakestoreapi.com/products`
  - `Add a second endpoint called highly-rated-items that returns only products with more than 100 ratings and a higher rating than 3.0.`
  - `Create tests for those APIs.`
  - `Ensure there is a swagger endpoint exposed to view my new API information.  It should include property mappings for all upstream API properties.`
  - `Create a static html page for this application so that it renders on the / route.`
  - `Ensure that page loads the api endpoints in a pretty format as well.`
  - `I need both light and dark mode support`
  - `I need to break this up so that my application is properly structured.  My CSS, HTML, JS, and Python should all be independent.  Ensure that my application is following proper coding standards, with my source code in /src and tests in /test`
5. End when the app is complete.

Then immediately record baseline usage.

### Step 5) Record baseline usage

Record:

1. AI credits used after the raw chat run

### Step 6) Create a second blank repository

Use a different unique repository name for the detailed-prompt run so it is a separate blank canvas. Example:

```bash
cd ..
mkdir token-of-appreciation-detailed
cd token-of-appreciation-detailed
git init
mkdir -p .github/prompts
touch .github/copilot-instructions.md
touch .github/prompts/build-product-proxy-api.prompt.md
mkdir -p .vscode
touch .vscode/settings.json
code .
```

### Step 7) Copy settings before the detailed prompt run

Copy the workspace settings into the new repository before you run the good prompt so you can prove the settings help:

1. Copy [.vscode/settings.json](.vscode/settings.json) into the second repo as [.vscode/settings.json](.vscode/settings.json)
2. Create `.github/copilot-instructions.md` from [.github/copilot-instructions.md](.github/copilot-instructions.md)
3. Create `.github/prompts/build-product-proxy-api.prompt.md` from [.github/prompts/build-product-proxy-api.prompt.md](.github/prompts/build-product-proxy-api.prompt.md)
4. Copy [requirements.txt](requirements.txt) into the repository root as [requirements.txt](requirements.txt)

### Step 8) Efficient run with detailed prompt (single shot)

In the second blank repo:

1. Start a new chat.
2. Switch to a lightweight model.
3. Open and submit the prompt from [.github/prompts/build-product-proxy-api.prompt.md](.github/prompts/build-product-proxy-api.prompt.md).

### Step 9) Capture efficient-run usage

Record:

1. AI credits used after the single-shot run

### Step 10) Compare and document results

Record a short summary including:

1. AI credits used in the raw chat run
2. AI credits used in the single-shot run
3. Net reduction and percentage savings

## Compare Outcomes

1. AI credits spent: multi-turn powerful run vs single-shot lightweight run.
2. Generated behavior parity: validate against requirements in [.github/copilot-instructions.md](.github/copilot-instructions.md).
3. Runtime validation:

```bash
pip install -r requirements.txt
uvicorn main:app --reload
pytest
```

Open http://127.0.0.1:8000/docs and verify both endpoints.

