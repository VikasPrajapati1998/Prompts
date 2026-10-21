# Ollama Cloud & OpenCode — Command Cheat Sheet

> **Writing** — A concise, command-focused version with minimal explanation, designed as a technical cheat sheet.

## Contents

| # | Section | What it covers |
| --- | --- | --- |
| 1 | [Ollama Cloud](#1-ollama-cloud) | Sign in, run models, pull model references |
| 2 | [Claude Code + Ollama](#2-claude-code--ollama) | Launch Claude Code against Ollama |
| 3 | [OpenCode + Ollama](#3-opencode--ollama) | Launch and configure OpenCode via Ollama |
| 4 | [OpenCode — Direct Provider Setup](#4-opencode--direct-provider-setup) | Connect OpenCode straight to a provider |
| 5 | [Other OpenCode Cloud Providers](#5-other-opencode-cloud-providers) | Alternative providers and free tiers |
| 6 | [Recommended Models](#6-recommended-models) | Which model to use for what |
| 7 | [References](#7-references) | Official docs and links |

---

## 1. Ollama Cloud

### Authentication

```bash
ollama signin
```

### Run models

```bash
ollama run gemma4:31b-cloud
ollama run beam:cloud
ollama run gpt-oss:20b-cloud
ollama run gpt-oss:120b-cloud
```

### Pull cloud model references

```bash
ollama pull gemma4:31b-cloud
ollama pull beam:cloud
```

### Exit model

```text
/bye
```

---

## 2. Claude Code + Ollama

### Launch

```bash
ollama launch claude
```

### Select model

```bash
ollama launch claude --model gemma4:31b-cloud
ollama launch claude --model beam:cloud
```

---

## 3. OpenCode + Ollama

### Launch

```bash
ollama launch opencode
```

### Select model

```bash
ollama launch opencode --model gemma4:31b-cloud
ollama launch opencode --model beam:cloud
```

### Configure integration

```bash
ollama launch opencode --config
```

---

## 4. OpenCode — Direct Provider Setup

### Start OpenCode

```bash
opencode
```

### Inside OpenCode

```text
/connect
/models
```

Select Ollama Cloud in `/connect`, authenticate, then select a model through `/models`.

### OpenCode CLI

```bash
opencode models
opencode auth login
```

---

## 5. Other OpenCode Cloud Providers

```bash
opencode
```

Inside the TUI:

```text
/connect
/models
```

Provider options and free tiers: <https://opencode.ai/docs/providers/>

---

## 6. Recommended Models

| Model | Use |
| --- | --- |
| `gemma4:31b-cloud` | General coding |
| `beam:cloud` | Agentic coding |
| `gpt-oss:20b-cloud` | Coding and reasoning |
| `gpt-oss:120b-cloud` | Complex tasks |

Check current availability: <https://ollama.com/search?c=cloud>

---

## 7. References

| Resource | Link |
| --- | --- |
| Ollama | <https://docs.ollama.com/> |
| Ollama Cloud | <https://ollama.com/search?c=cloud> |
| OpenCode | <https://opencode.ai/docs/> |
| OpenCode providers | <https://opencode.ai/docs/providers/> |
| Ollama + OpenCode | <https://github.com/ollama/ollama/blob/main/docs/integrations/opencode.mdx> |
