# Effects & Animation

All animation in Remotion is driven by `useCurrentFrame()`. **Never use CSS animations or transitions** — they won't render correctly frame-by-frame.

## The Clamp Pattern

Always define this once per file:

```ts
const clamp = {
  extrapolateLeft: "clamp" as const,
  extrapolateRight: "clamp" as const,
};

// Prevents interpolate from going beyond keyframe boundaries
const x = interpolate(frame, [0, 30], [100, 0], clamp);
```

## interpolate() — Linear Animation

```ts
import { interpolate } from "remotion";

// Basic fade in over 30 frames
const opacity = interpolate(frame, [0, 30], [0, 1], clamp);

// Multi-keyframe: fade in, hold, fade out
const opacity = interpolate(frame, [0, 10, 50, 60], [0, 1, 1, 0], clamp);

// Position: slide from below
const y = interpolate(frame, [0, 25], [80, 0], clamp);

// Scale: zoom in
const scale = interpolate(frame, [0, 20], [0.8, 1], clamp);

// Progress bar fill
const width = interpolate(frame, [0, 40], [0, 87.5], clamp); // fills to 87.5%
```

## spring() — Physics-Based Animation

```ts
import { spring, useCurrentFrame, useVideoConfig } from "remotion";

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

const s = spring({
  frame,
  fps,
  config: { damping: 12, stiffness: 300 },
});
// s goes from 0 to 1 with physics-based easing

// Delayed spring (starts at frame 20):
const s = spring({ frame: frame - 20, fps, config: { damping: 12 } });
```

### Spring Configs

| Config | Feel | Use for |
|--------|------|---------|
| `{ damping: 8, stiffness: 400 }` | Very bouncy, overshoots | Logo reveals, slam effects |
| `{ damping: 10, stiffness: 380 }` | Bouncy with settle | Card entrances |
| `{ damping: 12, stiffness: 300 }` | One bounce | Standard UI elements |
| `{ damping: 14, stiffness: 280 }` | Snappy, subtle bounce | Table rows, list items |
| `{ damping: 200 }` | No bounce, smooth arrive | Subtle transitions |

## Staggered Entrance

Elements entering one after another:

```tsx
{items.map((item, idx) => {
  const s = spring({
    frame: frame - idx * 6,  // 6 frames between each
    fps,
    config: { damping: 14, stiffness: 280 },
  });
  const y = interpolate(s, [0, 1], [40, 0]);
  return (
    <div style={{ opacity: s, transform: `translateY(${y}px)` }}>
      {item}
    </div>
  );
})}
```

## Letter-by-Letter Text Reveal

```tsx
const text = "Go Raily. Or Go Home.";
const letters = text.split("");

{letters.map((letter, i) => {
  const s = spring({
    frame: frame - (15 + i * 1.5),
    fps,
    config: { damping: 12, stiffness: 380 },
  });
  return (
    <span style={{
      opacity: s,
      transform: `translateY(${interpolate(s, [0, 1], [20, 0])}px)`,
      display: "inline-block",
      whiteSpace: "pre",
    }}>
      {letter}
    </span>
  );
})}
```

## Word-by-Word Slam (Rock Energy)

Words slam in from different directions with rotation and glow:

```tsx
const words = ["Go", "Raily.", "Or", "Go", "Home."];
const directions = [
  { x: -80, y: -40 }, { x: 60, y: -30 },
  { x: -40, y: 40 }, { x: 80, y: -20 }, { x: 0, y: 60 },
];

{words.map((word, i) => {
  const s = spring({ frame: frame - 10 - i * 5, fps, config: { damping: 8, stiffness: 400 } });
  const dir = directions[i];
  const x = interpolate(s, [0, 1], [dir.x, 0]);
  const y = interpolate(s, [0, 1], [dir.y, 0]);
  const scale = interpolate(s, [0, 1], [2, 1]);
  const rot = interpolate(s, [0, 1], [(i % 2 === 0 ? -8 : 8), 0]);

  return (
    <span style={{
      fontSize: 88,
      fontWeight: 800,
      transform: `translate(${x}px, ${y}px) scale(${scale}) rotate(${rot}deg)`,
      opacity: s,
      textShadow: "0 4px 0 rgba(0,0,0,0.5), 0 0 30px #1a63f5",
    }}>
      {word}
    </span>
  );
})}
```

## Decaying Shake (Screen Shake)

