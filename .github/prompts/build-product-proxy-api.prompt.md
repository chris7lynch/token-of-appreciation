# Build Product Proxy API (Single Shot)

Generate a complete Python FastAPI application in this repository that proxies the public Fake Store API, serves a styled landing page, and ships with tests.

## Dependency File

- Use the repository's `requirements.txt` file as the source of Python dependencies.
- Ensure the generated app can be installed and run with `pip install -r requirements.txt`.
- Keep the dependency list minimal and appropriate for FastAPI, runtime HTTP calls, and testing.

## API Endpoints

Source upstream data from https://fakestoreapi.com/products.

| Method | Path                  | Description                                                      |
| ------ | --------------------- | ---------------------------------------------------------------- |
| GET    | `/`                   | Serves the static landing page (HTML). Hidden from docs.         |
| GET    | `/products`           | All products from the Fake Store API.                            |
| GET    | `/highly-rated-items` | Only products with `rating.count > 100` AND `rating.rate > 3.0`. |

### Route behavior rules

- `/` returns a `FileResponse` for `index.html` and is hidden from the OpenAPI docs.
- Mount static files at `/static`.
- Map upstream failures explicitly:
  - upstream non-2xx (`HTTPStatusError`) -> propagate that status code with a clear `detail` message.
  - network failure (`RequestError`) -> `502` "Unable to reach the Fake Store API."
- `/products/{id}`: the upstream returns an empty body for unknown IDs. Guard `response.json()` against `ValueError` and return `404` "Product not found."
- All list endpoints declare a `response_model` so the OpenAPI schema documents every property.

## Data Model (maps all upstream fields)

- `Product`: `id`, `title`, `price`, `description`, `category`, `image`, `rating`.
- `Rating`: `rate`, `count`.
- Preserve the full upstream product shape — do not drop fields, and model nested rating data explicitly.
- Keep the mapped response aligned with the upstream schema so every upstream field is represented in the API response and documentation.

### Example upstream response

The upstream `/products` endpoint returns an array of product objects shaped like this:

```json
[
   {
      "id": 1,
      "title": "Fjallraven - Foldsack No. 1 Backpack, Fits 15 Laptops",
      "price": 109.95,
      "description": "Your perfect pack for everyday use and walks in the forest. Stash your laptop (up to 15 inches) in the padded sleeve, your everyday",
      "category": "men's clothing",
      "image": "https://fakestoreapi.com/img/81fPKd-2AYL._AC_SL1500_t.png",
      "rating": {
         "rate": 3.9,
         "count": 120
      }
   },
   {
      "id": 2,
      "title": "Mens Casual Premium Slim Fit T-Shirts ",
      "price": 22.3,
      "description": "Slim-fitting style, contrast raglan long sleeve, three-button henley placket, light weight & soft fabric for breathable and comfortable wearing. And Solid stitched shirts with round neck made for durability and a great fit for casual fashion wear and diehard baseball fans. The Henley style round neckline includes a three-button placket.",
      "category": "men's clothing",
      "image": "https://fakestoreapi.com/img/71-3HjGNDUL._AC_SY879._SX._UX._SY._UY_t.png",
      "rating": {
         "rate": 4.1,
         "count": 259
      }
   }
]
```

## Swagger / OpenAPI

- `/docs` serves the Swagger UI and `/openapi.json` serves the raw schema.
- Every `Product` and `Rating` property is documented with a description and example.

## Front-End Behavior

### Landing page (`index.html`)

- A header with the title and a theme-toggle button.
- A set of working links to the API documentation: the Swagger UI at `/docs` and the raw OpenAPI schema at `/openapi.json`.
- A "Live Products" section with two toggle buttons (All products / Highly rated), a status line, and an empty grid that JS fills.
- A footer.

### Theming behavior

- The default theme is dark.
- Resolve the active theme in this priority order:
  1. The user's explicit choice saved in `localStorage.theme`.
  2. The OS preference via `prefers-color-scheme` when the user has not chosen.
- The toggle button flips `data-theme`, persists the choice to `localStorage`, and updates the icon/label.
- Listen to `prefers-color-scheme` changes only when the user has not made an explicit choice.
- Light mode uses a darker accent (`#0284c7`) for contrast; dark mode uses `#38bdf8`.
- Use a responsive product grid (`repeat(auto-fit, minmax(240px, 1fr))`) with smooth background and color transitions.

### Product loading behavior

- `loadProducts(endpoint)` fetches the selected endpoint, surfaces non-OK responses and network errors into the status line, and renders product cards.
- Each card shows the image, category, title, price formatted to 2 decimals, and a star rating with count.
- The toggle buttons switch the active endpoint; load `/products` on start.

## Test Coverage

- `/` returns HTML containing the title.
- `/products`: success; upstream 500 -> 500; network error -> 502.
- `/highly-rated-items`: correct filtering (only `count > 100` AND `rate > 3.0`); upstream error propagates.

## Acceptance Criteria

- All endpoints behave per the rules above and the tests pass.
- `/docs` shows every `Product`/`Rating` property with descriptions.
- The landing page renders the live product grid and toggles between All and Highly rated.
- Light and dark modes both work, follow the OS by default, and persist the user's explicit choice.
- HTML, CSS, JS, and Python are each in separate files; source in `src/`, tests in `test/`.
