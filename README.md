# wirebot-voice-relay

One relay interface, one adapter per transport. Adding the telephone later
means adding one adapter — never rewriting the relay.

- **Version:** 0.1.0 (prototype)
- **Spec:** `SPEC.md` (reconciled against wirebot-core docs 21/22/25 + voice
  correction map; §§5–6 name their sources)
- **Status:** scaffold — schema and contract declared; `voice-chat` adapter
  build is tracked in wirebot-core beads (`voice: build discord-voice adapter`
  slice); implementation lands here, in `src/`.

## Layout

- `SPEC.md` — the interface, slots, config home, semantics, registration rule.
- `openclaw.plugin.json` — the relay's own schema. Relay routing lives here;
  per-adapter credentials stay in the owning plugin's config or env.
- `src/` — adapter implementations (one folder per slot).
- `config.example.json` — routing shape with placeholder ids (no live ids, no keys).

### Wirebot prototype crosswalk

[Wirebot prototype runtime BOM](https://github.com/Startempire-Wire/agent-driven-life-business-os/blob/a21e1e9baa9819faa58317cc198f7d8694c8be53/docs/agent-os-golden-path/wirebot-prototype-runtime-bom.md#matrix-by-first-eligible-expression) — Optional voice transport and interface. Presence does not prove configured/use/entitlement. WB-H03.
