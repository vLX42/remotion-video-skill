---
name: remotion-video
description: Create promotional videos from code using Remotion. Reads your app's existing components, builds animated scenes with transitions and effects, syncs to music, and renders to MP4. Use when the user asks to make a video, create a promo, build a demo reel, or showcase app features in video form.
---

# Remotion Video Production

Create promotional videos from your codebase using Remotion — React-based programmatic video. Read existing components, build animated mock scenes, add transitions, sync to music, render to MP4.

An easy and fun way to highlight a new feature or tool for your users. Takes code from your project and generates a polished promo video.

## References

- [Project Setup](./references/project-setup.md) — Scaffold, package.json, configs, folder structure, static assets
- [Core Concepts](./references/core-concepts.md) — Composition, scenes, TransitionSeries, Audio, frame math
- [Effects & Animation](./references/effects-animation.md) — spring(), interpolate(), stagger, shake, glow, particles
- [Mock UI Components](./references/mock-ui-components.md) — Turning real app components into video-ready mocks
- [Music Sync](./references/music-sync.md) — Beat-synced overlays, chant components, energy mapping
- [Rendering & Output](./references/rendering-output.md) — Studio preview, still frames, render, codecs

## Workflow

1. Scaffold a `video/` project next to your app
2. Plan scenes and calculate frame budget (total frames = scene sum - transition sum)
3. Read source components and build inline-styled mocks (no Tailwind, no external deps)
4. Build scenes with `spring()` and `interpolate()` animations
5. Add custom transitions between scenes
6. Wire `<TransitionSeries>` + `<Audio>` in root composition
7. Preview in Remotion Studio, sync to music beats
8. Render to MP4

## When to Load References

- Starting a new video project → load [project-setup.md](./references/project-setup.md)
- Adding animations or effects → load [effects-animation.md](./references/effects-animation.md)
- Building app UI mocks for video → load [mock-ui-components.md](./references/mock-ui-components.md)
- Syncing visuals to music → load [music-sync.md](./references/music-sync.md)
- Previewing or rendering output → load [rendering-output.md](./references/rendering-output.md)

## Key Rules

- All `@remotion/*` packages must be the same version
- Use `staticFile()` for assets in `public/` — never relative imports
- Never use CSS animations — all motion via `interpolate()` / `spring()`
- `useCurrentFrame()` returns 0 at each scene's start inside TransitionSeries
- Instantiate transitions at module level, not inside components
- Build mocks with inline styles only — no external CSS/Tailwind dependencies
- Frame budget: `total output = sum(scene durations) - sum(transition durations)`
