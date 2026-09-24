# screen-demo

A skill for cutting a long screen recording into a short demo.

The viewer sees the whole window, the typing sped up, the finished sentence once, a real keyboard bed under the typing, and a hold on the result. Dead waiting comes out. The payoff stays.

Built from a Grok session that ordered a Bitkey and paid the Lightning invoice. The playbook is the reusable part.

## Install

See [AGENTS.md](AGENTS.md). Short version:

```bash
git clone https://github.com/vincenzopalazzo/screen-demo-skill.git
cd screen-demo-skill
mkdir -p ~/.config/goose/skills
ln -s "$PWD/skills/screen-demo" ~/.config/goose/skills/screen-demo
```

Then tell the agent to use the screen-demo skill on your recording.

## Skill

[skills/screen-demo/SKILL.md](skills/screen-demo/SKILL.md) is what the next agent reads. It has the pacing, the one-subtitle rule, the keyboard bed, the source-seconds trap, and the failures from the first cut.

## License

MIT. See [LICENSE](LICENSE).
