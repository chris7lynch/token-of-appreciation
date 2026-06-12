# Build Product Proxy API (Single Shot)

Generate a complete Python FastAPI application in this repository.

## Dependency File

- Use the repository's `requirements.txt` file as the source of Python dependencies.
- Ensure the generated app can be installed and run with `pip install -r requirements.txt`.
- Keep the dependency list minimal and appropriate for FastAPI, runtime HTTP calls, and testing.

## Functional Requirements

1. Build GET /products.
   - Fetch products from https://fakestoreapi.com/products.
   - Return all product properties from upstream.

2. Build GET /highly-rated-items.
   - Source data from https://fakestoreapi.com/products.
   - Return only items where rating.count > 100 and rating.rate > 3.0.

3. Ensure Swagger docs are available at /docs.

4. Preserve the full upstream product shape in the response model.
   - Do not drop upstream fields.
   - Model nested rating data explicitly.
   - Treat the upstream product as an object with these top-level fields at minimum: `id`, `title`, `price`, `description`, `category`, `image`, and `rating`.
   - Treat `rating` as a nested object with at least `rate` and `count`.
   - Keep the mapped response aligned with the upstream schema so every upstream field is represented in the API response and documentation.
   - Preserve any additional upstream fields if they are present.

## Example Upstream Response

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

5. Add proper error handling for upstream request failures.
   - Return meaningful HTTP status codes.
   - Fail cleanly if the upstream API is unavailable or returns an error.

## Testing Requirements

1. Create tests for the API behavior in this prompt.
2. Use `pytest`.
3. Mock outbound HTTP requests so tests run offline.
4. Cover:
   - success responses for both endpoints
   - filtering behavior for highly-rated items
   - upstream failure handling
   - OpenAPI/Swagger availability or schema shape where practical

## Implementation Expectations

- Use Python 3.11+ and FastAPI.
- Use explicit response models and keep them aligned with the upstream API.
- Create or update `requirements.txt` in the repository root and make sure the application uses it.
- Keep the implementation simple and beginner-friendly.
- Use clear module boundaries and readable naming.

## Output Expectations

- Create the application files needed for the solution.
- Prioritize executable code over explanation but ensure it is well documented and readable.
- Do not make assumptions; ask for clarifications if needed before generating code.