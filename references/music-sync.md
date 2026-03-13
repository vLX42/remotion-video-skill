# Music Sync

## Adding Audio

```tsx
import { Audio, staticFile } from "remotion";

// Place OUTSIDE TransitionSeries, inside root AbsoluteFill
<AbsoluteFill>
  <Audio src={staticFile("music.mp3")} />
  <TransitionSeries>{/* scenes */}</TransitionSeries>
</AbsoluteFill>
```

Audio plays from frame 0 for the full composition duration.

## Timestamp → Frame Conversion

```ts
const FPS = 30;
const timestampToFrame = (seconds: number) => Math.round(seconds * FPS);

// Example: chant at 18.5s = frame 555
// Example: beat at 13.0s = frame 390
```

## Beat-Synced Overlays

Use `<Sequence>` for text/effects that should appear at specific audio moments ON TOP of whatever scene is playing:

```tsx
// Define chant hits with absolute frame positions
const CHANT_HITS = [
  { frame: 390, variant: 0, duration: 30 },  // 13.0s — lead-in
  { frame: 555, variant: 1, duration: 30 },  // 18.5s — chant start
  { frame: 600, variant: 2, duration: 30 },  // 20.0s — repetition
  { frame: 645, variant: 3, duration: 30 },  // 21.5s — repetition
  { frame: 675, variant: 4, duration: 35 },  // 22.5s — final big hit
];

// Render ON TOP of TransitionSeries
<AbsoluteFill>
  <TransitionSeries>{/* scenes */}</TransitionSeries>

  {CHANT_HITS.map((hit, i) => (
    <Sequence key={i} from={hit.frame} durationInFrames={hit.duration}>
      <ChantHit variant={hit.variant} />
    </Sequence>
  ))}
</AbsoluteFill>
```

## Chant Overlay Component (Rock Energy)

A beat-synced text slam with escalating intensity per variant:

```tsx
const ChantHit: React.FC<{ variant: number }> = ({ variant }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Slam-in spring (very punchy, low damping = overshoot)
  const slamS = spring({ frame, fps, config: { damping: 8, stiffness: 400 } });

  // Decaying shake
  const shakeX = Math.sin(frame * 1.8) * 12 * Math.pow(Math.max(0, 1 - frame / 25), 2);

  // Scale: slam from big → settle at 1
  const scale = interpolate(slamS, [0, 1], [2.2, 1]);

  // Rotation kick (alternates per variant)
  const rot = interpolate(slamS, [0, 1], [variant % 2 === 0 ? -9 : 9, variant % 2 === 0 ? -3 : 3]);

  // Glow pulse
  const glow = interpolate(frame, [0, 5, 15, 30], [0, 1, 0.6, 0.3], clamp);

  // Fade lifecycle
  const opacity = interpolate(frame, [0, 3, 22, 30], [0, 1, 1, 0], clamp);

  return (
    <AbsoluteFill style={{ pointerEvents: "none" }}>
      {/* Screen flash */}
      <AbsoluteFill style={{
        background: "radial-gradient(circle at 50% 50%, #1a63f544 0%, transparent 60%)",
        opacity: interpolate(frame, [0, 3, 8], [0, 0.5, 0], clamp),
      }} />

      {/* Text */}
      <div style={{
        position: "absolute", top: "45%", left: "50%",
        transform: `translate(-50%, -50%) scale(${scale}) rotate(${rot}deg) translateX(${shakeX}px)`,
        opacity,
        fontSize: variant >= 3 ? 110 : 96,
        fontWeight: 800,
        textTransform: "uppercase",
        textShadow: `0 0 ${20 * glow}px #1a63f5, 0 4px 0 rgba(0,0,0,0.4)`,
        whiteSpace: "nowrap",
      }}>
        GO RAILY OR GO HOME
      </div>
    </AbsoluteFill>
  );
};
```

## Escalating Intensity Pattern

Each repeated chant hit should feel bigger than the last:

| Variant | Size | Extras | Energy |
|---------|------|--------|--------|
| 0 | 80px, shorter text ("GO RAILY!") | None | Lead-in |
| 1 | 96px, full text | Glow | Medium |
| 2 | 96px | Glow + subtitle accents | High |
| 3 | 96px | Glow + speed lines | Very high |
| 4 | 110px | Glow + speed lines + screen flash | Maximum |

Also vary:
- **Position** — don't always center, shift Y and X slightly per variant
- **Color** — alternate white/gold/white/red/white
- **Rotation** — alternate negative/positive angles

## Transition Timing to Beats

When a transition should land on a beat:
1. Identify the beat's absolute frame
2. Calculate the scene boundary frame from your TransitionSeries math
3. Adjust scene durations so the transition overlap falls on the beat frame

```
Scene A ends at frame X
Transition duration = T
Scene B starts at frame X - T (overlap)
→ The "midpoint" of the transition (most visual impact) = frame X - T/2
→ Align this midpoint to your beat
```

## Getting Audio Timestamps

If the user provides audio but no timestamps:
1. Ask them to provide beat/lyric timestamps
2. Or suggest tools: Audacity (free), Whisper (transcription), or manual listening
3. Key timestamps to capture: song structure changes, vocal hits, drops, and endings

## Duration Matching

Always match total composition frames to audio duration:

```ts
const AUDIO_SECONDS = 30.589;
const FPS = 30;
const TOTAL_FRAMES = Math.round(AUDIO_SECONDS * FPS); // 918

// In Root.tsx:
<Composition durationInFrames={918} fps={30} ... />
```

If audio is slightly longer than the composition, it gets cut. If shorter, there's silence at the end.
