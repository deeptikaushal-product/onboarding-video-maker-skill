---
name: create-onboarding-video
description: Produce short demo/onboarding videos in Remotion that walk through a product flow using full-screen desktop or mobile screenshots. Each screen gets a caption, optional GlowRing highlight, and optional cursor tap. Use when the user asks to create, build, or generate a demo video, onboarding clip, feature walkthrough, or any short video that showcases a flow using supplied screenshots.
---

# Create Onboarding Video

Produce a **short, polished demo video** in Remotion that walks through a product flow screen-by-screen. Each beat shows one screenshot with an optional highlight (GlowRing), optional cursor tap, and a caption. Beats are stitched together with smooth crossfades.

## Default spec

| Property | Default |
|----------|---------|
| Resolution | 1920×1080 (landscape) |
| Frame rate | 30fps |
| Duration per beat | 60–120 frames (2–4s) |
| Crossfade between beats | 15 frames (0.5s) |
| Node binary | `/opt/homebrew/bin/npm` (Homebrew install) |

---

## Phase 1: Intake — collect the screen manifest

Ask the user to provide a screen manifest. Each row is one beat. Accept it in any form (table, bullets, plain text) and normalise it internally.

**Required per screen:**

| Field | Notes |
|-------|-------|
| `filename` | e.g. `screen-01.png` — placed in `public/` |
| `caption` | Short text shown at the top of the frame |
| `glowRing` | `yes` or `no` |
| `glowRingCoords` | TL and BR corners in composition pixels — fill with `TBD` until CoordPicker session |
| `glowRingColor` | Optional. Default: `rgba(26, 115, 232, 0.9)` (blue) |
| `tap` | `yes` or `no` |
| `tapStart` | Cursor fade-in point — typically visual center of focal element. `TBD` until CoordPicker. |
| `tapTarget` | Where the tap fires — center of button/element. `TBD` until CoordPicker. |
| `durationSeconds` | Beat length in seconds. Default `3` |

**Example manifest:**

```
| Screen      | Caption                        | GlowRing | GlowRing TL→BR              | Tap | Tap start   | Tap target  | s |
|-------------|--------------------------------|----------|-----------------------------|-----|-------------|-------------|---|
| screen-01   | Start with a blank canvas      | yes      | TBD                         | yes | TBD         | TBD         | 2 |
| screen-02   | Build your Blog Writer Agent   | yes      | TBD                         | no  | —           | —           | 3 |
| screen-03   | Chain an Insertion Agent       | yes      | TBD                         | yes | TBD         | TBD         | 3 |
| screen-04   | Feed it your PRD               | no       | —                           | yes | TBD         | TBD         | 3 |
| screen-05   | Agent writes the blog          | no       | —                           | no  | —           | —           | 4 |
```

If the user gives you screenshots but no manifest, derive captions from filenames + context and leave coordinates as `TBD`. **Never guess pixel coordinates.**

---

## Phase 2: Build mode (CoordPicker enabled)

Build mode = CoordPicker active on every beat. This is the working state while coordinates are unknown.

### 2a. Scaffold the project

```
<project-name>/
├── public/
│   ├── screen-01.png
│   └── screen-NN.png
├── src/
│   ├── components/
│   │   ├── Cursor.tsx         ← copy verbatim from resources/cursor-component.md
│   │   ├── CoordPicker.tsx    ← copy verbatim from resources/coord-picker.md
│   │   ├── ScreenFrame.tsx    ← see pattern below
│   │   └── Caption.tsx        ← see pattern below
│   ├── scenes/
│   │   ├── BeatA.tsx          ← one file per screen
│   │   └── BeatN.tsx
│   ├── theme.ts
│   └── Root.tsx
└── package.json
```

### 2b. Required component patterns

#### theme.ts

```ts
export const COLORS = {
  bg: "#1A2151",           // navy — change to match brand
  blue: "#1A73E8",
  blueLight: "#7EB3F5",
  green: "#4ECFA8",
  white: "#FFFFFF",
};
export const FONT = "'Inter', 'Google Sans', -apple-system, sans-serif";
export const CAPTION_BAND = 140;
export const FPS = 30;
export const p = (seconds: number) => Math.round(seconds * FPS);
```

