Panboticon skills
=================

A collection of AI agent skills. Currently:

* `/audit` reports on what those sneaky bots have been up to.
* `/bot` sends a prompt from the human user to the bot user, which works on it in Pi in a tmux session.
* `/slack` sends a message to Slack using an incoming webhook URL found in the environment.

Install
-------

In Claude Code:

    /plugin marketplace add rcrowley/panboticon-skills
    # or
    /plugin marketplace add ./

    /plugin install panboticon-skills@panboticon-skills

In OpenCode:

    git clone https://github.com/rcrowley/panboticon-skills.git
    mkdir -p ~/.config/opencode/skills
    find panboticon-skills -name SKILL.md | xargs readlink -f | xargs dirname | xargs -I _ ln -s _ ~/.config/opencode/skills

In Pi:

    pi install https://github.com/rcrowley/panboticon-skills
    # or
    pi install ./

Usage
-----

    /audit # Claude Code

    /skills audit # OpenCode

    /skill:audit # Pi

    echo "* * * * * poll-github -d ~/.poll-github.sqlite3 -q @GITHUB_APP_NAME[bot] --agent pi" | crontab
