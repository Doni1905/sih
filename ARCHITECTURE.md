# Linguisticus OS — System Architecture Diagram

## High-Level Data Flow (Complete 7-Layer)

```
╔══════════════════════════════════════════════════════════════════════╗
║                              USER                                    ║
║                    (Voice / Text / Context Events)                    ║
╚══════════════════════════════════╤═══════════════════════════════════╝
                                   │
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│ LAYER 1: INTERACTION LAYER (Input Capture)                           │
│                                                                      │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────────────┐ │
│  │ Wake Word      │  │ ASR / STT      │  │ Context Event Listener │ │
│  │ (OpenWakeWord) │─▶│ (Vosk)         │  │ (Notifications,        │ │
│  │ Always-on,     │  │ English+Tamil  │  │  Sensors, Time)        │ │
│  │ <200ms         │  │ +Tanglish mix  │  │                        │ │
│  └────────────────┘  │ <500ms         │  └───────────┬────────────┘ │
│                      └───────┬────────┘              │              │
│                              │                       │              │
│  ┌────────────────┐          │                       │              │
│  │ Text Input     │──────────┼───────────────────────┘              │
│  │ (Keyboard)     │          │                                      │
│  └────────────────┘          │                                      │
│                              ▼                                      │
│  Output: TranscribedInput {text, language_hint, confidence, source} │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│ LAYER 2: LINGUISTIC INTELLIGENCE (Inbound NLU)                       │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Personalized Linguistic Engine                                  │ │
│  │                                                                 │ │
│  │  ┌──────────────┐ ┌────────────────┐ ┌───────────────────────┐ │ │
│  │  │ Dialect      │ │ Code-Switch    │ │ Slang Interpreter     │ │ │
│  │  │ Detector     │ │ Analyzer       │ │ (DB lookup + hypothesis│ │ │
│  │  │ (area, tone, │ │ (Tamil%,       │ │  for unknown words)   │ │ │
│  │  │  formality)  │ │  English%,     │ │                       │ │ │
│  │  │              │ │  mix ratio)    │ │                       │ │ │
│  │  └──────────────┘ └────────────────┘ └───────────────────────┘ │ │
│  │                                                                 │ │
│  │  ┌──────────────┐ ┌────────────────────────────────────────┐   │ │
│  │  │ Emotion      │ │ User Linguistic Profile                │   │ │
│  │  │ Detector     │ │ (vocab, address words, history,        │   │ │
│  │  │ (tone, mood, │ │  code_switch_ratio, area preference)   │   │ │
│  │  │  urgency)    │ │                                        │   │ │
│  │  └──────────────┘ └────────────────────────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Canonical Intent Generator                                      │ │
│  │ Extracts: ACTION + TARGET + PARAMETERS + CONTEXT                │ │
│  │ Resolves ambiguity via user profile + conversation history      │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  Output: ParsedIntent {action, target, params[], tone,              │
│                         dialect_area, confidence}                     │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│ LAYER 3: COGNITIVE OPERATING LAYER (Decision & Planning)             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Intent Understanding & Goal Recognition                         │ │
│  │ (single-step vs multi-step task detection)                      │ │
│  └──────────────────────────────┬──────────────────────────────────┘ │
│                                 │                                    │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Context Awareness & User Memory                                 │ │
│  │ (conversation history, habits, time/location context)           │ │
│  └──────────────────────────────┬──────────────────────────────────┘ │
│                                 │                                    │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Decision Engine                                                 │ │
│  │                                                                 │ │
│  │        ParsedIntent                                             │ │
│  │             │                                                   │ │
│  │             ▼                                                   │ │
│  │  ┌──────────────────────┐                                       │ │
│  │  │ Rule-Based Parser    │──── solved? ───▶ YES ──▶ Plan         │ │
│  │  │ (TRY FIRST, <100ms) │                                       │ │
│  │  └──────────┬───────────┘                                       │ │
│  │             │ NO (complex/ambiguous/unknown)                     │ │
│  │             ▼                                                   │ │
│  │  ┌──────────────────────┐                                       │ │
│  │  │ On-Device LLM        │                                       │ │
│  │  │ (Phi-3 Mini, 4-bit)  │──────────────────────▶ Plan           │ │
│  │  │ via llama.cpp, <2s   │                                       │ │
│  │  └──────────────────────┘                                       │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                 │                                    │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Task Planner / Mission Planner                                  │ │
│  │ simple → direct execution | complex → multi-step plan           │ │
│  └──────────────────────────────┬──────────────────────────────────┘ │
│                                 │                                    │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Permission & Safety Validator                                   │ │
│  │ Destructive actions → confirm in user's dialect before exec     │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  Output: ExecutionPlan {steps[], requires_confirmation: bool}        │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│ LAYER 4: ORCHESTRATION LAYER (Workflow & Routing)                    │
│                                                                      │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────────────┐   │
│  │ Notification   │ │ Calendar       │ │ Communication Manager  │   │
│  │ Manager        │ │ Manager        │ │ ├─ Calls (dialer)      │   │
│  │                │ │                │ │ ├─ SMS (native)        │   │
│  └────────────────┘ └────────────────┘ │ ├─ Email (intent)     │   │
│                                        │ └─ WA/TG/IG            │   │
│  ┌────────────────┐ ┌────────────────┐ │   (Accessibility API) │   │
│  │ File Manager   │ │ Media          │ └────────────────────────┘   │
│  │                │ │ Controller     │                              │
│  └────────────────┘ └────────────────┘                              │
│                                                                      │
│  Output: ActionSequence {action_type, target_service, params}        │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│ LAYER 5: ANDROID EXECUTION LAYER (Action Engine)                     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Accessibility Service       │ Android APIs (Direct)             │ │
│  │ ├─ UI automation            │ ├─ TelephonyManager              │ │
│  │ ├─ 3rd party app control    │ ├─ AlarmManager                  │ │
│  │ ├─ Click / Scroll / Read    │ ├─ ContactsProvider              │ │
│  │ └─ Screen content parsing   │ └─ MediaStore                    │ │
│  ├─────────────────────────────┼──────────────────────────────────┤ │
│  │ System Services (Privileged)                                    │ │
│  │ ├─ WiFi, Bluetooth, NFC                                        │ │
│  │ ├─ Flashlight, Volume, Brightness                              │ │
│  │ └─ Power, DND, Airplane Mode                                   │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  Output: ActionResult {success: bool, data?, error?}                 │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│ LAYER 6: PERSONALIZED RESPONSE ENGINE (Outbound)                     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Response Formulation                                            │ │
│  │ (action result → human-readable text + personality)             │ │
│  └──────────────────────────────┬──────────────────────────────────┘ │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Dialect & Style Adaptor                                         │ │
│  │ (match user's dialect, mirror tone, code-switch ratio)          │ │
│  └──────────────────────────────┬──────────────────────────────────┘ │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Emotion & Pitch Harmonizer                                      │ │
│  │ (happy→upbeat, error→apologetic, casual→chill)                  │ │
│  └──────────────────────────────┬──────────────────────────────────┘ │
│                                 ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ TTS / Voice Synthesizer (Piper TTS)                             │ │
│  │ Tamil voice + English voice + Tanglish blend                    │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  Output: Voice + Text response in user's exact style                 │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               ▼
╔══════════════════════════════════════════════════════════════════════╗
║                              USER                                    ║
║              (Hears response in their own dialect/language)           ║
╚══════════════════════════════════════════════════════════════════════╝


═══════════════════════════════════════════════════════════════════════
 LAYER 7: SELF-REINFORCEMENT ENGINE (Background — Always Running)
═══════════════════════════════════════════════════════════════════════

    Runs parallel to all interactions. Feeds back into Layers 2, 3, 6.

    ┌──────────────────────────────────────────────────────────────┐
    │                                                              │
    │   OBSERVE ──▶ ANALYZE ──▶ HYPOTHESIZE ──▶ CONFIRM ──▶ APPLY │
    │      │                         │              │          │   │
    │      │                         ▼              ▼          │   │
    │      │                   confidence:     ask user       │   │
    │      │                   >0.8 auto-store  naturally      │   │
    │      │                   0.4-0.8 ask                     │   │
    │      │                   <0.4 wait                       │   │
    │      │                                                   │   │
    │      ▼                                                   ▼   │
    │  ┌─────────┐    ┌──────────┐    ┌──────────────────────────┐│
    │  │Log every│    │Frequency │    │REINFORCE                 ││
    │  │interaction   │counting, │    │├─ Positive/negative      ││
    │  │track     │    │patterns, │    │├─ 30-day decay           ││
    │  │unknowns  │    │co-occur  │    │└─ 90-day archive         ││
    │  └─────────┘    └──────────┘    └──────────────────────────┘│
    │                                                              │
    │  Storage: SQLite (SlangDB + UserProfile +                    │
    │           ConversationLog + Patterns)                         │
    │                                                              │
    │  Feeds back into:                                            │
    │  ├─ Layer 2 (Linguistic Engine) — new slang, updated profile │
    │  ├─ Layer 3 (Cognitive) — improved intent resolution         │
    │  └─ Layer 6 (Response) — adapted tone & vocabulary           │
    └──────────────────────────────────────────────────────────────┘
```

