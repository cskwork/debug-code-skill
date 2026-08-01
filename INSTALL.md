# Install debug-code

<details>
<summary><strong>Claude Code</strong></summary>

### Install

```bash
claude plugin marketplace add cskwork/debug-code-skill
claude plugin install debug-code@debug-code
```

Type `/debug-code`.

### Verify

```bash
claude plugin list
```

### Update

```bash
claude plugin marketplace update debug-code
```

### Uninstall

```bash
claude plugin uninstall debug-code
claude plugin marketplace remove debug-code
```

</details>

<details>
<summary><strong>Codex</strong></summary>

### Install

```bash
codex plugin marketplace add cskwork/debug-code-skill --ref main
codex plugin add debug-code@debug-code
```

Type `$debug-code`.

### Verify

```bash
codex plugin list
```

### Uninstall

```bash
codex plugin remove debug-code
codex plugin marketplace remove debug-code
```

</details>

<details>
<summary><strong>Gemini CLI</strong></summary>

### Install (extension, always-on)

```bash
gemini extensions install https://github.com/cskwork/debug-code-skill
```

### Install (command, opt-in)

```bash
mkdir -p ~/.gemini/commands
curl -fsSL https://raw.githubusercontent.com/cskwork/debug-code-skill/main/skills/debug-code-skill/agents/gemini.toml \
  -o ~/.gemini/commands/debug-code.toml
```

Type `/debug-code` in a new session.

### Verify

```bash
gemini extensions list
```

### Uninstall

```bash
gemini extensions uninstall debug-code
```

</details>

<details>
<summary><strong>Cursor, OpenCode, Amp, and other agent-skills harnesses</strong></summary>

### Install

```bash
npx skills add cskwork/debug-code-skill
npx skills add cskwork/debug-code-skill -g
```

Type `/debug-code` in a new agent chat.

### Verify

```bash
npx skills list
```

### Update

```bash
npx skills update debug-code
```

### Uninstall

```bash
npx skills remove debug-code
```

</details>

<details>
<summary><strong>Antigravity (agy)</strong></summary>

### Install

```bash
agy plugin install https://github.com/cskwork/debug-code-skill
```

### Verify

```bash
agy plugin list
```

### Uninstall

```bash
agy plugin uninstall debug-code
```

</details>
