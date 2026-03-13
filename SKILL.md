# Remotion Video Production

Create promotional videos from code using Remotion. Reads your app's components, builds animated scenes with transitions and effects, renders to MP4.

## Quick Reference

- [Project Setup](./references/project-setup.md) — Scaffold, package.json, configs, folder structure, static assets
- [Core Concepts](./references/core-concepts.md) — Composition, scenes, TransitionSeries, Audio, frame math, timing
- [Effects & Animation](./references/effects-animation.md) — spring(), interpolate(), stagger, shake, glow, particles, floating elements
- [Mock UI Components](./references/mock-ui-components.md) — Building faithful app UI mocks for demo videos (inline styles, no external deps)
- [Music Sync](./references/music-sync.md) — Syncing visual beats to audio timestamps, chant overlays, energy mapping
- [Rendering & Output](./references/rendering-output.md) — Studio preview, still frames, full render, codecs, troubleshooting

## Required Companion Skill

| Skill | Purpose |
|-------|---------|
| `remotion-transitions` | Custom TransitionPresentation implementations (Striped Slam, Zoom Punch, Diagonal Reveal, etc.) |

## When to Use

- User asks to "make a video", "create a promo", "build a demo reel"
- User mentions Remotion or programmatic video
- User wants to showcase app features in video form

## Workflow

```
1. Scaffold video/ project
2. Plan scenes + frame budget
3. Read source components → build inline-styled mocks
4. Build scenes with animation
5. Add transitions (remotion-transitions skill)
6. Wire TransitionSeries + Audio
7. Preview → music sync → render MP4
```

## When to Load References

- Starting a new video → [project-setup.md](./references/project-setup.md)
- Adding animations → [effects-animation.md](./references/effects-animation.md)
- Mocking app UI → [mock-ui-components.md](./references/mock-ui-components.md)
- Syncing to music → [music-sync.md](./references/music-sync.md)
- Rendering → [rendering-output.md](./references/rendering-output.md)