#### ScreenFrame.tsx

Full-bleed screenshot with a navy gradient at the top for caption readability.

```tsx
import React from "react";
import { AbsoluteFill, Img, staticFile } from "remotion";
import { CAPTION_BAND, COLORS } from "../theme";

export const ScreenFrame: React.FC<{
  src: string;
  children?: React.ReactNode;
  zoom?: number;
  zoomOriginX?: number;
  zoomOriginY?: number;
}> = ({ src, children, zoom = 1, zoomOriginX = 960, zoomOriginY = 540 }) => (
  <AbsoluteFill style={{ background: COLORS.bg }}>
    <Img
      src={staticFile(src)}
      style={{
        position: "absolute",
        width: 1920,
        height: 1080,
        objectFit: "cover",
        objectPosition: "center top",
        transform: `scale(${zoom})`,
        transformOrigin: `${zoomOriginX}px ${zoomOriginY}px`,
      }}
    />
    <div
      style={{
        position: "absolute",
        top: 0, left: 0, right: 0,
        height: CAPTION_BAND + 80,
        background: `linear-gradient(to bottom, ${COLORS.bg} 0%, ${COLORS.bg} 45%, rgba(26,33,81,0) 100%)`,
        pointerEvents: "none",
        zIndex: 10,
      }}
    />
    {children}
  </AbsoluteFill>
);
```

> **Zoom and GlowRings don't mix.** If a beat uses `zoom`, GlowRing overlays will drift because the zoom transform applies to the image only, not the children. Either remove `zoom` from beats that need GlowRings, or accept post-zoom coordinates for illustrative-only beats.

#### Caption.tsx

Pure opacity fade — no slide. The crossfade between beats handles the visual transition.

```tsx
import React from "react";
import { interpolate, useCurrentFrame } from "remotion";
import { COLORS, FONT, p } from "../theme";

export const TopCaption: React.FC<{
  children: React.ReactNode;
  staticEntry?: boolean;
}> = ({ children, staticEntry = false }) => {
  const frame = useCurrentFrame();
  const opacity = staticEntry
    ? 1
    : interpolate(frame, [0, p(0.43)], [0, 1], {
        extrapolateLeft: "clamp",
        extrapolateRight: "clamp",
      });
  return (
    <div
      style={{
        position: "absolute",
        top: 52, left: 0, right: 0,
        textAlign: "center",
        fontFamily: FONT,
        fontSize: 52,
        fontWeight: 700,
        lineHeight: 1.2,
        letterSpacing: -0.5,
        color: COLORS.white,
        opacity,
        maxWidth: 1600,
        margin: "0 auto",
        padding: "0 80px",
        pointerEvents: "none",
        zIndex: 20,
      }}
    >
      {children}
    </div>
  );
};
```

#### Root.tsx — crossfades with FadeWrap

Each beat overlaps the next by `FADE` frames. First beat has no fade-in (starts fully visible). Last beat has no fade-out (ends fully visible). This prevents the transparent/checkerboard canvas showing at the start and end.

