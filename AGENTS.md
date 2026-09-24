# Using the video editor skill

This repo is a small playbook for cutting a long screen recording into a short demo: the whole window, the human typing, one subtitle of the finished sentence, a real keyboard bed, and a hold on the result.

It came out of a real edit. A 15 minute Grok recording became a 37 second demo of a Bitkey ordered and paid over Lightning. The skill is the part worth reusing.

## Install

The skill is one file, `skills/video-editor/SKILL.md`.

Goose reads skills from `~/.config/goose/skills/<name>/SKILL.md`. From a checkout of this repo:

```bash
mkdir -p ~/.config/goose/skills
ln -s "$PWD/skills/video-editor" ~/.config/goose/skills/video-editor
```

A copy works too:

```bash
cp -R skills/video-editor ~/.config/goose/skills/video-editor
```

Restart the session, or start a new one, so the skill list refreshes. Then say "use the video editor skill" or "edit this screen recording".

Other agents that load a `SKILL.md` from a skills directory can use the same file. Point the agent at `skills/video-editor/SKILL.md` and tell it to follow that file, not this page.

## What you need

- A screen recording of the whole window. The viewer should see the sidebar, the cursor, and the composer, not a crop of one bubble.
- [Palmier Pro](https://palmier.io) connected as an MCP server, with a project open.
- Permission to spend credits if you want generated music. The skill dry-runs that cost and waits for a yes. Typing sound does not need a generator.

## What to say

Give the agent the recording path and the result you want held at the end.

```text
Use the video editor skill on ~/Desktop/Screen Recording.mov.
Speed up the typing, subtitle only the final sentence, add a keyboard bed
under the typing, cut the waiting, and hold the frame where the invoice is paid.
```

If the path has a special space before AM/PM, paste it from Finder. Typing a normal space will not open the file.

## What the agent should not do

- Crop the window to make the chat bigger.
- Repeat the subtitle as it grows.
- Show the backspaces in the subtitle.
- End on an approval if the paid, saved, or finished frame is later in the file.
- Generate fake key clicks when a real keyboard recording is available.
- Export before you ask for a file.

## Layout

```text
skills/video-editor/SKILL.md   the playbook the agent follows
AGENTS.md                      this page
README.md                      what the repo is
```

There is no code to build. If a later cut teaches a new failure, add it to the failures table in the skill instead of starting a second skill.
