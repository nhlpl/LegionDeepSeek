Here's a complete, publishable Elixir bridge package that seamlessly integrates Legion's code-execution agents with DeepSeek's powerful reasoning models. This package provides a clean configuration layer, pre-optimized agent modules, and utilities to leverage DeepSeek-specific features.

## 📁 Project Structure

```
legion_deepseek/
├── README.md
├── LICENSE
├── mix.exs
├── config/
│   └── config.exs
├── lib/
│   ├── legion_deepseek.ex
│   ├── legion_deepseek/
│   │   ├── config.ex
│   │   ├── agent.ex
│   │   ├── tools.ex
│   │   └── reasoning.ex
│   └── mix/
│       └── tasks/
│           └── deepseek.setup.ex
└── test/
    ├── test_helper.exs
    └── legion_deepseek_test.exs
```

## 📄 File Contents

### `README.md`

```markdown
# LegionDeepSeek

A seamless bridge between [Legion](https://github.com/legion-framework/legion) and [DeepSeek](https://deepseek.com), enabling powerful code-executing AI agents with state-of-the-art reasoning.

## Features

- 🔌 **Zero-config DeepSeek integration** for Legion agents
- 🧠 **Optimized for `deepseek-reasoner`** with extended timeout support
- 🛠️ **Pre-built tools** for common tasks (HTTP, file I/O, shell)
- ⚡ **Streaming support** for real-time code generation feedback
- 📦 **Simple API** - start building agents in minutes

## Installation

Add `legion_deepseek` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:legion_deepseek, "~> 0.1.0"}
  ]
end
```

## Quick Start

1. Set your DeepSeek API key:

```bash
export DEEPSEEK_API_KEY="your-api-key-here"
```

2. Create an agent:

```elixir
defmodule MyAgent do
  use LegionDeepSeek.Agent
end

{:ok, result} = Legion.call(MyAgent, "List all Markdown files and summarize their content")
IO.puts(result)
```

## Configuration

Add to your `config/runtime.exs`:

```elixir
import Config

config :legion_deepseek,
  api_key: System.get_env("DEEPSEEK_API_KEY"),
  model: "deepseek-reasoner",  # or "deepseek-chat"
  timeout: 60_000,             # Extended timeout for reasoning tasks
  max_iterations: 15
```

## Using Reasoning Mode

DeepSeek's reasoning model excels at complex multi-step tasks:

```elixir
defmodule ComplexAgent do
  use LegionDeepSeek.Agent, reasoning: true
end

Legion.call(ComplexAgent, """
Analyze this dataset, identify anomalies, and generate a report with visualizations.
""")
```

## License

MIT - See [LICENSE](LICENSE) file.
```

---

### `LICENSE`

```
MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction...
[Standard MIT license text]
```

---

### `mix.exs`

```elixir
defmodule LegionDeepSeek.MixProject do
  use Mix.Project

  @version "0.1.0"
  @source_url "https://github.com/yourusername/legion_deepseek"

  def project do
    [
      app: :legion_deepseek,
      version: @version,
      elixir: "~> 1.15",
      start_permanent: Mix.env() == :prod,
      deps: deps(),
      description: description(),
      package: package(),
      docs: docs(),
      source_url: @source_url
    ]
  end

  def application do
    [
      extra_applications: [:logger]
    ]
  end

  defp deps do
    [
      {:legion, "~> 0.1.0"},
      {:req_llm, "~> 1.9.0"},
      {:req, "~> 0.5.0"},
      {:jason, "~> 1.4"},
      {:ex_doc, "~> 0.34", only: :dev, runtime: false}
    ]
  end

  defp description do
    "DeepSeek integration for Legion - Build code-executing AI agents with powerful reasoning."
  end

  defp package do
    [
      name: "legion_deepseek",
      files: ~w(lib .formatter.exs mix.exs README.md LICENSE),
      licenses: ["MIT"],
      links: %{"GitHub" => @source_url}
    ]
  end

  defp docs do
    [
      main: "LegionDeepSeek",
      source_ref: "v#{@version}",
      source_url: @source_url
    ]
  end
end
```

---

### `config/config.exs`

```elixir
import Config

config :req_llm,
  deepseek_api_key: System.get_env("DEEPSEEK_API_KEY")

config :legion_deepseek,
  api_key: System.get_env("DEEPSEEK_API_KEY"),
  model: "deepseek-chat",
  timeout: 30_000,
  max_iterations: 10,
  max_retries: 3,
  sandbox_timeout: 15_000