```tsx
import React from "react";
import { AbsoluteFill, Composition, Sequence, interpolate, useCurrentFrame } from "remotion";
// ... beat imports
import { FPS } from "./theme";

const FADE = 15; // 0.5s crossfade — adjust freely

const BEATS = [
  { Component: BeatA,   frames: 60  },
  { Component: BeatB,   frames: 90  },
  // ...
  { Component: EndCard, frames: 120 },
] as const;

// Each beat starts FADE frames before the previous one ends
const { offsets, totalFrames } = (() => {
  let pos = 0;
  const offsets = BEATS.map((beat, i) => {
    const from = pos;
    pos += i < BEATS.length - 1 ? beat.frames - FADE : beat.frames;
    return from;
  });
  return { offsets, totalFrames: pos };
})();

const FadeWrap: React.FC<{
  duration: number;
  fadeIn: boolean;
  fadeOut: boolean;
  children: React.ReactNode;
}> = ({ duration, fadeIn, fadeOut, children }) => {
  const frame = useCurrentFrame();
  let opacity = 1;
  if (fadeIn && fadeOut) {
    opacity = interpolate(frame, [0, FADE, duration - FADE, duration], [0, 1, 1, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  } else if (fadeIn) {
    opacity = interpolate(frame, [0, FADE], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  } else if (fadeOut) {
    opacity = interpolate(frame, [duration - FADE, duration], [1, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  }
  return <AbsoluteFill style={{ opacity }}>{children}</AbsoluteFill>;
};

const BuilderStackVideo: React.FC = () => (
  <AbsoluteFill>
    {BEATS.map(({ Component, frames }, i) => (
      <Sequence key={i} from={offsets[i]} durationInFrames={frames}>
        <FadeWrap duration={frames} fadeIn={i > 0} fadeOut={i < BEATS.length - 1}>
          <Component />
        </FadeWrap>
      </Sequence>
    ))}
  </AbsoluteFill>
);

export const RemotionRoot: React.FC = () => (
  <Composition
    id="BuilderStack"
    component={BuilderStackVideo}
    durationInFrames={totalFrames}
    fps={FPS}
    width={1920}
    height={1080}
  />
);
```

#### Beat scene template (build mode — CoordPicker active)

```tsx
// BeatA.tsx — GlowRing beat (illustrative, no tap)
import React from "react";
import { AbsoluteFill, useCurrentFrame } from "remotion";
import { GlowRing } from "../components/Cursor";
import { TopCaption } from "../components/Caption";
import { ScreenFrame } from "../components/ScreenFrame";
import { CoordPicker } from "../components/CoordPicker"; // BUILD MODE
import { p } from "../theme";

export const BeatA: React.FC = () => {
  const frame = useCurrentFrame();

  // Replace TBD values after CoordPicker session
  const left = 0, top = 0, right = 0, bottom = 0; // TBD

  return (
    <AbsoluteFill>
      <ScreenFrame src="screen-01.png">
        <GlowRing
          x={left} y={top}
          width={right - left} height={bottom - top}
          startAt={p(0.2)} duration={p(2.4)}
          color="rgba(26, 115, 232, 0.9)" radius={14}
        />
        <TopCaption>Your caption here</TopCaption>
        <CoordPicker /> {/* REMOVE BEFORE RENDER */}
      </ScreenFrame>
    </AbsoluteFill>
  );
};
```

```tsx
// BeatD.tsx — Tap beat (interactive, with cursor)
import React from "react";
import { AbsoluteFill, Easing, interpolate, useCurrentFrame } from "remotion";
import { Pointer, TapDot } from "../components/Cursor";
import { TopCaption } from "../components/Caption";
import { ScreenFrame } from "../components/ScreenFrame";
import { CoordPicker } from "../components/CoordPicker"; // BUILD MODE
import { p } from "../theme";

export const BeatD: React.FC = () => {
  const frame = useCurrentFrame();

  // Replace TBD values after CoordPicker session
  const startX = 960, startY = 540;   // TBD — cursor fade-in point
  const targetX = 960, targetY = 540; // TBD — tap target

  const tapAt = p(1.8);

  const pointerOpacity = interpolate(
    frame,
    [p(0.3), p(0.7), tapAt + p(0.3), tapAt + p(0.6)],
    [0, 1, 1, 0],
    { extrapolateLeft: "clamp", extrapolateRight: "clamp" },
  );
  const moveProgress = interpolate(frame, [p(0.7), tapAt], [0, 1], {
    easing: Easing.bezier(0.16, 1, 0.3, 1),
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const pointerX = interpolate(moveProgress, [0, 1], [startX, targetX]);
  const pointerY = interpolate(moveProgress, [0, 1], [startY, targetY]);

  return (
    <AbsoluteFill>
      <ScreenFrame src="screen-04.png">
        <Pointer x={pointerX} y={pointerY} opacity={pointerOpacity} />
        <TapDot tapAt={tapAt} x={targetX} y={targetY} size={90} color="rgba(26,115,232,0.65)" />
        <TopCaption>Feed it your PRD</TopCaption>
        <CoordPicker /> {/* REMOVE BEFORE RENDER */}
      </ScreenFrame>
    </AbsoluteFill>
  );
};
```

