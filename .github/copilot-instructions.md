# Copilot Instructions For Product Proxy Demo App

These instructions are intended for the blank app repository where you generate code for the token-efficiency lab.

## Tech Stack

- Python 3.11+ with FastAPI (web framework) and uvicorn (ASGI server).
- httpx for async outbound HTTP calls.
- pydantic for response models / schema.
- pytest + respx for tests (respx mocks the upstream API so tests never hit the network).
- Vanilla HTML + CSS + JS front end — no framework and no build step.

## Project Structure

- Source lives in `src/`, tests in `test/`.
- Each language goes in its own file (HTML, CSS, JS, and Python are separate).

```
pyproject.toml          # build config, dependencies, pytest settings
requirements.txt
src/app/
  __init__.py
  config.py             # constants: FAKE_STORE_URL, REQUEST_TIMEOUT, STATIC_DIR
  models.py             # Pydantic models: Product, Rating
  main.py               # FastAPI app + routes + static mount
  static/
    index.html          # markup only — links the css/js
    styles.css          # all styling, including theme variables
    app.js              # all client logic (fetch + render + theme)
test/
  test_main.py          # endpoint tests (import from app.*)
```

`pyproject.toml` must set:

```toml
[tool.setuptools.packages.find]
where = ["src"]

[tool.setuptools.package-data]
app = ["static/*"]

[tool.pytest.ini_options]
pythonpath = ["src"]
testpaths = ["test"]
```

## Environment Setup

1. Use Python 3.11 or newer.
2. Create a virtual environment before installing dependencies:
   - `python3 -m venv .venv`
   - `.venv/bin/pip install -r requirements.txt`
3. Run the API locally with `.venv/bin/uvicorn app.main:app --app-dir src --reload`.
4. Run the test suite with `.venv/bin/python -m pytest -q`.

## FastAPI Coding Standards

- Use FastAPI as the web framework.
- Declare an explicit `response_model` on every list endpoint so the OpenAPI schema documents every property.
- Use `Field(..., description=..., examples=[...])` on model fields so Swagger shows descriptions and examples.
- Use a shared `httpx.AsyncClient` and apply `REQUEST_TIMEOUT` per request.
- Ensure the `lifespan` function checks for an already bounded `app.state.client` (such as one injected by your test runner) before initializing a new one. This keeps tests from accidentally wiping mock configurations.
- Keep route handlers small, constants in `config.py`, and models in `models.py`.
- Add concise docstrings to route handlers that describe behavior.
- Use meaningful HTTP status codes for upstream failures and validation problems.
- Keep naming, module boundaries, and file organization simple and beginner-friendly.

## Front-End Coding Standards

- Keep HTML, CSS, and JS in separate files (`index.html`, `styles.css`, `app.js`).
- HTML holds markup only; link the stylesheet and script rather than inlining them.
- Drive all theming with CSS custom properties on `:root`; do not hardcode colors outside `:root`.
- Escape every interpolated value from upstream data (e.g. an `escapeHtml()` helper) to prevent XSS.
- Keep the front end framework-free with no build step.

## API Documentation Expectations

- Ensure Swagger/OpenAPI is available at `/docs` and the raw schema at `/openapi.json`.
- Rely on FastAPI's auto-generated docs — no extra code beyond `response_model`.
- Make sure response models are explicit so the OpenAPI schema is complete and readable.

## Testing Expectations

- Write tests for the API behavior described in the prompt.
- Use `respx.mock` to intercept the upstream API so tests never require real network calls.
- When testing endpoints with a shared/persistent client in `lifespan`, standard `@respx.mock` decorators can bypass routers. Provide an autouse fixture in the test suite to bind the `respx_mock` transport explicitly:
  ```python
  @pytest.fixture(autouse=True)
  def mock_app_client(respx_mock):
      client = httpx.AsyncClient(transport=respx.transports.MockTransport(router=respx_mock))
      app.state.client = client
      yield
      app.state.client = None
  ```
- Define a sample products fixture that covers every model field.
- Cover success cases, filtering behavior, upstream failure handling, and HTML/OpenAPI expectations where appropriate.
- Keep tests focused and readable, using `pytest` conventions.

## Teardown Expectations

- Stop any running `uvicorn` process when finished.
- Deactivate the virtual environment when you are done.
- Leave the generated project files in place for review and comparison.

## Delivery Behavior

- Prefer generating the complete app in one pass when given a complete prompt.
- Follow a sensible build order: scaffold the package and config, then `config.py`/`models.py`/`main.py`, then the static front end, then theming, then tests.
- Do not make assumptions, ask the user for clarifications and ensure you have a full plan before generating code.
- Avoid unnecessary explanation text in generated output; prioritize executable code that is well documented and readable.