# claude-auto-clicker

Auto-pilot for Claude Code. It starts Claude Code and answers the permission
prompts with `1. Yes`.

## Why

Claude Code has a bypass-permissions mode. That mode can still fall back to a
permission prompt and stop work until you answer it. `claude-auto-clicker`
watches for those prompts and answers them, so a long task does not wait for
you.

You keep full control of the session. The wrapper passes your keyboard through
to Claude Code, so you can type, interrupt and quit as usual.

## Requirements

- `expect`. On macOS, install it with `brew install expect`. On Debian or
  Ubuntu, use `sudo apt install expect`.
- `claude` on your `PATH`.

## Install

Copy the script to a directory on your `PATH` and make it executable. Make the
directory first, because `install` does not create it:

```sh
mkdir -p ~/.local/bin
install -m 755 claude-auto.sh ~/.local/bin/claude-auto-clicker
```

If `claude-auto-clicker` is not found after this, add the directory to your `PATH`.

## Use

Use `claude-auto-clicker` in place of `claude`. It passes on every argument:

```sh
claude-auto-clicker
claude-auto-clicker --resume
claude-auto-clicker "fix the failing tests"
claude-auto-clicker --dangerously-skip-permissions
```

## How it works

The script starts Claude Code on a pseudo-terminal and connects your keyboard
to it. At the same time it reads the output of Claude Code and looks for two
things in order:

1. A question that starts with `Do you want to` and ends with `?`, or the
   title `Network request outside of sandbox`.
2. The first option, `1. Yes`.

The sandbox dialog draws its question in a box. A narrow window wraps the
question across two lines, and then only the title identifies the dialog.

Claude Code does not always write the spaces between the words. In full-screen
mode it moves the cursor to the start of the next word instead. The wrapper
accepts a space or an escape sequence at each of these positions.

When it finds both, it waits 400 ms and then sends `1` and a carriage return.
This is the same input as a manual selection of `1. Yes`.

The wait is necessary. Claude Code refuses input that arrives less than 150 ms
after a dialog appears, so that a stray keypress cannot approve it. An
immediate `1` is discarded, and a dialog does not redraw while it waits for an
answer, so the wrapper gets no second chance.

Two checks keep the wrapper quiet:

- It answers only when the option list comes after the question. If Claude Code
  writes `Do you want to proceed?` in ordinary text, the wrapper does nothing.
- It answers a given prompt once. A redrawn screen cannot cause a second `1`.

The script also copies the size of your terminal to Claude Code, and does it
again when you change the size of the window.

## Trace

Set `CLAUDE_AUTO_TRACE` to a file to record every byte that Claude Code writes.
Use this to build a pattern for a dialog that the wrapper does not answer.

```sh
CLAUDE_AUTO_TRACE=/tmp/claude-auto-clicker.trace claude-auto-clicker
```

## Log

Set `CLAUDE_AUTO_LOG` to record each automatic answer:

```sh
CLAUDE_AUTO_LOG=~/claude-auto-clicker.log claude-auto-clicker
```

The wrapper does not write to the screen, because that damages the Claude Code
display.

## Tests

```sh
./test/run.sh
```

The tests replace `claude` with a stub program and drive the wrapper through a
pseudo-terminal. They need no network and no credentials.

## Limits

- The wrapper answers `Yes` to every permission prompt that it finds. It
  removes that safety check. Use it only for work that you accept in advance.
- In sandbox mode, this includes `Network request outside of sandbox`. The
  wrapper allows the connection to each host that Claude Code asks about.
- The wrapper finds a prompt by its text. A change to the text in a new version
  of Claude Code can stop it. If this occurs, the wrapper sends nothing and you
  answer the prompt yourself.
- Use the wrapper for interactive sessions. For a piped or scripted run, use
  `claude` directly.

## License

MIT. See [LICENSE](LICENSE).
