# Copilot Tokens Saver — Instructions

## Why This Approach

GitHub Copilot's consumption-based pricing model bills **input and output tokens separately**, with output tokens approximately **5× more expensive** than input tokens. The dominant source of unnecessary output token spend is not the answer itself — it is the prose surrounding it: introductions, restatements of the user's question, hedging language, filler words, and conclusions that add no signal.

The straightforward fix is to tell Copilot to stop producing that prose. The challenge is making that instruction persistent and universal without requiring every developer to manually select a custom agent on each session.

### Expected Token Savings

| Scenario | Output token reduction |
|---|---|
| General Q&A, code review | 20–35% (filler prose eliminated) |
| File analysis (YAML, JSON, logs, configs) | **70–90%** (no raw file echo) |
| Weighted average across typical dev workflow | ~40–60% |

The largest single source of waste is Copilot echoing back entire input files verbatim — reading a YAML config, analyzing a log, reviewing a JSON schema. Without the no-echo rule, Copilot re-outputs the full file content as "context" before answering. A 500-line YAML file ≈ 3,000–5,000 output tokens per operation. With the rule: a 2–3 line summary. That alone drives the 70–90% reduction on those operations.

| Factor | Estimate |
|---|---|
| Prompt overhead (input) | ~300 tokens per conversation |
| Output tokens saved per interaction (avg) | ~150–2,000+ depending on operation |
| Net saving per interaction | always positive (output at 5× cost) |

**Per developer at 50 interactions/day:** ~$2–10/month saved.
**Team of 20 devs:** ~$40–200/month — from eliminating prose and file echoing that adds no information.

The solution is to **load a Copilot Tokens Saver prompt file as a default instruction extension** via `.github/copilot-instructions.md`. 

This is technically a hack: Copilot resolves the `#file:` reference in the instruction file and injects the full prompt into every conversation automatically, without any manual agent switching. The behavior becomes a repository-level default that applies to all team members across all Copilot surfaces (IDE chat, PR reviews, inline suggestions where applicable).

## How It Works

```
.github/
  copilot-instructions.md       ← loaded automatically by Copilot for every chat
  tokens-saver/
    tokens-saver.prompt.md        ← the actual Copilot Tokens Saver rules, injected via #file: reference
```

`.github/copilot-instructions.md` contains a single line:

```
**Default behavior:** Apply Copilot Tokens Saver for every response — token-minimal, caveman speech. #file:tokens-saver/tokens-saver.prompt.md
```

Copilot resolves the `#file:` path relative to the instruction file's directory (`.github/`), so `tokens-saver/tokens-saver.prompt.md` resolves correctly to `.github/tokens-saver/tokens-saver.prompt.md`.

## What Copilot Tokens Saver Does

Enforces token-minimal responses in every conversation:

- **No filler prose** — no intros, outros, hedging, or restatements of the question
- **Answer first** — lead with the top finding, elaborate only if asked
- **Bullets over paragraphs** — structured output, not narrative
- **Caveman fragments OK** — drops articles and filler words where meaning is preserved
- **Never truncates deliverables** — code, configs, scripts, and docs always complete

## Usage

Download the shared prompts into your project:

```sh
git clone https://git.dhl.com/Software-Reuse/copilot-tokens-saver.git .github/tokens-saver
```

### Configure Copilot Instructions

If `.github/copilot-instructions.md` does not exist, copy it from `tokens-saver/copilot-instructions.md`:

```sh
cp tokens-saver/copilot-instructions.md .github/copilot-instructions.md
```

If `.github/copilot-instructions.md` already exists, add this line:

```
**Default behavior:** Apply Copilot Tokens Saver for every response — token-minimal, caveman speech. #file:tokens-saver/tokens-saver.prompt.md
```

Copilot Tokens Saver activates automatically for all Copilot chat sessions in that repo — no agent switching required.

Type `/about tokens-saver` in any Copilot chat session to confirm Copilot Tokens Saver is active.
Expected response: `Copilot Tokens Saver for GitHub Copilot v1.0.0 — reduces AI token consumption for day-to-day development. Caveman on. Cmd: /about tokens-saver`

### Codex

Copy `AGENTS.md` to `~/.codex/AGENTS.md`:

```sh
cp .github/tokens-saver/AGENTS.md ~/.codex/AGENTS.md
```

If `~/.codex/AGENTS.md` already exists, append the `# Personal Coding Rules` section:

```sh
cat .github/tokens-saver/AGENTS.md >> ~/.codex/AGENTS.md
```

### Claude Code

Copy `AGENTS.md` to `~/.claude/AGENTS.md`:

```sh
cp .github/tokens-saver/AGENTS.md ~/.claude/AGENTS.md
```

If `~/.claude/AGENTS.md` already exists, append the `# Personal Coding Rules` section:

```sh
cat .github/tokens-saver/AGENTS.md >> ~/.claude/AGENTS.md
```