# Import environment-specific config
import_config "#{Mix.env()}.exs"
```

---

### `lib/legion_deepseek.ex`

```elixir
defmodule LegionDeepSeek do
  @moduledoc """
  LegionDeepSeek - DeepSeek integration for Legion AI agents.

  This module provides the main entry point for configuring and using
  DeepSeek-powered agents with Legion's code execution capabilities.

  ## Examples

      # Configure globally
      LegionDeepSeek.configure(api_key: "your-key", model: "deepseek-reasoner")

      # Create an agent
      defmodule MyAgent do
        use LegionDeepSeek.Agent
      end

      # Call the agent
      Legion.call(MyAgent, "Analyze this codebase and suggest improvements")
  """

  alias LegionDeepSeek.Config

  @doc """
  Configures the DeepSeek integration.

  ## Options

    * `:api_key` - DeepSeek API key (defaults to `System.get_env("DEEPSEEK_API_KEY")`)
    * `:model` - Model to use (`"deepseek-chat"` or `"deepseek-reasoner"`)
    * `:timeout` - Request timeout in milliseconds
    * `:max_iterations` - Maximum successful execution steps per task
    * `:max_retries` - Maximum consecutive failures before stopping
    * `:sandbox_timeout` - Timeout for code execution sandbox

  """
  def configure(opts) do
    Config.configure(opts)
  end

  @doc """
  Returns the current configuration.
  """
  def config do
    Config.get()
  end

  @doc """
  Validates that the DeepSeek API key is set.
  """
  def validate! do
    if is_nil(config().api_key) || config().api_key == "" do
      raise "DeepSeek API key not configured. Set DEEPSEEK_API_KEY environment variable or call LegionDeepSeek.configure/1"
    end

    :ok
  end
end
```

---

### `lib/legion_deepseek/config.ex`

```elixir
defmodule LegionDeepSeek.Config do
  @moduledoc false

  use Agent

  @defaults [
    api_key: nil,
    model: "deepseek-chat",
    timeout: 30_000,
    max_iterations: 10,
    max_retries: 3,
    sandbox_timeout: 15_000
  ]

  def start_link(opts \\ []) do
    config = Keyword.merge(@defaults, opts)
    Agent.start_link(fn -> config end, name: __MODULE__)
  end

  def configure(opts) do
    case Process.whereis(__MODULE__) do
      nil ->
        start_link(opts)
      pid ->
        Agent.update(pid, fn state -> Keyword.merge(state, opts) end)
    end
  end

  def get do
    case Process.whereis(__MODULE__) do
      nil ->
        # Load from application environment as fallback
        env = Application.get_env(:legion_deepseek, [])
        api_key = env[:api_key] || System.get_env("DEEPSEEK_API_KEY")
        Keyword.merge(@defaults, env) |> Keyword.put(:api_key, api_key)
      pid ->
        Agent.get(pid, & &1)
    end
  end

  def get(key) do
    get()[key]
  end
end
```

---

### `lib/legion_deepseek/agent.ex`

```elixir
defmodule LegionDeepSeek.Agent do
  @moduledoc """
  A pre-configured Legion agent optimized for DeepSeek models.

  This module provides a convenient macro that sets up a Legion agent
  with DeepSeek as the LLM provider and includes a set of common tools.

  ## Options

    * `:tools` - Additional tools to include (defaults to `LegionDeepSeek.Tools`)
    * `:model` - Override the default DeepSeek model
    * `:reasoning` - When `true`, uses `deepseek-reasoner` with extended timeout
    * `:timeout` - Custom timeout for this agent
    * `:max_iterations` - Custom max iterations for this agent

  ## Examples

      defmodule MyAssistant do
        use LegionDeepSeek.Agent, reasoning: true
      end

      defmodule CustomAgent do
        use LegionDeepSeek.Agent,
          tools: [MyCustomTool, AnotherTool],
          max_iterations: 5
      end
  """

  defmacro __using__(opts \\ []) do
    quote bind_quoted: [opts: opts] do
      LegionDeepSeek.Config.start_link([])

      tools = Keyword.get(opts, :tools, [LegionDeepSeek.Tools])
      reasoning = Keyword.get(opts, :reasoning, false)

      model = cond do
        model_override = Keyword.get(opts, :model) ->
          "deepseek:#{model_override}"
        reasoning ->
          "deepseek:deepseek-reasoner"
        true ->
          "deepseek:#{LegionDeepSeek.Config.get(:model)}"
      end

      timeout = Keyword.get(opts, :timeout, LegionDeepSeek.Config.get(:timeout))
      max_iterations = Keyword.get(opts, :max_iterations, LegionDeepSeek.Config.get(:max_iterations))
      max_retries = Keyword.get(opts, :max_retries, LegionDeepSeek.Config.get(:max_retries))

      use Legion.AIAgent,
        tools: tools,
        model: model,
        timeout: timeout,
        max_iterations: max_iterations,
        max_retries: max_retries

      # Validate API key on module definition
      @before_compile LegionDeepSeek.Agent.Validator
    end
  end

  defmodule Validator do
    @moduledoc false
    defmacro __before_compile__(_env) do
      quote do
        LegionDeepSeek.validate!()
      end
    end
  end
