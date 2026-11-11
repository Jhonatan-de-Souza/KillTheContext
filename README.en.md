<h1 align="center">KillTheContext</h1>

<p align="center">
  <a href="README.md">Ler em Português</a> · <strong>English</strong>
</p>

**Before you start: download and install Orca IDE. Also install the Orca CLI if it does not come installed by default.** Open your project in Orca and confirm that the CLI is available in the terminal running Claude Code.

A prompt for creating a Claude Code hook that monitors context size and prepares to continue work in a fresh session in Orca.

## How to use

1. Have Orca IDE, the Orca CLI, Claude Code, and Python installed.
2. Open **the project where you want to apply the hook** in Orca.
3. Start Claude Code in that project and send the prompt below.
4. Review the generated files and ask Claude to demonstrate tests with low thresholds before using the defaults.

This repository shares the installation prompt. Claude generates the Python script and configuration inside your project; cloning this repository does not install a hook.

## What the hook should do

| Trigger | Requested behavior |
| --- | --- |
| Context above 150,000 tokens | Request a `STATUS.md` checkpoint once per session, then continue. |
| Context above 200,000 tokens | Block tools except reading, waiting, and editing `STATUS.md`. Preserve running processes. |
| `Stop` event after the limit | Open a new Orca tab only if `STATUS.md` was written after the limit was reached and no background processes are running. Do this once per session. |

The requested measurement is the sum of `input_tokens`, `cache_read_input_tokens`, and `cache_creation_input_tokens` from the latest assistant response in the transcript. The hook does not shrink the current session's history: it requests a checkpoint and prepares a fresh session to continue from it.

## Prompt to send to Claude

Copy the entire block and send it **inside your target project**:

```text
Create a Claude Code hook in this project that controls context size. I use Orca.
Register the same Python script in .claude/settings.json for the UserPromptSubmit, PreToolUse, PostToolUse, and Stop events.
The script reads the transcript_path provided to the hook and gets the context size from the latest assistant response
(input_tokens + cache_read_input_tokens + cache_creation_input_tokens).
- Above WARNING (default 150000): once per session, ask for a STATUS.md containing what is done, what is running, the exact next step, and the decisions I made. Then continue.
- Above LIMIT (default 200000): deny all tools except reading, waiting, and editing STATUS.md. Never stop or kill running processes.
- On the Stop event: if STATUS.md was written after LIMIT was reached and no background processes are running, open a new Orca tab with:
  orca terminal create --worktree path:<project folder> --title "continue" --command "claude --permission-mode auto \"continue from STATUS.md\"" --focus
  (do not use orca agent launch, because it opens Claude with --dangerously-skip-permissions). Do this only once per session.
- Make WARNING and LIMIT configurable through environment variables, with one variable to disable automatic opening and another to only log without opening anything. On any script error, exit without doing anything.
Show me how to test with low thresholds before using this for real.
```

## Test before using

Since Claude generates the implementation, it must define and document the variable names and log location. You can send this follow-up:

```text
Before enabling the hook, document the exact environment variable names and log location. Preserve existing hooks and settings.
Show commands for my operating system to test with WARNING=1000 and LIMIT=2000, using the actual implemented variable names and starting a fresh Claude session with them.
Start in log-only mode without opening tabs. Use a test transcript with known values to validate the thresholds without spending real tokens.
Test: below WARNING; above WARNING with one warning per session; above LIMIT with tool blocking and narrowly scoped exceptions; stale STATUS.md; STATUS.md written after the limit; an active background process; automatic opening disabled; repeated Stop events; and invalid input without side effects.
Do not run or terminate real processes to simulate these cases. Do not treat missing information as proof that no background processes are running; if this cannot be confirmed, do not open a new session.
Then demonstrate that opening happens only once in a test session with no background processes, an updated STATUS.md, and automatic opening enabled.
When finished, show how to restore WARNING=150000 and LIMIT=200000, leave log-only mode, and disable or remove the hook if needed.
```

The test numbers above describe desired values, not already implemented variable names. Tests and event support must be verified against the Claude Code and Orca versions used in your project. This repository does not include an executable implementation or previously validated integration tests.
