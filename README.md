# Remotion Video Skill

An agent skill for creating promotional videos from your codebase using [Remotion](https://remotion.dev).

Works with Claude Code, GitHub Copilot, and other AI coding tools that support the [Agent Skills](https://agentskills.io) standard.

## What It Does

Point your AI assistant at your app's components, describe what you want to showcase, and get a rendered MP4. It reads your existing code, recreates the UI as animated video scenes, adds transitions and music sync, and renders the final output.

An easy and fun way to highlight a new feature or tool for your users — turn your actual UI into a polished promo clip without touching After Effects.

## Example

[`examples/raily-promo.mp4`](examples/raily-promo.mp4) — a 30s promo generated from a rail operations app. Custom transitions, drag-and-drop animations, music-synced chant overlays. Built entirely from code.

## Install

### Claude Code

```bash
claude skill install vLX42/remotion-video-skill
```

### GitHub Copilot (VS Code)

Copy the skill into your project's `.github/skills/` directory:

```bash
# Clone and copy into your project
git clone https://github.com/vLX42/remotion-video-skill.git /tmp/remotion-video-skill
mkdir -p .github/skills/remotion-video
cp /tmp/remotion-video-skill/SKILL.md .github/skills/remotion-video/
cp -r /tmp/remotion-video-skill/references .github/skills/remotion-video/
```

Copilot will automatically pick up skills from `.github/skills/` (and `.claude/skills/` if you already have them there).

### GitHub Copilot CLI

```bash
copilot plugin install vLX42/remotion-video-skill
```

Or manually place the skill folder in `~/.copilot/skills/remotion-video/`.

## Usage

Just ask:

- *"Create a 30s promo video showcasing our dashboard"*
- *"Make a demo reel of the booking flow set to this track"*
- *"Build a product video for the new feature"*

Your AI assistant will scaffold a Remotion project, build the scenes, and render to MP4.

## What's Covered

| Reference | Topic |
|-----------|-------|
| [project-setup.md](references/project-setup.md) | Scaffold, configs, fonts, assets |
| [core-concepts.md](references/core-concepts.md) | Composition, TransitionSeries, frame math |
| [effects-animation.md](references/effects-animation.md) | spring, interpolate, stagger, particles |
| [mock-ui-components.md](references/mock-ui-components.md) | Turning real components into video mocks |
| [music-sync.md](references/music-sync.md) | Beat-synced text overlays |
| [rendering-output.md](references/rendering-output.md) | Preview, render, codecs |

## License

MIT
