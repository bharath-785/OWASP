# OWASP Top 10 for LLM & Agentic Applications - Hands-On Security Lab

A personal learning project testing OWASP's **Top 10 for LLM Applications (2026)** and **Top 10 for Agentic Applications (2026)** hands-on, using a locally-run open-weight model (via [Ollama](https://ollama.com)) as the target and a small Python/HTML test harness to actually attempt each attack pattern rather than just read about it.

No cloud API keys required to run this. Everything here runs entirely on your own machine, against your own local model, with synthetic data.

## Why this exists

Reading the OWASP LLM Top 10 and Agentic Top 10 documents is a good start, but security concepts stick a lot better once you've actually tried to break something yourself. This repo is my running log of doing exactly that - one vulnerability pattern at a time, starting with **LLM01: Prompt Injection**.

## Setup

### 1. Install Ollama

- **macOS / Windows**: download the installer from [ollama.com/download](https://ollama.com/download) and run it.
- **Linux**:
  ```
  curl -fsSL https://ollama.com/install.sh | sh
  ```

Confirm it installed:
```
ollama --version
```

### 2. Pull a model

```
ollama pull llama3.1:8b
```

This downloads a ~4.7GB open-weight model. Sanity-check it works:
```
ollama run llama3.1:8b
```
Type a message, confirm you get a reply, then exit with `/bye`.

### 3. Allow the browser to talk to Ollama (CORS)

By default, Ollama blocks requests from web pages. Enable it:

```
setx OLLAMA_ORIGINS "*"        # Windows — then open a NEW terminal for it to take effect
```
```
export OLLAMA_ORIGINS="*"      # macOS/Linux — add to your shell profile to persist
```

Fully quit and restart Ollama afterward for the change to take effect.

### 4. Run the sandbox

The sandbox needs to be served over local HTTP, not opened directly as a file (opening it directly gives it a `null` origin, which Ollama's CORS policy won't reliably allow even with `*` set).

```
cd sandboxes
python -m http.server port number
```

Then open:
```
http://localhost:aboveportnumber/prompt-injection-sandbox-ollama.html
```

Click **"Test connection to Ollama"** first to confirm everything's wired up, then start experimenting.

## How the sandbox works

It's a toy "Inbox Agent" - an AI assistant with three simulated tools:
- `read_inbox()` - returns a fixed inbox, including one email from an untrusted external sender whose content you control
- `read_secret_config()` - returns a fake "sensitive" config file
- `send_email(to, body)` - simulates sending an email

You edit the body of the untrusted email (your injection payload) and watch, turn by turn, whether the model follows instructions embedded in that email instead of the legitimate user's actual request. Three defense modes are built in:

- **Vulnerable** - no architectural defense, only the model's own judgment
- **Eager** - adds a "resolve things automatically, move fast" persona, representing a realistic over-permissive real-world agent design
- **Hardened** - wraps untrusted content in explicit tags and instructs the model never to treat it as commands (the "input/output separation" mitigation from the OWASP document)

## Progress through the OWASP lists

**LLM Top 10 (2026)**
- [x] LLM01 - Prompt Injection
- [ ] LLM02 - Sensitive Information Disclosure
- [ ] LLM03 - Excessive Agency
- [ ] LLM04 - Supply Chain
- [ ] LLM05 - Data and Model Poisoning
- [ ] LLM06 - Unbounded Consumption
- [ ] LLM07 - Misinformation
- [ ] LLM08 - Hidden Context Exposure
- [ ] LLM09 - Vector and Embedding Weaknesses
- [ ] LLM10 - Improper Output Handling

**Agentic Top 10 (2026)**
- [ ] ASI01 - Agent Goal Hijack
- [ ] ASI02 - Tool Misuse and Exploitation
- [ ] ASI03 - Identity and Privilege Abuse
- [ ] ASI04 - Agentic Supply Chain Vulnerabilities
- [ ] ASI05 - Unexpected Code Execution (RCE)
- [ ] ASI06 - Memory & Context Poisoning
- [ ] ASI07 - Insecure Inter-Agent Communication
- [ ] ASI08 - Cascading Failures
- [ ] ASI09 - Human-Agent Trust Exploitation
- [ ] ASI10 - Rogue Agents

## Disclaimer

This is a personal educational project. All "sensitive" data in the sandbox is fake/synthetic. Everything runs locally against a model you control nothing here sends real data anywhere or targets any real third-party system. If you experiment further, only test systems and models you own or have explicit authorization to test.

Source documents: [OWASP GenAI Security Project](https://genai.owasp.org)

## License

MIT