---

## Simplified Flow (Quick Reference)

```
USER
 │
 ├─── Voice ──▶ Wake Word (<200ms) ──▶ ASR/STT (<500ms)
 ├─── Text ───▶ Direct Input
 └─── Event ──▶ Context Listener
                    │
                    ▼
         Linguistic Intelligence
         (Dialect + Slang + Code-Switch + Emotion)
                    │
                    ▼
             Canonical Intent
         {action, target, params[]}
                    │
                    ▼
          Cognitive Engine
          ├─ Rule-Based (<100ms) ──▶ if solved ──▶ Plan
          └─ LLM Fallback (<2s) ──────────────▶ Plan
                    │
                    ▼
          Permission Check
          (destructive? → confirm)
                    │
                    ▼
          Orchestration → Route to Service
                    │
                    ▼
          Android Execution
          (APIs + Accessibility + System)
                    │
                    ▼
               Result
                    │
                    ▼
          Response Engine
          (formulate + adapt dialect + TTS)
                    │
                    ▼
                  USER

    [Self-Reinforcement runs in background, always learning]
```

---

## Interface Contracts

```
Layer 1 → Layer 2:  {raw_text, language_hint, confidence, audio_features}
Layer 2 → Layer 3:  {action, target, params[], context, tone, urgency, confidence}
Layer 3 → Layer 4:  {steps[], rollback_plan, permissions_needed, requires_confirmation}
Layer 4 → Layer 5:  {action_type, target_service, params, timeout}
Layer 5 → Layer 6:  {success, data?, error?, execution_time}
Layer 6 → User:     {spoken_text, display_text, voice_params}
```

## Error Flows

```
STT confidence < 0.4        → fallback to text input prompt
Intent confidence < 0.5     → ask clarification in user's dialect
Accessibility action fails  → retry with alternative path
Destructive action detected → confirm in user's dialect before executing
```

## Latency Targets

```
Wake word detection:    <200ms
Speech-to-text:         <500ms
Rule-based parsing:     <100ms
LLM fallback:          <2s
Action execution:       <5s per step
Response generation:    <300ms
Text-to-speech:         <500ms
─────────────────────────────────
Total (rule-based):     ~1.5s end-to-end
Total (LLM fallback):  ~3.5s end-to-end
```

---

## Language Spectrum (Unified Pipeline)

```
Pure English ←──── Indian English ←──── Tanglish ←──── Chennai Slang ←──── Pure Tamil

"Turn off WiFi"   "Switch off wifi na"  "wifi ah off pannu da"  "dai wifi ah off adi"  "வைஃபை அணை"
       │                   │                     │                    │                │
       └───────────────────┴─────────────────────┴────────────────────┴────────────────┘
                                              │
                                              ▼
                              Canonical Intent: {action: DEACTIVATE, target: WIFI}
                                              │
                                              ▼
                              Response mirrors input language/dialect
```
