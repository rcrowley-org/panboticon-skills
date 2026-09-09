---
name: bot
description: Dispatch the Panboticon bot user to a task in the background. Use this skill when you have a detailed plan that the bot can implement non-interactively.
allowed-tools: Agent, Bash
user-invocable: true
---

# Bot

Dispatch the Panboticon bot user to a task in the background. Use this skill when you have a detailed plan that the bot can implement non-interactively.

Construct a command from this template:

    sudo -u "#10101" tmux new-session -d -s "SESSION_NAME" zsh -i -c "pi --model \"MODEL\" \"PROMPT\" 2>/tmp/pi.err"

Make the following substitutions:

* Provide a `PROMPT` containing the context and instructions to the bot.
* Choose a `MODEL` that's appropriate to the task. Run `pi --list-models` to list all available models.
* Derive a 2-4 word `SESSION_NAME` for this `PROMPT` that summarizes it, all lowercase with words separated by dashes.

Run the command.
