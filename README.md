***
https://github.com/user-attachments/assets/59ca5707-cfbc-4c4e-893c-9983858d5612

# Mockit 

Your OpenAPI spec is defined, but the backend implementation hasn't started. The frontend team is blocked, and development is on halt. 

Most existing mock tools fail you in one of three ways:
1.  They are **stateless**—you send a `POST` to create a resource, but a subsequent `GET` returns nothing or the same static file.
2.  They are **SaaS-only**—forcing you into a cloud subscription just to run a local mock server.
3.  They are **too lenient**—guessing semantic formats and silently bypassing contract issues. This gives a false sense of security; when the real backend finally launches, the frontend breaks.

**Mockit** is a zero-dependency, local binary that solves this. It maintains live in-memory state for CRUD flows, runs completely offline, and strictly enforces the OpenAPI contract. If your specification is sloppy or incomplete, Mockit refuses to work—just like a real, spec-aligned backend.

(_Note: Mockit is a commercial, closed-source developer tool. This repository serves as our documentation, issue tracker, and community feedback hub._)

---

## Technical Differentiation

Unlike general-purpose mocking tools or local servers that rely on custom databases, mock scripting languages, or loose fallback heuristics, Mockit operates using distinct structural mechanisms:

*   **Direct AST-to-Router Translation:** Mockit does not compile your OpenAPI document into a proprietary configuration format or secondary language. The HTTP routing engine (`go-chi`) dynamically registers routes and maps handlers directly to the `kin-openapi` Abstract Syntax Tree (AST) at startup.
*   **Path-Based Parameter Extraction:** Mockit does not maintain primary keys, database schemas, or internal IDs. It derives object identities dynamically. If a path is defined as `/users/{custom_sku}`, Mockit extracts `custom_sku` as the lookup key. On a `GET` request, it searches the live payload slice for a field matching that exact key and value.
*   **Emergent State Namespaces:** Ephemeral CRUD state is mapped dynamically on-the-fly based on the route hierarchy. A route like `/users/{id}/posts` translates directly into an isolated, memory-contained namespace string (`users.{id}.posts`). No foreign keys or database migrations are required.
*   **Strict Media-Type Interception:** Mockit evaluates incoming `Accept` headers at the edge against the defined response media types in the spec. If a client requests `Accept: application/xml` but the specification only defines `application/json`, Mockit rejects the request at the network boundary with `406 Not Acceptable`, rather than silently serializing fallback formats.
*   **Scream-Driven Logging:** Every byte emitted as a response is explicitly traced to its mechanical origin and printed using standardized log namespaces:
    *   `[EXAMPLE]`: Served from active, in-memory state or explicitly declared mock data.
    *   `[GENERATED]`: Created at runtime based strictly on schema validation constraints.
    *   `[FALLBACK]`: Emitted as a raw empty body because the specification defines no schema for the matched path and status.
    *   `[VIOLATION]`: Emitted when the client request violates the structural rules of the contract.

---

## Architectural Principles

The construction of Mockit is governed by three strict design principles:

### 1. OpenAPI is the Single Source of Truth
If a behavior, validation rule, security requirement, or response schema is not explicitly declared in your OpenAPI contract, it does not exist in Mockit. There are no shadow configurations, mock databases, or helper files.

### 2. Truth Over Realism
Most mock servers aim to return "realistic" data to make frontends look complete, even if the underlying API contract is poorly defined. Mockit treats contract validation as a compiler warning. If you define a string field but omit `format: email`, Mockit will return a basic random word—not an email address. If you do not specify a schema, Mockit returns an empty body. It forces developers to correct the specification rather than patching the mock.

### 3. Zero Invention
Mockit operates purely as an OpenAPI contract simulator. It does not contain natural language processing or artificial intelligence to guess developer intent. It executes static mathematical calculations and deterministic pseudo-random generations based strictly on the parameters specified in your document.

---

## Developer Expectations

### What to Expect
*   **Absolute Contract Compliance:** A request that passes validation under Mockit will pass validation against any backend that strictly adheres to the same OpenAPI document.
*   **Dynamic, Ephemeral CRUD:** Memory state is mutated at runtime when you send `POST`, `PUT`, `PATCH`, or `DELETE` requests. Creating an item will immediately make it retrievable in subsequent `GET` requests for the duration of the server session.
*   **Deterministic Mocking:** Providing a specific seed flag guarantees that every generated string, number, and array will be mathematically identical across different machines and restarts.
*   **Instant Hot Reloading:** Opting into watch mode registers a file listener that hot-swaps active routes inside the running network socket in milliseconds upon saving the contract.

### What NOT to Expect
*   **Persistent Storage:** Mockit does not save state to disk. Restarting the server completely purges the ephemeral memory.
*   **Custom Scripting or DSLs:** You cannot inject custom JavaScript, middleware, or conditional logic to alter how the server behaves. All behaviors must be represented through OpenAPI primitives (like `examples`, `enums`, or `formats`).
*   **Semantic Guessing:** Mockit does not scan field names (like `username` or `zip_code`) to guess appropriate dummy data. If a field lacks structural constraint rules (such as `minimum`, `maximum`, or `format`), it generates generic primitives.

---

## Command Line Interface

Mockit is run by passing the path of your OpenAPI specification as the final argument:

```bash
mockit [flags] path/to/spec.yaml
```

### Reference Flags

*   `--port <int>`  
    The network port the server binds to (default: `8080`).
*   `--watch`  
    An opt-in filesystem listener that watches the spec file. Upon save, it re-evaluates the AST and updates routing dynamically without dropping connections.
*   `--strict`  
    Enforces strict validation. Disables lenient HTTP fallbacks, returns empty bodies when schemas are omitted, and strictly enforces `406 Not Acceptable` media boundaries.
*   `--seed <int>`  
    An integer seed for the pseudo-random generator. Used to ensure generated mock payloads are completely reproducible across automated testing runs.
*   `--state-policy <string>`  
    Defines the system behavior when a requested collection state is empty.
    *   `auto-seed` (default): Automatically populates the state by generating objects matching the schema constraints.
    *   `empty`: Returns an empty collection.
    *   `fail`: Returns a `500 Internal Server Error` to simulate missing state scenarios.
*   `--max-generated-array-size <int>`  
    Sets an upper safety ceiling (default: `1000`) for the number of items generated in arrays, preventing memory exhaustion when schemas contain excessively large `minItems` directives.

---

## License & Purchase

Mockit is distributed as a commercial, 100% DRM-free standalone binary. You can purchase a lifetime license for a one-off fee of approximately $19.

**[Get Mockit Standalone Binary ($19)](https://paystack.com/buy/mockit-openapi)**

_Note: Payments are securely processed via Paystack (a Stripe company). The checkout page will display the price in **NGN (Nigerian Naira)** (approximately 26,000 NGN). Your bank or card issuer will automatically convert this to your local currency at checkout._

_All purchases include binaries for macOS (Intel & Apple Silicon), Linux (amd64 & arm64), and Windows._

### Terms of Delivery & Refund Policy
* **Delivery:** Upon successful payment, you will receive an immediate digital download link for the pre-compiled binaries via your provided email address.
* **Refund Policy:** Because Mockit is a downloadable software tool, all sales are generally final. However, we offer a full **14-day money-back guarantee** if the tool experiences a critical technical incompatibility with your environment that our support team cannot resolve. For refund requests, contact duke.williamsx@gmail.com.
