# Wirebot voice transport adapter spec (27)

One relay interface, one adapter per transport. Adding the telephone later
means adding one adapter — never rewriting the relay.

## 1. Interface (every adapter implements all of it)

- `join(target)` / `leave()` — enter and exit the conversation surface.
- `speak(audioOrText)` — outbound voice; text is synthesized at the edge.
- `listen()` — inbound audio stream with speaker attribution where the surface allows.
- `transcript()` — append-only text of both sides into the shared relay session.
- `health()` — transport-local readiness; failure here never fails another adapter.

## 2. Adapters (generic names; providers live in qualified bindings)

Per the voice correction map, provider names belong in qualified adapter
bindings, not the generic domain. Adapter slots are generic; each slot's
binding names the provider surface it is qualified against.

| Slot | Surface | Status | Qualified binding |
|---|---|---|---|
| `voice-chat` | Discord voice channels | to build (slice 1) |Discord voice; relay routing (agentId, guild, channel, owner, dm) binds here |
| `meeting-a` | Meet via OpenClaw `google-meet` plugin | started (transcribe live) | extend plugin use, don't fork it |
| `meeting-b` | Zoom via OpenClaw `zoom-meetings` plugin | started (transcribe live) | extend plugin use, don't fork it |
| `phone-call` | 800 number via telephony provider | later (`planned`) | binding qualified when operator supplies account + number keys |

## 3. Config home (schema-valid)

The relay block lives under a dedicated `wirebot-voice-relay` entry with its
own schema — never smuggled into `openai` or `discord-context` entries, whose
schemas reject it. Per-adapter credentials stay in the owning plugin's config
or env; the relay holds only routing (ids, channels, agent).

```json
"wirebot-voice-relay": {
  "enabled": true,
  "agentId": "verious",
  "adapters": {
    "voice-chat": { "binding": "discord-voice", "guildId": "…", "channelId": "…", "ownerId": "…", "dmChannelId": "…" },
    "meeting-a": { "binding": "plugin:google-meet" },
    "meeting-b": { "binding": "plugin:zoom-meetings" },
    "phone-call": { "binding": "plugin:telephony", "status": "planned" }
  }
}
```

## 5. Conversation semantics (from doc 22 §C03 and capture rules)

- One committed turn, one all-speech admission per conversation; interruption
  is atomic and truthful heard-history is preserved.
- Speaking never aborts the underlying job; stop means stop speaking, not cancel.
- Transcript capture has one qualified owner per turn even where two surfaces
  observe it; replayed events and compaction never double-capture.

## 6. Registration (from doc 21 — extension by registration, no second registry)

- Adapters register into the existing relay; 27 defines no second registry,
  no duplicate controller, no parallel authority store.
- Optional adapters never become mandatory: a `planned` slot reports itself
  and the relay routes around it.

## 4. Portable/OO/DRY rules

- Shared behavior (session scope, transcript shape, stop-vs-abort, health
  reporting) lives once in the relay; adapters hold only transport code.
- No host paths, no account ids, no keys in the adapter code — all injected.
- Transcription-first: every adapter delivers text even where audio is degraded.
- A missing adapter (Twilio today) reports `planned`, never breaks the relay.
