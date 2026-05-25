# @smooai/agent

**Rust-native AI agent framework** — checkpointing, tool system, LLM client. Shared across [Big Smooth](https://github.com/SmooAI/smooth) and the smooai platform.

Inspired by LangGraph, CrewAI, and Agno — purpose-built for orchestrated agent workloads with security-first design.

## Repository layout

This is a multi-language SmooAI package. Each language has its own subdirectory:

| Directory | Language | Status |
| --------- | -------- | ------ |
| [`rust/`](./rust) | Rust (primary) | Active — crate `smooai-agent` |
| [`go/`](./go) | Go | Planned |
| [`python/`](./python) | Python | Planned |
| [`dotnet/`](./dotnet) | C# / .NET | Planned |

The Rust implementation is the source of truth; bindings in other languages will mirror its surface.

## Rust quick start

Add to your `Cargo.toml`:

```toml
[dependencies]
smooai-agent = { git = "https://github.com/SmooAI/agent.git", branch = "main" }
```

```rust
use smooai_agent::{Agent, AgentConfig, LlmConfig, Tool, ToolRegistry, ToolSchema};
use async_trait::async_trait;

struct GetWeather;

#[async_trait]
impl Tool for GetWeather {
    fn schema(&self) -> ToolSchema {
        ToolSchema {
            name: "get_weather".into(),
            description: "Get current weather for a city".into(),
            parameters: serde_json::json!({
                "type": "object",
                "properties": { "city": { "type": "string" } },
                "required": ["city"]
            }),
        }
    }

    async fn execute(&self, args: serde_json::Value) -> anyhow::Result<String> {
        let city = args["city"].as_str().unwrap_or("unknown");
        Ok(format!("Weather in {city}: 72F, sunny"))
    }
}

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let llm = LlmConfig {
        api_url: "https://api.openai.com/v1".into(),
        api_key: std::env::var("OPENAI_API_KEY")?,
        model: "gpt-4o".into(),
        ..Default::default()
    };

    let config = AgentConfig::new("assistant", "You are a helpful assistant.", llm)
        .with_max_iterations(10)
        .with_parallel_tools(true);

    let mut registry = ToolRegistry::new();
    registry.register(GetWeather);

    let mut agent = Agent::new(config, registry);
    let events = agent.run("What's the weather in Tokyo?").await?;

    for event in events { println!("{event:?}"); }
    Ok(())
}
```

## Features

- **LLM Client** — OpenAI-compatible and Anthropic API support with streaming, retry policies, and rate limit handling
- **Tool System** — Trait-based tools with pre/post hooks for surveillance, secret detection, and prompt injection guards
- **Agent Loop** — Observe-think-act cycle with configurable iteration limits, parallel tool execution, and cost budgets
- **Checkpointing** — Pluggable checkpoint stores for session resume and fault tolerance (in-memory + sqlite via `sqlite` feature)
- **Conversation Management** — Token-aware context window with compaction strategies
- **Memory & Knowledge** — Trait-based memory and knowledge base integration
- **Cost Tracking** — Per-model pricing, token budgets, and budget enforcement

## License

MIT — see [LICENSE](./LICENSE).

## Links

- [smoo.ai](https://smoo.ai) — the product
- [github.com/SmooAI](https://github.com/SmooAI) — other open-source packages
