# create-onboarding-video — Claude Code Skill

A Claude Code skill that produces short, polished **product demo videos** in [Remotion](https://www.remotion.dev/) by walking through a flow screen-by-screen. Each beat shows a full-screen desktop or mobile screenshot with an optional highlight ring, optional cursor tap, animated caption, and smooth crossfades between screens.

## What it produces

- **Input:** A manifest of screenshots + per-screen config (caption, highlight, tap)
- **Output:** An MP4 video rendered at 1920×1080, 30fps
- **Stack:** Remotion · React · TypeScript

## Features added in this fork

| Feature | What it does |
|---------|-------------|
| **Structured screen manifest** | Define each beat as a row: filename, caption, GlowRing (yes/no + coords), tap (yes/no + coords) |
| **CoordPicker** | In-browser overlay that shows live composition-space coordinates on hover and pins clicks — eliminates coordinate guessing |
| **Build mode / render mode** | CoordPicker on during development, checklist to remove before render |
| **FadeWrap crossfades** | Each beat fades into the next using overlapping `<Sequence>` — no extra packages |
| **Loopback-only port binding** | Servers bind to `127.0.0.1` only — not exposed to local network |
| **Corporate network port patch** | Fixes "No available ports found" errors on networks that block TCP client socket probes (e.g. Jamf MDM) |
| **Pure opacity caption fade** | Captions fade in without slide animation |

## Original skill

This is a fork of [**bidah/skill-set**](https://github.com/bidah/skill-set) by [@bidah](https://github.com/bidah). The original `create-onboarding-video` skill was designed for iOS portrait onboarding videos with cropped UI components. This version extends it for full-screen desktop demo videos with the additions listed above.

## Skill file

The skill lives in `SKILL.md`. It is loaded by Claude Code and instructs Claude on how to scaffold and build Remotion video projects end-to-end.

Supporting resources:
- `resources/coord-picker.md` — CoordPicker component source + usage guide
- `resources/cursor-component.md` — GlowRing, Pointer, TapDot component source

## Usage

1. In Claude Code, invoke the skill: `/create-onboarding-video`
2. Provide screenshots and a screen manifest (see `SKILL.md` for format)
3. Claude scaffolds the Remotion project, enables CoordPicker, starts Studio
4. You click element corners in the browser to get exact coordinates
5. Send coordinates back → Claude locks them in and removes CoordPicker
6. Run `npm run render` → `out/<name>.mp4`

## Node.js note

Assumes Node installed via Homebrew. All commands use `/opt/homebrew/bin/npm`.

## Security notes

Two patches are applied to `node_modules/@remotion/renderer/dist/`:
- `get-port.js` — uses `server.listen` (not client sockets) for port detection
- `port-config.js` — binds servers to `127.0.0.1` only

Re-apply after any `npm install`. See `SKILL.md` → Security hardening for details.

---

Built by [@mmt11289](https://github.com/mmt11289) · Forked from [bidah/skill-set](https://github.com/bidah/skill-set) · Repo: [onboarding-video-maker-skill](https://github.com/mmt11289/onboarding-video-maker-skill)