### 2c. Start Studio and share with user

```bash
cd "<project-dir>"
/opt/homebrew/bin/npm start
# Studio opens at http://localhost:3000 (or next free port)
```

Tell the user:
> **CoordPicker is active on all beats.** Open the Studio, navigate to each beat, and:
> - For GlowRing: click **top-left** then **bottom-right** corner of the element → sends `TL=x,y BR=x,y`
> - For tap start: click the **visual center** of the focal area
> - For tap target: click the **center of the button/element** to tap
>
> Send me the coordinates for each screen and I'll update all the files.

---

## Phase 3: Lock coordinates

When the user returns coordinates, update every beat file with exact values. Then **remove CoordPicker from all beats**:

```bash
grep -rn "CoordPicker" src/scenes/
# Should return nothing after removal
```

Verify each scene file:
- GlowRing `x`, `y`, `width`, `height` replaced with exact values
- Tap `startX`/`startY` and `targetX`/`targetY` replaced
- `import { CoordPicker }` line removed
- `<CoordPicker />` JSX removed

---

## Phase 4: Render

```bash
cd "<project-dir>"
/opt/homebrew/bin/npm run render
# Output: out/<name>.mp4
```

`package.json` render script:
```json
"render": "remotion render BuilderStack out/builder-stack.mp4 --codec=h264"
```

---

## Security hardening

### The exposure

By default Remotion binds both the Studio HTTP server and the render-time `serve-static` server to `::` (all network interfaces). Anyone on the same WiFi can reach `http://<your-local-ip>:3000` and browse your project files — screenshots, source code, and anything in `public/`. There is no authentication on either server.

### Patch 1 — lock servers to loopback only

Apply to `node_modules/@remotion/renderer/dist/port-config.js`. Replace `getHostToBind` and `getHostsToTry`:

```js
// Before (binds to all interfaces):
const getHostToBind = (flattened, preferIpv4) => {
    if (preferIpv4 || !isIpV6Supported(flattened)) return '0.0.0.0';
    return '::';
};
const getHostsToTry = (flattened) => {
    return [
        hasIPv6LoopbackAddress(flattened) ? '::1' : null,
        hasIpv4LoopbackAddress(flattened) ? '127.0.0.1' : null,
        isIpV6Supported(flattened) ? '::' : null,
        '0.0.0.0',
    ].filter(truthy_1.truthy);
};

// After (loopback only):
const getHostToBind = (flattened, preferIpv4) => {
    return '127.0.0.1';
};
const getHostsToTry = (flattened) => {
    return ['127.0.0.1'];
};
```

This makes both the Studio and the render server listen only on `127.0.0.1` — unreachable from any other machine. It also simplifies port detection to a single host (consistent with the `get-port.js` patch).

### Patch 2 — port detection uses actual host (already required for render to work)

See the port detection fix section below — `get-port.js` must also be patched. Together, both patches ensure the tested host matches the bound host.

### Shutdown: kill the Studio when you're done

The Studio process keeps its port open until you kill it. Always shut it down when you're finished:

```bash
pkill -f "remotion studio"
# verify nothing is left listening
lsof -i :3000-3100 | grep LISTEN
```

Add this to your session-end checklist. Don't leave it running overnight or on a shared machine.

### Keep credentials out of the project

- `public/` should contain **only screenshots**. Never place `.env`, API keys, auth tokens, or config files there — Remotion's Studio server will serve anything in `public/` to anyone who can reach it.
- Add a `.gitignore` to the project root:

```gitignore
node_modules/
out/
.env
*.env.*
```

- The port patches live in `node_modules/` and are therefore excluded from git automatically. Re-apply them after any `npm install`.

### macOS firewall check

Confirm the macOS application firewall is on and that Node is not listed as an exception that allows incoming connections from outside localhost:

```
System Settings → Network → Firewall → Options
# Node.js or node should NOT have "Allow incoming connections" checked
```