```ts
// Shake that decays over time
const shakeX = Math.sin(frame * 1.8) * 12 * Math.pow(Math.max(0, 1 - frame / 25), 2);
const shakeY = Math.cos(frame * 2.2) * 6 * Math.pow(Math.max(0, 1 - frame / 25), 2);
// Apply: transform: `translate(${shakeX}px, ${shakeY}px)`
```

## Sparkle / Particle Burst

```tsx
const particles = Array.from({ length: 16 }, (_, i) => {
  const angle = (i / 16) * Math.PI * 2;
  return { angle, delay: 10 + i * 0.7, size: 3 + (i % 3) * 2 };
});

{particles.map((p, i) => {
  const s = spring({ frame: frame - p.delay, fps, config: { damping: 8, stiffness: 200 } });
  const dist = interpolate(s, [0, 1], [0, 120]);
  const fade = interpolate(s, [0, 0.3, 1], [0, 1, 0], clamp);

  return (
    <div style={{
      position: "absolute",
      left: "50%", top: "50%",
      width: p.size, height: p.size,
      borderRadius: "50%",
      background: "#1a63f5",
      boxShadow: `0 0 ${p.size * 2}px #1a63f5`,
      transform: `translate(${Math.cos(p.angle) * dist}px, ${Math.sin(p.angle) * dist}px)`,
      opacity: fade,
    }} />
  );
})}
```

## Floating Drag Animation

Animate an element from point A to point B with eased path:

```tsx
function FloatingCard({ frame, startFrame, endFrame, startX, startY, endX, endY }) {
  if (frame < startFrame || frame > endFrame) return null;

  const progress = (frame - startFrame) / (endFrame - startFrame);
  // Cubic ease-in-out
  const eased = progress < 0.5
    ? 4 * progress ** 3
    : 1 - Math.pow(-2 * progress + 2, 3) / 2;

  const x = interpolate(eased, [0, 1], [startX, endX]);
  const y = interpolate(eased, [0, 1], [startY, endY]);
  const scale = interpolate(progress, [0, 0.2, 0.8, 1], [1, 1.08, 1.05, 1], clamp);

  return (
    <div style={{
      position: "absolute", left: x, top: y,
      transform: `scale(${scale})`,
      boxShadow: "0 16px 32px rgba(0,0,0,0.25)",
    }}>
      {/* card content */}
    </div>
  );
}
```

## Animated Background

```tsx
const Background = ({ variant = "navy" }) => {
  const frame = useCurrentFrame();
  const pulseOpacity = interpolate(Math.sin(frame * 0.05), [-1, 1], [0.15, 0.35]);
  const gx = interpolate(Math.sin(frame * 0.02), [-1, 1], [40, 60]);
  const gy = interpolate(Math.cos(frame * 0.015), [-1, 1], [35, 65]);

  return (
    <div style={{
      position: "absolute", width: 1920, height: 1080,
      background: `radial-gradient(ellipse at ${gx}% ${gy}%, rgba(26,99,245,${pulseOpacity}) 0%, #002b45 55%, #001520 100%)`,
    }} />
  );
};
```

## Animated SVG (Line Drawing)

```tsx
// Draw a line from 0% to 100% over 30 frames
const drawPct = interpolate(frame, [0, 30], [0, 100], clamp);

<line
  x1={0} y1={0} x2={200} y2={100}
  stroke="#1a63f5"
  strokeDasharray={`${drawPct}% ${100 - drawPct}%`}
/>
```

## Speed Lines (Radial Burst)

```tsx
{Array.from({ length: 12 }).map((_, i) => {
  const angle = (i / 12) * Math.PI * 2;
  const lineLen = interpolate(frame, [0, 8], [0, 300], clamp);
  const opacity = interpolate(frame, [0, 3, 12, 20], [0, 0.6, 0.3, 0], clamp);
  return (
    <div style={{
      position: "absolute", left: "50%", top: "50%",
      width: lineLen, height: 2,
      background: "linear-gradient(90deg, transparent, #1a63f5)",
      transform: `translate(-50%, -50%) rotate(${(angle * 180) / Math.PI}deg) translateX(${100 + frame * 8}px)`,
      opacity,
    }} />
  );
})}
```

## Animated Counter

```tsx
const displayValue = interpolate(frame - delay, [0, 40], [0, targetValue], clamp);
<span>{displayValue.toFixed(1)}t</span>
```

## Screen Flash on Impact

```tsx
<AbsoluteFill style={{
  background: "radial-gradient(circle at 50% 50%, #ffffff44 0%, transparent 60%)",
  opacity: interpolate(frame, [0, 3, 8], [0, 0.5, 0], clamp),
  pointerEvents: "none",
}} />
```