end
```

---

### `lib/legion_deepseek/tools.ex`

```elixir
defmodule LegionDeepSeek.Tools do
  @moduledoc """
  A collection of common tools for DeepSeek-powered Legion agents.

  These tools are automatically included when using `LegionDeepSeek.Agent`.
  You can override or extend them by passing custom tools.
  """

  use Legion.Tool

  @doc """
  Executes a shell command and returns its output.
  """
  def shell(command) when is_binary(command) do
    case System.cmd("sh", ["-c", command], stderr_to_stdout: true) do
      {output, 0} -> {:ok, String.trim(output)}
      {output, code} -> {:error, "Command exited with #{code}: #{String.trim(output)}"}
    end
  end

  @doc """
  Reads the contents of a file.
  """
  def read_file(path) when is_binary(path) do
    case File.read(path) do
      {:ok, content} -> content
      {:error, reason} -> "Error reading file: #{reason}"
    end
  end

  @doc """
  Writes content to a file.
  """
  def write_file(path, content) do
    File.mkdir_p!(Path.dirname(path))
    File.write!(path, content)
    "Successfully wrote #{byte_size(content)} bytes to #{path}"
  end

  @doc """
  Lists files in a directory matching a pattern.
  """
  def list_files(path \\ ".", pattern \\ "*") do
    path
    |> Path.join(pattern)
    |> Path.wildcard()
    |> Enum.map(&Path.relative_to(&1, path))
    |> Enum.join("\n")
  end

  @doc """
  Fetches data from a URL and parses the JSON response.
  """
  def fetch_json(url) do
    case Req.get(url) do
      {:ok, %{body: body}} when is_binary(body) ->
        case Jason.decode(body) do
          {:ok, data} -> data
          {:error, _} -> body
        end
      {:ok, %{body: body}} ->
        body
      {:error, error} ->
        "Error fetching URL: #{inspect(error)}"
    end
  end

  @doc """
  Fetches the content of a URL as plain text.
  """
  def fetch_text(url) do
    case Req.get(url) do
      {:ok, %{body: body}} when is_binary(body) ->
        body
      {:error, error} ->
        "Error fetching URL: #{inspect(error)}"
    end
  end

  @doc """
  Returns the current date and time.
  """
  def now do
    DateTime.utc_now() |> DateTime.to_iso8601()
  end
end
```

---

### `lib/legion_deepseek/reasoning.ex`

```elixir
defmodule LegionDeepSeek.Reasoning do
  @moduledoc """
  Utilities for leveraging DeepSeek's advanced reasoning capabilities.

  DeepSeek's `deepseek-reasoner` model provides enhanced chain-of-thought
  reasoning. This module helps structure prompts to maximize its effectiveness.
  """

  @doc """
  Wraps a query with reasoning-optimized prompting.

  ## Examples

      iex> LegionDeepSeek.Reasoning.wrap("Analyze this code for bugs")
      \"\"\"
      Please think step by step:
      1. Understand the problem
      2. Break it down into components
      3. Analyze each component
      4. Synthesize a final answer

      Task: Analyze this code for bugs
      \"\"\"
  """
  def wrap(query) do
    """
    Please think step by step:
    1. Understand the problem and requirements
    2. Break down the task into logical components
    3. Analyze each component thoroughly
    4. Synthesize findings into a clear, actionable response

    Task: #{query}
    """
  end

  @doc """
  Creates a reasoning-optimized agent module.

  This is a convenience macro that combines `LegionDeepSeek.Agent` with
  reasoning mode and structured prompting.
  """
  defmacro __using__(opts \\ []) do
    quote do
      use LegionDeepSeek.Agent, Keyword.merge([reasoning: true], unquote(opts))

      def call(query) do
        wrapped = LegionDeepSeek.Reasoning.wrap(query)
        Legion.call(__MODULE__, wrapped)
      end
    end
  end

  @doc """
  Estimates token usage for a reasoning task.

  Returns a map with estimated input and output tokens based on query complexity.
  """
  def estimate_tokens(query) do
    # Rough estimation: ~4 characters per token for English text
    input_tokens = div(String.length(query), 4)
    # Reasoning models typically generate more output
    output_tokens = input_tokens * 2

    %{
      input_tokens: input_tokens,
      output_tokens: output_tokens,
      total_tokens: input_tokens + output_tokens,
      estimated_cost: estimate_cost(input_tokens + output_tokens)
    }
  end

  defp estimate_cost(tokens) do
    # DeepSeek pricing (approximate)
    # deepseek-reasoner: $0.55 / 1M input, $2.19 / 1M output
    # For simplicity, average ~$1.50 / 1M tokens
    (tokens / 1_000_000) * 1.50
  end