With the loopback patch above this is belt-and-suspenders — the server won't accept non-loopback traffic at the socket level regardless of the firewall state.

---

## Port detection fix (critical on macOS with corporate networks / Jamf)

Remotion's default port detection uses TCP client sockets which **time out** on networks with firewall/Jamf policies instead of returning ECONNREFUSED. This causes "No available ports found" on both `npm start` and `npm run render`.

**Apply this patch once after `npm install`.** Do NOT run `npm install` again after patching — it will be overwritten.

### Patch: `node_modules/@remotion/renderer/dist/get-port.js`

Replace the `isPortAvailableOnHost` function and `testPortAvailableOnMultipleHosts` with:

```js
const isPortAvailableOnHost = ({ portToTry, host, }) => {
    return new Promise((resolve) => {
        const server = net_1.default.createServer();
        server.once('error', () => resolve('unavailable'));
        server.once('listening', () => {
            server.close(() => resolve('available'));
        });
        server.listen(portToTry, host);  // use actual host, not hardcoded 127.0.0.1
    });
};
const testPortAvailableOnMultipleHosts = async ({ hosts, port, }) => {
    // Sequential — avoids parallel probes that race on corporate networks
    for (const host of hosts) {
        const result = await isPortAvailableOnHost({ portToTry: port, host });
        if (result === 'unavailable') return 'unavailable';
    }
    return 'available';
};
```

**Why this works:**
- Original code opened a TCP client socket (outbound connection) — corporate firewalls block/timeout these
- Patched code opens a server socket (bind/listen) — purely local, unaffected by network policy
- `host` is passed through (not hardcoded to `127.0.0.1`) so the test matches what `serve-static.js` will actually bind to (often `::` on IPv6-enabled machines)
- Sequential testing avoids race conditions between parallel probes on the same port

---

## Remotion conventions

**Motion**
- Prefer `spring()` for physical motion (elements entering, snapping into place)
- Always set `extrapolateLeft: "clamp"` and `extrapolateRight: "clamp"` on `interpolate()`
- Use `Easing.bezier(0.16, 1, 0.3, 1)` as the default deceleration ease
- Reserve `Easing.bezier(0.34, 1.56, 0.64, 1)` for bouncy tap feedback

**Determinism**
- Never use `Date.now()`, `Math.random()`, `useEffect`, or `useState` in render paths
- All animation must derive purely from `useCurrentFrame()`

**GlowRing rules**
- GlowRings are illustrative — no cursor needed on the same beat
- Never place a GlowRing inside a zoomed beat's image transform — it will drift
- GlowRing coordinates are in composition space (1920×1080), not screen pixels

**Cursor rules**
- Every tap beat needs a `Pointer` that visibly moves to the target before the ripple fires
- Pointer fades in at the visual center of the focal area, moves in one straight line
- For multi-tap on the same UI: one pointer, continuous glide, never reset between taps
- For a new screen / different UI: reset the pointer with a fresh fade-in at center

**Common mistakes**
- Removing `Easing` import but leaving `Easing.bezier` in the code → ReferenceError
- Hardcoding `127.0.0.1` in port check while server binds to `::` → port mismatch errors
- Running `npm install` after applying the port patch → patch is overwritten, re-apply it
- Zoom on a GlowRing beat → ring drifts off element as zoom animates
- First/last beat fading from/to transparent → checkerboard in Studio → use `fadeIn={false}` / `fadeOut={false}` in FadeWrap

---

## Operating rules

- **Screenshots required.** Never invent UI from descriptions.
- **Never guess coordinates.** All GlowRing corners and tap targets come from the CoordPicker session.
- **CoordPicker in, render out.** Build mode = CoordPicker on every beat. Render mode = CoordPicker removed from every beat. Always verify with `grep -rn "CoordPicker" src/scenes/` before rendering.
- **One caption per beat, top-anchored.** Reuse `TopCaption` everywhere. Never position captions per-scene.
- **Same caption across consecutive beats = `staticEntry`.** Prevents caption flicker on cuts where the text doesn't change.
- **`/opt/homebrew/bin/npm` always.** Homebrew Node is not on the system PATH by default.
