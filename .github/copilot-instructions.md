# Copilot Instructions For Product Proxy Demo App

These instructions are intended for the blank app repository where you generate code for the token-efficiency lab.

## Environment Setup

1. Use Python 3.11 or newer.
2. Create a virtual environment in the blank app repo before installing dependencies:
   - `python -m venv .venv`
   - `source .venv/bin/activate`
3. Install dependencies with `pip install -r requirements.txt`.
4. Use `uvicorn main:app --reload` to run the API locally.
5. Use `pytest` to run the test suite.

## FastAPI Coding Standards

- Use FastAPI as the web framework.
- Prefer explicit Pydantic response models for every response body.
- Keep route handlers small and place reusable logic in helper modules or services.
- Add concise docstrings to route handlers that describe behavior.
- Use meaningful HTTP status codes for upstream failures and validation problems.
- Keep naming, module boundaries, and file organization simple and beginner-friendly.
- Keep the implementation easy to read for a beginner while still being production-sensible.

## API Documentation Expectations

- Ensure Swagger/OpenAPI is available at `/docs`.
- Make sure response models are explicit so the OpenAPI schema is complete and readable.
- Keep the docs aligned with the runtime behavior of the API.

## Testing Expectations

- Write tests for the API behavior described in the prompt.
- Mock outbound HTTP requests so tests never require real network calls.
- Cover success cases, filtering behavior, upstream failure handling, and OpenAPI/schema expectations where appropriate.
- Keep tests focused and readable, using `pytest` conventions.

## Teardown Expectations

- Stop any running `uvicorn` process when finished.
- Deactivate the virtual environment when you are done.
- Leave the generated project files in place for review and comparison.

## Delivery Behavior

- Prefer generating the complete app in one pass when given a complete prompt.
- Do not make assumptions, ask the user for clarifications and ensure you have a full plan before generating code.
- Avoid unnecessary explanation text in generated output; prioritize executable code that is well documented and readable.