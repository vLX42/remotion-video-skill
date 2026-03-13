# Remotion Video Skill for Claude Code

A Claude Code skill that lets you create promotional videos from your codebase using [Remotion](https://remotion.dev) — React-based programmatic video.

Point it at your app's components, describe what you want to showcase, and get a rendered MP4.

## What It Does

- Reads your existing React components and recreates them as video-ready mocks
- Builds animated scenes with transitions, beat-synced text, and effects
- Renders to MP4 with audio

It's an easy and fun way to highlight a new feature or tool for your users — turn your actual UI into a polished promo clip.

## Example Output

See [`examples/raily-promo.mp4`](examples/raily-promo.mp4) — a 30s promo video generated from a rail operations app, with custom transitions, drag-and-drop animations, and music-synced text overlays. Built entirely by Claude Code using this skill.

## Install

```bash
claude skill install vLX42/remotion-video-skill
```

## Required Companion Skill

This skill works best with the **remotion-transitions** skill for custom scene-to-scene transitions:

```bash
claude skill install vLX42/remotion-transitions
```

## Usage

Ask Claude Code to make a video. Some examples:

```
"Create a 30s promo video showcasing our dashboard"
"Make a demo reel of the new wagon planner feature set to this music"
"Build a product video highlighting the booking flow"
```

Claude will:
1. Read your source components to understand the UI
2. Scaffold a `video/` project with Remotion
3. Build mock components, animated scenes, and transitions
4. Sync to your audio track if provided
5. Render to MP4

## What's Inside

| Reference | Covers |
|-----------|--------|
| [project-setup.md](references/project-setup.md) | Scaffold, package.json, configs, fonts, assets |
| [core-concepts.md](references/core-concepts.md) | Composition, TransitionSeries, frame math, Audio |
| [effects-animation.md](references/effects-animation.md) | spring, interpolate, stagger, shake, particles |
| [mock-ui-components.md](references/mock-ui-components.md) | Turning real components into video-ready mocks |
| [music-sync.md](references/music-sync.md) | Beat-synced overlays, chant components |
| [rendering-output.md](references/rendering-output.md) | Preview, render, codecs, troubleshooting |

## How It Works

The skill teaches Claude Code to:

1. **Scaffold** a standalone Remotion project (`video/`) next to your app
2. **Mock your UI** — reads your components and recreates them with inline styles (no Tailwind, no external deps)
3. **Animate** — uses `spring()` and `interpolate()` for entrance animations, drag simulations, progress bars
4. **Transition** — custom `TransitionPresentation` effects between scenes (striped slam, zoom punch, diagonal reveal, etc.)
5. **Sync to audio** — maps timestamps to frames for beat-matched text overlays
6. **Render** — outputs H.264 MP4 via Chrome Headless

## Quick Start (Manual)

If you want to use Remotion directly without the skill:

```bash
mkdir video && cd video
npm init -y
npm install remotion @remotion/cli @remotion/transitions @remotion/google-fonts react react-dom
npx remotion studio src/index.ts    # Preview
npx remotion render src/index.ts MyVideo out/video.mp4  # Render
```

## License

MIT
