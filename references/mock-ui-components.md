# Mock UI Components

When showcasing an app in a promo video, build **simplified mock components** that look like the real app but have no external dependencies (no dnd-kit, no Tailwind, no shadcn).

## Golden Rules

1. **Inline styles only** — no CSS classes, no Tailwind, no external stylesheets
2. **No interactivity** — no event handlers, no state management, no hooks (except Remotion's)
3. **Hardcoded data** — define mock data arrays directly in the component
4. **Match visual fidelity** — copy colors, border-radius, shadows, font sizes from source
5. **Animate with Remotion** — use `spring()` / `interpolate()` for entrance animations

## Pattern: Reading Source Components

Before building a mock, read the real component and extract:
- **Layout structure** — flex direction, grid, padding, gap
- **Colors** — background, text, border, status indicators
- **Typography** — font sizes, weights, font-family (monospace for codes)
- **Visual details** — border-radius, box-shadow, opacity, badges
- **State variations** — empty/filled, selected, error states

## Example: Mock Table

```tsx
const ROWS = [
  { code: "RV-2026-0412", route: "Fredericia → Trieste", status: "Open", loaded: 18, total: 28 },
  // ... more rows
];

export const MockTable: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <div style={{
      width: 1100,
      background: "#fff",
      borderRadius: 12,
      overflow: "hidden",
      boxShadow: "0 4px 24px rgba(0,0,0,0.15)",
    }}>
      {/* Header row */}
      <div style={{ display: "flex", background: "#f9fafb", padding: "12px 20px", borderBottom: "1px solid #e5e7eb" }}>
        {columns.map(col => (
          <div style={{ width: col.width, fontSize: 12, fontWeight: 600, color: "#6b7280", textTransform: "uppercase" }}>
            {col.label}
          </div>
        ))}
      </div>

      {/* Staggered row entrance */}
      {ROWS.map((row, idx) => {
        const s = spring({ frame: frame - idx * 8, fps, config: { damping: 14, stiffness: 280 } });
        const y = interpolate(s, [0, 1], [40, 0]);

        return (
          <div style={{
            display: "flex",
            padding: "14px 20px",
            borderBottom: "1px solid #f3f4f6",
            transform: `translateY(${y}px)`,
            opacity: s,
          }}>
            {/* cells */}
          </div>
        );
      })}
    </div>
  );
};
```

## Example: Mock Progress Bar

```tsx
export const MockProgressBar: React.FC<{ label: string; used: number; max: number; delay?: number }> = ({
  label, used, max, delay = 0
}) => {
  const frame = useCurrentFrame();
  const pct = Math.min(100, (used / max) * 100);
  const fillWidth = interpolate(frame - delay, [0, 50], [0, pct], clamp);
  const barColor = pct >= 80 ? "#c19b06" : "#159f65";

  return (
    <div style={{ borderRadius: 8, border: "1px solid rgba(0,0,0,0.1)", padding: "10px 14px" }}>
      <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
        <span style={{ fontSize: 12, color: "#6b7280" }}>{label}</span>
        <span style={{ fontSize: 12, fontWeight: 600 }}>{used} / {max}</span>
      </div>
      <div style={{ height: 6, borderRadius: 9999, background: "#e5e7eb", overflow: "hidden" }}>
        <div style={{ height: "100%", width: `${fillWidth}%`, borderRadius: 9999, background: barColor }} />
      </div>
    </div>
  );
};
```

## Example: Mock Wagon (Complex Geometric Component)

For components with detailed visual elements (wheels, rivets, couplers):

```tsx
// Wheel subcomponent — nested circles with shadow
<div style={{
  width: 22, height: 22, borderRadius: "50%",
  background: "rgba(10,10,10,0.8)",
  boxShadow: "0 2px 6px rgba(0,0,0,0.3), inset 0 1px 0 rgba(255,255,255,0.12)",
}}>
  <div style={{ position: "absolute", inset: 3, borderRadius: "50%", border: "1px solid rgba(255,255,255,0.2)" }}>
    <div style={{ position: "absolute", inset: 3, borderRadius: "50%", background: "#ECECF0" }} />
  </div>
</div>

// Rivets row
<div style={{ height: 9, background: "rgba(240,235,236,0.6)", display: "flex", gap: 11, padding: "0 6px", alignItems: "center" }}>
  {Array.from({ length: 7 }).map((_, i) => (
    <div key={i} style={{ width: 4, height: 4, borderRadius: "50%", background: "rgba(113,113,130,0.35)" }} />
  ))}
</div>
```

## Example: Mock Slot Card (State Variations)

```tsx
type SlotState =
  | { type: "empty" }
  | { type: "dropTarget" }
  | { type: "blocked"; reason: string }
  | { type: "filled"; iluCode: string; weight: string };

// Empty: dashed border, gray text
// Drop target: dashed green border, green text "✓ Drop here"
// Blocked: dashed red border, red text "✗ {reason}"
// Filled: solid border, grip dots + ILU code + weight
```

## Example: App Chrome (Nav Bar)

```tsx
<div style={{
  position: "absolute", top: 0, left: 0, right: 0,
  height: 56, background: "#fff",
  borderBottom: "1px solid #e5e7eb",
  display: "flex", alignItems: "center", padding: "0 32px",
  boxShadow: "0 1px 4px rgba(0,0,0,0.06)",
  transform: `translateY(${navY}px)`,  // Slide in from top
}}>
  <span style={{ fontSize: 18, fontWeight: 700 }}>AppName</span>
  {["Dashboard", "Voyages", "Settings"].map(tab => (
    <span style={{
      fontSize: 14,
      fontWeight: tab === activeTab ? 600 : 400,
      color: tab === activeTab ? "#1a63f5" : "#6b7280",
      borderBottom: tab === activeTab ? "2px solid #1a63f5" : "none",
      marginLeft: 24,
    }}>
      {tab}
    </span>
  ))}
</div>
```

## Status Badges

```tsx
const statusStyle = (status: string) => {
  switch (status) {
    case "Open": return { background: "#1a63f5", color: "#fff" };
    case "Departed": return { background: "#e5e7eb", color: "#6b7280" };
    case "Complete": return { background: "#fff", color: "#b45309", border: "1px solid #f59e0b" };
  }
};

<span style={{
  ...statusStyle(status),
  padding: "3px 10px",
  borderRadius: 9999,
  fontSize: 11,
  fontWeight: 600,
}}>
  {status}
</span>
```

## Highlight / Selection Glow

```tsx
const glowOpacity = isHighlighted
  ? interpolate(frame - highlightFrame, [0, 15], [0, 1], clamp)
  : 0;

<div style={{
  background: `rgba(26,99,245,${glowOpacity * 0.08})`,
  boxShadow: `inset 0 0 0 2px rgba(26,99,245,${glowOpacity * 0.5})`,
}} />
```

## Two-Panel Layout (Sidebar + Main)

```tsx
<div style={{ display: "flex", position: "absolute", top: 48, left: 0, right: 0, bottom: 0 }}>
  {/* Sidebar */}
  <div style={{ width: 320, borderRight: "1px solid rgba(0,0,0,0.1)", background: "#fff" }}>
    {/* sidebar content */}
  </div>

  {/* Main content */}
  <div style={{ flex: 1, display: "flex", gap: 16, padding: 24, justifyContent: "center" }}>
    {/* columns / cards */}
  </div>
</div>
```
