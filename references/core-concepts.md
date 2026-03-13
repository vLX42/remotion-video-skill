# Core Concepts

## Composition Registration

Every video is a `<Composition>` registered in `Root.tsx`. The `id` is used in CLI commands.

```tsx
<Composition
  id="MyVideo"
  component={MyVideo}
  durationInFrames={918}   // MUST equal your total output frames
  fps={30}
  width={1920}
  height={1080}
/>
```

## TransitionSeries (Scene Sequencing)

The primary way to sequence scenes with transitions between them:

```tsx
import { TransitionSeries, linearTiming } from "@remotion/transitions";

export const MyVideo = () => (
  <AbsoluteFill>
    <Audio src={staticFile("music.mp3")} />

    <TransitionSeries>
      <TransitionSeries.Sequence durationInFrames={145}>
        <SceneOne />
      </TransitionSeries.Sequence>

      <TransitionSeries.Transition
        presentation={MY_TRANSITION}    // From remotion-transitions skill
        timing={linearTiming({ durationInFrames: 20 })}
      />

      <TransitionSeries.Sequence durationInFrames={140}>
        <SceneTwo />
      </TransitionSeries.Sequence>
    </TransitionSeries>
  </AbsoluteFill>
);
```

## Frame Budget Math

**This is the most important formula in Remotion video production.**

```
Total output frames = Sum(all scene durations) - Sum(all transition durations)
```

Transitions OVERLAP scenes — they don't add time, they subtract it.

Example:
```
Scenes:      145 + 140 + 185 + 265 + 140 + 134 = 1009 frames
Transitions:  20 +  18 +  20 +  15 +  18       =   91 frames
Output:      1009 - 91 = 918 frames = 30.6s @ 30fps  ✓
```

### Working backwards from audio duration

```ts
const AUDIO_DURATION_S = 30.589;
const FPS = 30;
const TOTAL_FRAMES = Math.round(AUDIO_DURATION_S * FPS); // 918

// Then distribute frames across scenes:
// 1. Decide transition count and durations first
// 2. Allocate remaining frames to scenes proportionally
// 3. Give the hero scene more frames
```

## Scene Architecture

Each scene is a self-contained React component. Inside a `TransitionSeries.Sequence`, `useCurrentFrame()` returns 0 at the scene's start.

```tsx
export const MyScene: React.FC = () => {
  const frame = useCurrentFrame();    // 0 at scene start
  const { fps } = useVideoConfig();

  // All animation driven by frame
  const opacity = interpolate(frame, [0, 30], [0, 1], clamp);

  return (
    <AbsoluteFill>
      {/* scene content */}
    </AbsoluteFill>
  );
};
```

## Audio

```tsx
import { Audio, staticFile } from "remotion";

// In your top-level composition (NOT inside a scene):
<Audio src={staticFile("music.mp3")} />

// Audio plays for the full composition duration automatically.
// Place it OUTSIDE TransitionSeries but inside the root AbsoluteFill.
```

## Overlays on Top of TransitionSeries

Use `<Sequence>` for elements that should appear at specific absolute frames ON TOP of scenes:

```tsx
<AbsoluteFill>
  <TransitionSeries>{/* scenes */}</TransitionSeries>

  {/* This overlay appears at frame 390 for 30 frames, on top of everything */}
  <Sequence from={390} durationInFrames={30}>
    <TextOverlay />
  </Sequence>
</AbsoluteFill>
```

This is perfect for beat-synced text flashes, watermarks, or chant overlays.

## Timing Types

```ts
import { linearTiming, springTiming } from "@remotion/transitions";

// Frame-perfect control (dramatic transitions)
linearTiming({ durationInFrames: 20 })

// Physics-based (springy, natural motion)
springTiming({ config: { damping: 200 }, durationInFrames: 18 })
```

## Scene-to-Scene Transitions

See the **remotion-transitions** skill for building custom `TransitionPresentation` implementations. The catalog includes:

- **Striped Slam** — horizontal bars slam in alternating directions
- **Zoom Punch** — old scene zooms out, new punches in
- **Diagonal Reveal** — dark panel sweeps with accent edge
- **Emerald Burst** — radial flash from center
- **Vertical Shutter** — venetian blind panels snap shut/open
- **Glitch Slam** — shake + RGB strips + hard pop

### Instantiate at module level (critical)

```ts
// ✅ Outside component — stable reference
const MY_TRANSITION = stripedSlam(8);

// ❌ Inside component — re-mounts every render
function Bad() { const t = stripedSlam(8); } // DON'T
```

## Common Patterns

### Staggered element entrance
```ts
items.map((item, idx) => {
  const s = spring({
    frame: frame - idx * 6,    // 6-frame gap between items
    fps,
    config: { damping: 14, stiffness: 280 },
  });
  const y = interpolate(s, [0, 1], [40, 0]);
  return <div style={{ opacity: s, transform: `translateY(${y}px)` }}>{item}</div>;
});
```

### App chrome (nav bar sliding down)
```ts
const navS = spring({ frame, fps, config: { damping: 14, stiffness: 200 } });
const navY = interpolate(navS, [0, 1], [-60, 0]);
// Apply: transform: `translateY(${navY}px)`
```
