# Agentic Proxy: Liberating Free-Tier LLM Access

## The Problem
Developers in the agentic AI community frequently encounter a frustrating limitation: free-tier large language models (LLMs) are often restricted to usage within specific official applications (e.g., OpenCode TUI, certain web UIs). This "walled garden" approach prevents developers from integrating these models into their custom agentic systems, development environments, or experimental setups. This significantly hinders rapid prototyping, experimentation, and the broader adoption of these powerful models in diverse agentic architectures.

## Why this Project Shape/Stack?
This prototype is a lightweight **Go service** acting as a **local HTTP proxy**.
*   **Go:** Chosen for its excellent concurrency primitives, small footprint, and ease of deployment as a standalone binary, making it ideal for a local proxy. It's performant enough to handle local requests without noticeable latency.
*   **HTTP Proxy:** This is the most natural and least intrusive way to intercept and modify network requests. By acting as a local proxy, it can transparently intercept requests intended for a restricted API endpoint, modify them if necessary (e.g., inject headers, transform payloads), and forward them to the actual upstream service. This allows existing tools and codebases to "just work" by simply configuring them to use the local proxy.
*   **No LLM SDKs in Core Logic:** The core proxy logic is completely agnostic to the specific LLM provider. It operates purely at the HTTP request/response level. This keeps the solution provider-agnostic and focused on the infrastructure problem.

## Setup and Usage (Zero API Keys Required)

This prototype provides a proof-of-concept for how such a proxy would work. It includes fixture data to simulate responses from a restricted LLM API, demonstrating its functionality without requiring any actual LLM provider keys.

1.  **Clone the repository:**
    ```bash
    git clone <this-repo-url>
    cd agentic-proxy
    ```

2.  **Run the proxy:**
    ```bash
    go run main.go
    ```
    The proxy will start on `http://localhost:8080`.

3.  **Configure your client to use the proxy:**
    For `curl` (example):
    ```bash
    # This curl command normally would go directly to the restricted endpoint.
    # We're simulating this by targeting a non-existent URL that the proxy will intercept.
    # The proxy will then return our fixture data.
    
    # Using the proxy:
    http_proxy=http://localhost:8080 curl -X POST http://restricted-llm.example.com/v1/chat/completions \
        -H "Content-Type: application/json" \
        -d '{"messages": [{"role": "user", "content": "Tell me a story."}]}'
    
    # You should see the fixture response printed, demonstrating the proxy intercepted and responded.
    ```
    
    For other applications/environments, you would typically set the `HTTP_PROXY` and `HTTPS_PROXY` environment variables to `http://localhost:8080` or configure your client library to use a proxy.

## Optional Real-LLM Adapter
This prototype **does not include a real-LLM adapter**. The core problem is about overcoming access restrictions, not about directly interacting with an LLM via an SDK. The proxy's role is to *enable* existing client code (which might use an LLM SDK or direct HTTP calls) to reach a service it otherwise couldn't.

If a real LLM adapter were to be added, it would involve:
1.  Identifying a specific free-tier LLM that has such restrictions (e.g., via a proprietary TUI).
2.  Reverse-engineering the HTTP requests made by that official application.
3.  Implementing logic within the proxy to mimic those requests, potentially adding specific headers, authentication tokens (if obtainable, which is often the restriction), or transforming the payload to match the expected format of the *actual* upstream service.
4.  This would be a highly specific, LLM-provider-dependent implementation, separate from the generic proxy logic, and would likely be brittle as official APIs change.

The current setup with fixture data demonstrates the *mechanism* of interception and response modification, which is the core value proposition for overcoming access restrictions.
