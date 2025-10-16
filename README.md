# Term Assistant

Python-powered terminal assistant for WSL (Ubuntu) and Linux. It validates and runs commands, suggests fixes, shows docs, offers Smart Mode suggestions and code snippets, supports Chat Mode (Gemini), logs usage, and lets you extend behavior with plugins.

##  Key features

- Interactive shell UI (prompt_toolkit): history, autosuggest, basic completion
- Command validation + fuzzy suggestions for unknown commands
- Option/flag hints after the first token by parsing command help/man
- On-demand docs: `:doc <cmd>` opens man/--help content
- Smart Mode (SM): natural language task suggestions, e.g. `SM: delete file`
- Code snippets: `SM: code c <query>` and nano prefill for new .c files
- Chat Mode (CM): chat with Gemini; one-shot with `CM? <prompt>`
- Analytics: command log, error rate, top commands (SQLite-backed)
- Sessions: save/load and inspect history
- Auto-interactive TTY detection and fallback rerun when a TTY is required
- Plugin system: pre/post command hooks via simple classes
- Configurable prompt, colors, aliases, plugins, interactive patterns

##  Install

Requires Python 3.8+.

### WSL/Ubuntu (recommended)

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip

cd "/mnt/d/OS project/try7"
python3 -m venv --upgrade-deps .venv
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e .

# Install ONE Gemini client (pick one)
# New client (preferred):
python -m pip install google-genai
# Or classic client:
# python -m pip install google-generativeai
```

### Windows PowerShell (project tooling only)

It’s best to run the assistant itself in WSL. For development commands in PowerShell:

```powershell
py -3 -m venv .venv
".\.venv\Scripts\Activate.ps1"
python -m pip install -U pip
python -m pip install -e .
# One Gemini client
python -m pip install google-genai
# or
# python -m pip install google-generativeai
```

##  Run

```bash
# WSL
source .venv/bin/activate
ta
```

You’ll see a prompt like:

```
Type :help for built-ins. Use Ctrl-D or :quit to exit.
user@host:/path$ 
```

##  Built-ins and modes

- `:help` — show help
- `:doc <cmd>` — show man/--help for a command
- `:stats` — error rate and top commands
- `:save <name>` / `:load <name>` — save and print sessions
- `:history [n]` — show last n items (default 20)
- `:interactive` — list interactive commands
- `:patterns` — show interactive detection patterns
- `:mode auto-tty [on|off]` — toggle auto-tty fallback
- `:suggest <token>` — fuzzy alternatives for a token
- `SM: <task>` — Smart Mode suggestions, e.g. `SM: find a large file`
- `SM: code c <query>` — C snippets, e.g. `SM: code c main`
- `CM:` — enter Chat Mode (Gemini). Type `exit` to leave
- `CM? <prompt>` — one-shot chat question

Tip: After leaving interactive programs like nano, the assistant prints a clean newline so the next prompt appears on a separate line.

##  Chat Mode (Gemini)

You can use either client (the app supports both):

- New client (preferred): `pip install google-genai` with `from google import genai`
- Classic client: `pip install google-generativeai` with `import google.generativeai as genai`

Provide an API key using any of these (highest priority first):

1) Hardcoded (dev only): copy `src/term_assistant/secrets_template.py` to `secrets_local.py` and set `GEMINI_API_KEY`.
2) Config file: set `gemini_api_key` or `gemini_api_key_file` in `~/.term_assistant/config.yml`.
3) Key file(s): put your key in `~/.term_assistant/gemini.key` or `.env` in `~/.term_assistant/secret.env`.
4) Environment variable: `GEMINI_API_KEY`.

Start chat inside the assistant:

```
CM:
what is java
```

One-shot without entering chat:

```
CM? what is java
```

### Setting GEMINI_API_KEY

- WSL/bash (current session):

```bash
export GEMINI_API_KEY="YOUR_KEY"
```

- PowerShell (current session):

```powershell
[Environment]::SetEnvironmentVariable("GEMINI_API_KEY","YOUR_KEY","Process")
```

##  Configuration

Config lives at `~/.term_assistant/config.yml`. Example:

```yaml
prompt: "{user}@{host}:{cwd}$ "
color: true
auto_tty: true
interactive_cmds: [nano, vim, less, more, htop, man]
interactive_patterns: [python*, ssh, tmux]
aliases:
  gs: git status
plugins:
  - term_assistant.plugins.example:ExamplePlugin
gemini_model: gemini-2.0-flash-exp
# gemini_api_key: "..."            # or provide through file/env as described above
# gemini_api_key_file: ~/.term_assistant/gemini.key
```

##  Plugins

Create a plugin by subclassing `PluginBase` and reference it by dotted path in the config.

```python
from term_assistant.plugins import PluginBase

class MyPlugin(PluginBase):
    name = "myplugin"
    def pre_command(self, ctx):
        # ctx: { command, expanded, cwd, env, start }
        pass
    def post_command(self, ctx):
        # ctx: { ..., exit_code }
        pass
```

##  Examples

```text
:doc ls               # view man/--help for ls
SM: find big files    # show suggested commands to solve the task
SM: code c main       # print C snippet for a main function
CM? explain pipes     # one-shot chat answer
CM:                   # enter chat; type 'exit' to leave
nano hello.c          # opens nano; .c files may be prefilled
```

##  Troubleshooting

- pip installs to user site instead of venv
  - Symptom: “Defaulting to user installation because normal site-packages is not writeable”
  - Fix: recreate venv so it has its own pip:
    ```bash
    rm -rf .venv
    python3 -m venv --upgrade-deps .venv
    source .venv/bin/activate
    python -m pip install -U pip
    ```

- `ModuleNotFoundError: No module named 'google'`
  - Install ONE Gemini client inside your venv (not system user site):
    ```bash
    source .venv/bin/activate
    python -m pip install google-genai
    # or: python -m pip install google-generativeai
    ```

- `export` fails in PowerShell
  - Use the PowerShell style:
    ```powershell
    [Environment]::SetEnvironmentVariable("GEMINI_API_KEY","YOUR_KEY","Process")
    ```

- Leaving nano/vim makes the prompt appear on the same line
  - The assistant now detects interactive commands and prints a clean newline after they exit.

##  License

No license specified. Add one if you plan to distribute.

---

Built with Python and prompt_toolkit. Entry points: `ta` (assistant), `tasnippet` (snippet CLI).
