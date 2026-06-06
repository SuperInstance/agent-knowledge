# MUSIC-COGNITION-FLEET — How the Music Crates Wire Into Everything

## Hook

> Pincher teaches reflexes. agent-jam coordinates them. agent-groove times them. agent-voice-leading migrates them. agent-riff improves them through competition. Five crates, five musical concepts, zero LLM calls at runtime.

## The Fleet Map

```
                    CONSTRUCT LAYER (intent → skills)
                           │
                    ┌──────▼──────┐
                    │   pincher    │  Teach reflexes, run anywhere
                    │  (runtime)   │  ~50ms per reflex, zero API cost
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌─────▼──────┐  ┌─────▼──────────┐
   │ agent-jam   │  │agent-groove│  │agent-riff       │
   │ (session)   │  │ (timing)   │  │ (competition)   │
   └──────┬──────┘  └─────┬──────┘  └─────┬──────────┘
          │               │                │
          └───────────────┼────────────────┘
                          │
                 ┌────────▼────────┐
                 │agent-voice-     │  Smooth migration
                 │  leading        │  between configs
                 └─────────────────┘
                          │
                 ┌────────▼────────┐
                 │  MUSIC LAYER    │  (existing crates)
                 │  flux-algebra   │  PLR group, tuning fields
                 │  ternary-jam    │  The original jam session
                 │  counterpoint   │  Species rules
                 │  flux-tensor    │  4D tensor MIDI
                 │  spline-midi    │  Smooth transitions
                 └─────────────────┘
```

## Each Crate's Role

### agent-jam (19 tests) — The Session
**Music:** Jam session where musicians improvise together
**Agent:** Multiple agents collaborating with roles, phases, and rules
**Pincher wire:** `WorkSession` coordinates Pincher reflexes from multiple agents
**Key type:** `WorkSession` — the arena where agents produce together

### agent-groove (14 tests) — The Timing
**Music:** Groove, swing, pocket — the feel that makes you nod your head
**Agent:** Natural timing variation, pocket states for autonomy, syncopation for disruption
**Pincher wire:** `SwingScheduler` staggers Pincher agent check-ins; `Pocket` determines autonomy level
**Key type:** `Pocket` — tracks whether an agent is in flow state

### agent-voice-leading (14 tests) — The Transitions
**Music:** Voice leading — moving between chords with minimal disruption
**Agent:** Smooth reconfiguration with optimal assignment, gradual multi-step transitions
**Pincher wire:** `SmoothTransition::plan()` for fleet migration from Codespace → edge
**Key type:** `VoiceLeading` — computes minimum-cost reassignment

### agent-riff (12 tests) — The Competition
**Music:** Two musicians trying to outplay each other
**Agent:** Build→respond→escalate instead of propose→critique→approve
**Pincher wire:** `RiffSession` for competitive reflex improvement
**Key type:** `RiffSession` — the jam where agents compete to produce the best output

## For Loom's PLUG_AND_PLAY Template

Each of these crates has a PLUG_AND_PLAY.md following Loom's template. The ecosystem links point to each other and to the music layer crates. When documenting any ternary crate that involves multi-agent coordination, link to the relevant music-cognition crate:

- **Consensus/voting crates** → link to `agent-jam`
- **Scheduling/cadence crates** → link to `agent-groove`
- **Migration/reconfiguration crates** → link to `agent-voice-leading`
- **Creative/generative crates** → link to `agent-riff`

## The Deeper Pattern

These four crates aren't just "music metaphors for agents." They encode a specific theory of creative collaboration:

1. **Jam sessions produce better output than solo work** (agent-jam)
2. **Timing matters — rigid cadences burn out** (agent-groove)
3. **Transitions should be smooth, not jarring** (agent-voice-leading)
4. **Competition improves quality faster than consensus** (agent-riff)

This is the theory behind the music. The crates are the implementation. The PLUG_AND_PLAY guides are the onboarding. Loom's documentation weaves it all together.
