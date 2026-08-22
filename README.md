# Gemini CLI Configuration

Configures the Gemini CLI (v0.56.0) to use **Gemini 3.7 Flash** (`gemini-3.7-flash`) as the default model with extended thinking enabled.

## Setup

### 1. Install/upgrade the CLI

```bash
npm install -g @google/gemini-cli@latest
```

### 2. Set up authentication

Store the API key in 1Password (item: "Google API Key"), then export it:

```bash
# In ~/.zshrc
export GEMINI_API_KEY=$(op read "op://Personal/Google API Key/password")
```

Or for one-off use:

```bash
op run --env-file <(echo "GEMINI_API_KEY=op://Personal/Google API Key/password") -- gemini -p "hello"
```

### 3. settings.json

The `~/.gemini/settings.json` file contains:

- **`model.name`**: `gemini-3.7-flash` — the default model for all sessions.
- **`experimental.dynamicModelConfiguration: true`** — bypasses the CLI's hardcoded flash-model remapping. Without this, v0.56.0 silently remaps any model ending in "flash" to `gemini-3.5-flash`.
- **`modelConfigs.aliases["gemini-3.7-flash"]`** — registers the model extending `chat-base-3` (which provides `includeThoughts: true`), with `thinkingLevel: "MEDIUM"` override.
- **`modelConfigs.modelDefinitions["gemini-3.7-flash"]`** — declares it as a Gemini 3 family flash model with thinking and multimodal tool use enabled.
- **`ui.inlineThinkingMode: "full"`** — displays model thinking inline in interactive mode.

### 4. Headless / non-interactive usage

```bash
# From a trusted directory:
gemini -p "What is 2+2?" --output-format json

# From $HOME or untrusted dirs:
gemini -p "What is 2+2?" --output-format json --skip-trust
```

To trust `$HOME` permanently for headless use:

```bash
export GEMINI_CLI_TRUST_WORKSPACE=true
```

## Verification

After setup, verify the model is correct:

```bash
gemini -p "What is 2+2? Answer in one word." --output-format json --skip-trust | jq '.stats.models | keys'
```

Expected output: `["gemini-3.7-flash"]`

The `stats.models` object keys show which model actually handled the request. If you see `gemini-3.5-flash` instead, the `experimental.dynamicModelConfiguration` setting is not taking effect.

## Thinking levels

Gemini 3.7 Flash supports three thinking levels (no `minimal` — that was removed from 3.6 Flash):

| Level | Use case | Cost/latency |
|-------|----------|-------------|
| `low` | Triage, drafts, real-time chat, fast data analysis | Lowest |
| `medium` (current) | Standard coding, multi-step agents, default | Balanced |
| `high` | Hard refactors, deep tool loops, complex math | Highest |

To change the thinking level, edit `modelConfigs.aliases["gemini-3.7-flash"].modelConfig.generateContentConfig.thinkingConfig.thinkingLevel` in `settings.json`.

## Notes

- The CLI v0.56.0 predates Gemini 3.7 Flash (Aug 2026). The model ID passes through to the API correctly, but the CLI's built-in model definitions only go up to `gemini-3.5-flash`. The custom `modelConfigs` entries bridge this gap.
- When a future CLI release adds native `gemini-3.7-flash` support, the custom `modelConfigs` entries and `experimental.dynamicModelConfiguration` can be removed.
- The `GEMINI_MODEL` environment variable overrides `model.name` in settings.json if set.
- The `--model` / `-m` flag overrides everything.