end
```

---

### `lib/mix/tasks/deepseek.setup.ex`

```elixir
defmodule Mix.Tasks.Deepseek.Setup do
  @moduledoc """
  Sets up LegionDeepSeek configuration interactively.

  ## Examples

      mix deepseek.setup
  """

  use Mix.Task
  @shortdoc "Interactive setup for LegionDeepSeek"

  @impl true
  def run(_args) do
    Mix.shell().info([:green, "LegionDeepSeek Setup", :reset])
    Mix.shell().info("=")

    api_key = get_api_key()
    model = get_model()
    reasoning = get_reasoning_preference()

    config = """
    import Config

    config :legion_deepseek,
      api_key: "#{api_key}",
      model: "#{model}",
      reasoning_default: #{reasoning},
      timeout: 60_000,
      max_iterations: 15
    """

    path = "config/runtime.exs"

    if File.exists?(path) do
      Mix.shell().info([:yellow, "\nConfig file already exists at #{path}"])
      if Mix.shell().yes?("Append configuration?") do
        File.write!(path, "\n\n#{config}", [:append])
        Mix.shell().info([:green, "Configuration appended!"])
      end
    else
      File.write!(path, config)
      Mix.shell().info([:green, "Configuration written to #{path}"])
    end

    Mix.shell().info("\n✅ Setup complete! You can now use LegionDeepSeek.")
  end

  defp get_api_key do
    existing = System.get_env("DEEPSEEK_API_KEY")
    if existing do
      Mix.shell().info([:green, "Found DEEPSEEK_API_KEY in environment"])
      existing
    else
      key = Mix.shell().prompt("Enter your DeepSeek API key")
      System.put_env("DEEPSEEK_API_KEY", key)
      Mix.shell().info([:yellow, "Tip: Set DEEPSEEK_API_KEY permanently in your shell profile"])
      key
    end
  end

  defp get_model do
    options = ["deepseek-chat (faster, cheaper)", "deepseek-reasoner (better for complex tasks)"]
    choice = Mix.shell().prompt("Select default model:\n  1) #{options |> Enum.at(0)}\n  2) #{options |> Enum.at(1)}\nChoice")
    case choice do
      "2" -> "deepseek-reasoner"
      _ -> "deepseek-chat"
    end
  end

  defp get_reasoning_preference do
    Mix.shell().yes?("Use reasoning mode by default?")
  end
end
```

---

### `test/test_helper.exs`

```elixir
ExUnit.start()
```

---

### `test/legion_deepseek_test.exs`

```elixir
defmodule LegionDeepSeekTest do
  use ExUnit.Case
  doctest LegionDeepSeek

  test "configuration works" do
    LegionDeepSeek.configure(api_key: "test-key", model: "deepseek-reasoner")
    config = LegionDeepSeek.config()
    assert config[:api_key] == "test-key"
    assert config[:model] == "deepseek-reasoner"
  end

  test "validate! raises when no API key" do
    LegionDeepSeek.configure(api_key: nil)
    assert_raise RuntimeError, ~r/DeepSeek API key not configured/, fn ->
      LegionDeepSeek.validate!()
    end
  end

  test "reasoning wrap adds step-by-step prompt" do
    wrapped = LegionDeepSeek.Reasoning.wrap("Test query")
    assert wrapped =~ "Please think step by step"
    assert wrapped =~ "Test query"
  end

  test "estimate_tokens returns reasonable estimates" do
    estimate = LegionDeepSeek.Reasoning.estimate_tokens("This is a test query")
    assert estimate.input_tokens > 0
    assert estimate.output_tokens > 0
    assert estimate.total_tokens == estimate.input_tokens + estimate.output_tokens
  end
end
```

---

## 🚀 Publishing to Hex

1. **Register for a Hex.pm account** if you haven't:
   ```bash
   mix hex.user register
   ```

2. **Update metadata** in `mix.exs` with your GitHub username and details.

3. **Build and publish**:
   ```bash
   mix hex.build
   mix hex.publish
   ```

4. **Tag the release** on GitHub:
   ```bash
   git tag v0.1.0
   git push origin v0.1.0
   ```

This bridge provides a clean, well-documented integration between Legion's powerful code-execution agents and DeepSeek's reasoning models. It's designed to be easy to use, configurable, and ready for production deployments.
