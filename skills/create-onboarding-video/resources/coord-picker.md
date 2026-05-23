# CoordPicker component reference

Bundled resource for the `create-onboarding-video` skill. **Always include this component in every new project.** Add it to every beat scene during development and remove it only before final render.

## What it does

Overlays a live coordinate picker on the Remotion Studio preview:
- **Hover** → green crosshair + live `x, y` badge follows the cursor (in 1920×1080 composition space)
- **Click** → pins a red dot + coordinate label; last 4 clicks kept on screen; all clicks logged to the browser console

This solves the core problem of guessing GlowRing / Pointer / TapDot positions. The user clicks the corners of any element and reads exact composition-space coordinates directly off the overlay.

## Important: zoom interactions

If a beat applies a `zoom` transform to the screenshot, the CoordPicker reports **post-zoom screen coordinates** (what the user sees). GlowRings placed at those coordinates belong in the overlay layer (not inside the zoomed image transform). If you need GlowRings to move *with* a zooming image, wrap them in a div with the same `transform: scale(zoom)` and `transformOrigin` as the image — then the user should measure coordinates at zoom=1 (frame 0).

**Simplest rule:** remove zoom from any beat where GlowRing positions need to be exact. Zoom and static overlays don't mix cleanly.

## Component source

```tsx
// src/components/CoordPicker.tsx
// Drop <CoordPicker /> inside any ScreenFrame children during development.
// REMOVE from all beats before final render.
import React, { useState } from "react";

export const CoordPicker: React.FC = () => {
  const [pos, setPos] = useState<{ x: number; y: number } | null>(null);
  const [pinned, setPinned] = useState<Array<{ x: number; y: number }>>([]);

  return (
    <div
      onMouseMove={(e) => {
        const rect = e.currentTarget.getBoundingClientRect();
        const scaleX = 1920 / rect.width;
        const scaleY = 1080 / rect.height;
        setPos({
          x: Math.round((e.clientX - rect.left) * scaleX),
          y: Math.round((e.clientY - rect.top) * scaleY),
        });
      }}
      onMouseLeave={() => setPos(null)}
      onClick={(e) => {
        const rect = e.currentTarget.getBoundingClientRect();
        const scaleX = 1920 / rect.width;
        const scaleY = 1080 / rect.height;
        const pt = {
          x: Math.round((e.clientX - rect.left) * scaleX),
          y: Math.round((e.clientY - rect.top) * scaleY),
        };
        setPinned((prev) => [...prev.slice(-3), pt]);
        console.log(`📍 clicked: x=${pt.x}, y=${pt.y}`);
      }}
      style={{ position: "absolute", inset: 0, cursor: "crosshair", zIndex: 9999 }}
    >
      {pos && (
        <div
          style={{
            position: "absolute",
            left: `${(pos.x / 1920) * 100}%`,
            top: `${(pos.y / 1080) * 100}%`,
            transform: "translate(12px, -28px)",
            background: "rgba(0,0,0,0.82)",
            color: "#4ECFA8",
            fontFamily: "monospace",
            fontSize: 22,
            fontWeight: 700,
            padding: "4px 10px",
            borderRadius: 6,
            pointerEvents: "none",
            whiteSpace: "nowrap",
            zIndex: 9999,
          }}
        >
          {pos.x}, {pos.y}
        </div>
      )}
      {pos && (
        <>
          <div style={{ position: "absolute", left: `${(pos.x / 1920) * 100}%`, top: 0, bottom: 0, width: 1, background: "rgba(78,207,168,0.5)", pointerEvents: "none" }} />
          <div style={{ position: "absolute", top: `${(pos.y / 1080) * 100}%`, left: 0, right: 0, height: 1, background: "rgba(78,207,168,0.5)", pointerEvents: "none" }} />
        </>
      )}
      {pinned.map((pt, i) => (
        <div key={i}>
          <div style={{ position: "absolute", left: `${(pt.x / 1920) * 100}%`, top: `${(pt.y / 1080) * 100}%`, width: 12, height: 12, borderRadius: "50%", background: "#FF4D4F", transform: "translate(-50%, -50%)", pointerEvents: "none", zIndex: 9999 }} />
          <div style={{ position: "absolute", left: `${(pt.x / 1920) * 100}%`, top: `${(pt.y / 1080) * 100}%`, transform: "translate(10px, -22px)", background: "rgba(0,0,0,0.75)", color: "#FF4D4F", fontFamily: "monospace", fontSize: 18, padding: "2px 8px", borderRadius: 5, pointerEvents: "none", whiteSpace: "nowrap" }}>
            {pt.x}, {pt.y}
          </div>
        </div>
      ))}
      <div style={{ position: "absolute", bottom: 16, left: "50%", transform: "translateX(-50%)", background: "rgba(0,0,0,0.75)", color: "#fff", fontFamily: "monospace", fontSize: 18, padding: "6px 18px", borderRadius: 8, pointerEvents: "none", whiteSpace: "nowrap" }}>
        hover to read coords · click to pin · last 4 clicks logged to console
      </div>
    </div>
  );
};
```

## Usage pattern

```tsx
// In any beat scene — add during development, remove before render
import { CoordPicker } from "../components/CoordPicker";

// Inside the ScreenFrame children:
<ScreenFrame src="screen-01.png">
  <GlowRing x={left} y={top} width={w} height={h} ... />
  <TopCaption>Your caption</TopCaption>
  <CoordPicker />   {/* ← add this */}
</ScreenFrame>
```

## Workflow

1. Add `<CoordPicker />` to all beats before sharing with user
2. User opens Remotion Studio, scrubs to each beat
3. For GlowRing elements: user clicks **top-left** then **bottom-right** corner → reads `TL=x,y  BR=x,y`
4. For tap targets (Pointer/TapDot): user clicks the **center** of the button/element → reads `x,y`
5. User sends coordinates back → update scene files with exact values
6. Remove all `<CoordPicker />` imports and usages before `npm run render`

## Removal checklist before render

Search for `CoordPicker` across `src/scenes/` — every import and usage must be gone:
```bash
grep -r "CoordPicker" src/
# should return nothing
```
