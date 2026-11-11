<h1 align="center">KillTheContext</h1>

<p align="center">
  <a href="README.md">Ler em Português</a> · <strong>English</strong>
</p>

**Before you start: download and install [Orca IDE](https://www.onorca.dev/download). Also install the Orca CLI if it does not come installed by default.** Open your project in Orca and confirm that the CLI is available in the terminal running Claude Code.

A prompt for creating a Claude Code hook that monitors context size and prepares to continue work in a fresh session in Orca.

## How to use

1. Have [Orca IDE](https://www.onorca.dev/download), the Orca CLI, Claude Code, and [Python](https://www.python.org/downloads/) installed.
2. Open **the project where you want to apply the hook** in Orca.
3. Start Claude Code in that project and send the prompt below.
4. Review the generated files and ask Claude to demonstrate tests with low thresholds before using the defaults.

This repository shares the installation prompt. Claude generates the Python script and configuration inside your project; cloning this repository does not install a hook.

## What the hook should do

| Trigger | Requested behavior |
| --- | --- |
| Context above 350,000 tokens (`WARNING`) | Request a `STATUS.md` checkpoint once per session, then continue. |
| Context above 400,000 tokens (`LIMIT`) | Block tools except reading, waiting, and editing `STATUS.md`. Preserve running processes. |
| `Stop` event after the limit | Open a new Orca tab only if `STATUS.md` was written after the limit was reached and no background processes are running. Do this once per session. |

The requested measurement is the sum of `input_tokens`, `cache_read_input_tokens`, and `cache_creation_input_tokens` from the latest assistant response in the transcript. The hook does not shrink the current session's history: it requests a checkpoint and prepares a fresh session to continue from it.

## Changing the limits

The defaults are **350,000 tokens** for `WARNING` (checkpoint request) and **400,000 tokens** for `LIMIT` (tools blocked). To change how the hook behaves, edit these two numbers in the prompt below, in the two lines that say `default 350000` and `default 400000`, before sending it to Claude:

| Value | Default | What it controls |
| --- | --- | --- |
| `WARNING` | `350000` | When Claude is asked to write `STATUS.md` and keep going. Lower it to checkpoint earlier. |
| `LIMIT` | `400000` | When tools are blocked and the hook prepares the fresh session. Keep it above `WARNING`. |

Both can also be overridden later with environment variables, without editing the script. Claude defines the exact variable names, so ask it to document them.

## Prompt to send to Claude

Copy the entire block and send it **inside your target project**:

```text
Create a Claude Code hook in this project that controls context size. I use Orca.
Register the same Python script in .claude/settings.json for the UserPromptSubmit, PreToolUse, PostToolUse, and Stop events.
The script reads the transcript_path provided to the hook and gets the context size from the latest assistant response
(input_tokens + cache_read_input_tokens + cache_creation_input_tokens).
- Above WARNING (default 350000): once per session, ask for a STATUS.md containing what is done, what is running, the exact next step, and the decisions I made. Then continue.
- Above LIMIT (default 400000): deny all tools except reading, waiting, and editing STATUS.md. Never stop or kill running processes.
- On the Stop event: if STATUS.md was written after LIMIT was reached and no background processes are running, open a new Orca tab with:
  orca terminal create --worktree path:<project folder> --title "continue" --command "claude --permission-mode auto \"continue from STATUS.md\"" --focus
  (do not use orca agent launch, because it opens Claude with --dangerously-skip-permissions). Do this only once per session.
- Make WARNING and LIMIT configurable through environment variables, with one variable to disable automatic opening and another to only log without opening anything. On any script error, exit without doing anything.
Show me how to test with low thresholds before using this for real.
```
