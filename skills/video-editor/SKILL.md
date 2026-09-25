---
name: video-editor
description: >
  Video editor skill for cutting a long screen recording of a human talking
  to an agent into a short full-screen demo. Speed the typing, show the typed
  sentence once as a subtitle, put a real keyboard bed under the typing only,
  delete dead waiting, and hold the payoff (order sent, invoice paid, command
  finished). Use when the user wants a screen recording edited for X, a demo
  of an agent chat, typing captions, typing sounds, or a Palmier cut of a
  desktop recording. Trigger on: video editor, screen recording, typing
  subtitle, keyboard sound, accelerate the chat, remove the waiting, show the
  whole screen, demo for X.
---

# Video editor

Turn a long desktop recording into a short demo where a viewer can see
what the human did, read what they typed, and watch the result land.

The reference cut is a Grok chat that ordered a Bitkey and paid the
Lightning invoice. The rules below are the ones that survived that edit.
Do not invent a new format.

## What the viewer must see

1. The whole window. Sidebar, cursor, composer, and the click. Not a crop of the chat bubble.
2. The human typing, sped up, with one subtitle of the finished sentence.
3. The payoff held long enough to read. An approval is not the payoff. "Paid and settled" is.

If the cut ends before the result, it is not done.

## Do this first

1. `get_timeline` and `get_media`. Note fps, canvas, and `canGenerate`.
2. Find the real source path. macOS screen recordings often use U+202F before AM/PM. An ASCII space fails with "No such file or directory".
3. Probe the file. ReplayKit recordings are often video-only, ~30 fps, and a different size than the timeline.
4. Sample the source before cutting. Pull a JPEG every 20–40 seconds and read them. Mark three spans:
   - typing in the composer, including backspaces
   - the send, and the first useful reply
   - the payoff frame (invoice paid, file written, command succeeded)
5. Do not start an export until a captured timeline frame shows the payoff.

## Picture

Default canvas for X is 1920×1080, 30 fps, H.264.

Fit the whole window. Do not crop the sidebar to make the chat bigger. For a 1920×1228 source in a 1920×1080 timeline, the fit that worked was:

```
transform: { width: 0.879, height: 0.879, centerX: 0.5, centerY: 0.5 }
crop: { left: 0, top: 0, right: 0, bottom: 0 }
```

If the source is a different aspect, scale uniformly so both edges fit. Letterbox is better than cutting off the cursor.

Local sharpen is enough. A full upscale is not worth a large credit spend unless the user asks and approves the estimate.

## Pacing

| Span | Speed | Why |
|---|---|---|
| Composer typing | 2× to 2.5× | Backspaces and pauses should fly past. 1× makes a 30 second type feel like waiting. |
| Chat after send | 4× to 6× | Keep the reply readable for about a second per beat. |
| Payoff | 1×, hold 3–4 s | This is the frame the post exists for. |

Cut dead waiting. Do not cut the action that proves the story.

Place video with `source: [startSeconds, endSeconds]`. Do not use `endFrame` for a source span. Stills are the opposite: they need a duration, not a source end frame.

Speed is `edit_speed` uniform. Never set speed through `set_clip_properties`. If a trim is refused because a speed curve is set, reset speed, trim, then reapply speed.

### The source-seconds trap

On some imports, `source: [800, 836]` still places a clip whose trim starts near the end of the file. After every `add_clips`, read `trimStartFrame` back.

```
sourceSeconds ≈ trimStartFrame / sourceFps
```

If that is not the span you asked for, remove the clip and place again. Do not trust the request alone. Capture a frame before you call the beat done.

A last clip that reports a small `trimEndFrame` and a long timeline duration is holding the rest of the source. Remove it. Place a short known-good span instead of trimming it in place.

## One subtitle

The subtitle is the finished sentence, typed once. It is not a transcript of the composer.

- Do not show the delete phase.
- Do not repeat the sentence in growing cards ("I want to buy" then "I want to buy a Bitkey and…"). One card, one typewriter pass.
- Start it a few frames after the typing starts. End it when the message is sent. It must not sit on the reply.
- Two lines max. Inter, bold, about 34 px, `#1A1A1A` on a white pill, corner radius 22, padding 22×12, centered at `y: 0.88`.

```
add_texts({
  entries: [{
    content: "I want to buy a Bitkey and ship it to the Presidio,\npaying with Lightning, using the Lexe MCP.",
    startFrame: 8,
    endFrame: 470,
    style: {
      fontName: "Inter",
      fontSize: 34,
      bold: true,
      color: "#1A1A1A",
      alignment: "center",
      background: { enabled: true, color: "#FFFFFF", cornerRadius: 22, padding: { x: 22, y: 12 } }
    },
    transform: { x: 0.5, y: 0.88 },
    animation: "typewriter"
  }]
})
```

`add_texts` uses `content`, `x` / `y`, and `bold`. It rejects `text`, `centerY`, and `fontWeight`. Pass `animation: "typewriter"`, not an object.

### Track order

Index 0 renders on top. Text must be on a track above the screen recording.

- Omit `trackIndex` on `add_texts` so Palmier creates a new top track.
- Never move a text clip onto the video track. That splits or replaces the picture, and the next frame is a caption on black.
- If a move collapses tracks, stop. Re-read `get_timeline` before the next index-based call. Notes that say "track indices shifted" mean the indexes you just used are stale.

## Typing sound

Use a real keyboard recording. Generated clicks sound wrong.

- Search Freesound for "fast typing on mechanical keyboard". Download a preview you can actually hear. Check RMS before importing. A file whose peak is a few hundred is silence.
- Cut a steady 12–16 second stretch. High-pass around 180 Hz, loudnorm to about −16 LUFS, 44.1 kHz mono WAV.
- Import with `import_media` from a local path. Place it only across the typing span.
- Duck the music bed to about −12 to −14 dB while the keys play. Keys sit near 0 to +2 dB.
- Stop the keys when the message sends. Do not leave them under the sped-up chat.

If the user links a specific YouTube keyboard video, try it. YouTube often blocks the download. Say that, then use a similar real recording. Do not synthesize a substitute and call it the same thing.

Credit the recording in the delivery note. A Freesound preview is fine for a draft. For a public post, prefer CC0 and name the sound.

## Music

One instrumental bed under the whole cut, quieter than the keys. Generate only after a dry-run cost and a yes from the user. Do not upload or render on a dry run.

## Proof before export

Capture timeline frames, not guesses:

- mid-typing: one subtitle, composer visible, no second copy of the sentence
- just after send: subtitle gone
- invoice or command result visible
- last second: the paid / done line, not black

Then export H.264 1080p only when the user asked for a file. Finishing the edit is not permission to export. Poll `manage_exports` until `completed`. Seek the exported file with `-ss` before `-i` and check the last second yourself.

## Privacy

A full-screen demo shows everything in the window. Email threads, street addresses, balances, and order numbers will be readable. Warn before calling it postable. Do not claim a reviewer approved it unless one did.

## Failures from the reference cut

| What happened | What to do |
|---|---|
| ASCII path cannot open the `.mov` | Use the real path. U+202F before AM/PM. |
| `add_texts` on the video track | New top track. Never share the picture track. |
| Growing subtitle cards | One typewriter card of the final sentence. |
| Synthetic key ticks | Real keyboard WAV. Check it is not silent. |
| `source` seconds ignored | Read `trimStartFrame / fps`. Replace the clip. |
| Last clip runs to the end of the file | Remove it. Place a short payoff span and hold it. |
| Music bed keeps the timeline open after picture ends | Trim the bed to the last picture frame. |
| Black tail in the export | The timeline was longer than the picture. Trim audio, re-export. |
| Speed trim refused | `edit_speed` reset, then trim, then uniform again. |
| Punch-in clips the subtitle pill | Widen or shift the crop, re-capture a typing frame before mixing. |
| VO mix collapses to mono | `aformat=channel_layouts=stereo` on both inputs before `amix`. |

## Punch-in on typing

When the composer text is too small to read at full frame, punch in while the human types, then cut back to full frame on send.

- Full frame for the first ~2 s so the viewer orients (sidebar, window, cursor).
- Punch to ~1.4x on the chat panel plus composer for the typing span. A crop that worked on 1920x1080: `crop=1360:765:460:300,scale=1920:1080`. It keeps the subtitle pill, the composer, and the send button. Verify with a captured frame that the pill's rounded corners are fully inside.
- Hard cuts are fine. Punch-in is a standard demo cut; a smooth animated zoom is nice but never required.
- Cut back to full frame the moment the message sends. The reply and payoff always play full screen.
- In Palmier, do this with a nested punch or a second angle, not by destroying the full-screen clip. In ffmpeg: split full / punched / full, concat.

## Voiceover

A short voiceover carries the story for viewers watching without reading. Keep it under the video length minus the payoff hold.

- Script first, in this order: the ask, "watch it type", what the agent does, the numbers (order id, sats), the obstacle, the payoff. ~60-70 words for a ~37 s cut.
- Grok has no TTS API, so Grok writes the script, not the voice. Generate a draft voice locally (`say`, ElevenLabs, or the timeline's `generate_audio`) and mark it as a draft. The user can re-record or swap in their own voice.
- Start the VO ~1-1.5 s in. End it before the payoff hold so the last seconds breathe with music only.
- Duck the bed to ~0.3 under the VO. Keys stay only under typing. VO sits on top at full level, stereo, same sample rate as the timeline (44.1 kHz).
- ffmpeg mix: `volume='if(between(t,START,END),0.3,1)'` on the bed, `adelay` on the VO, `amix=inputs=2:duration=first`.

## Done

The cut is done when a stranger can watch it muted and still see the ask, the action, and the result, and with sound can hear keys only while the human is typing.
