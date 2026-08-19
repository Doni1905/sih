USER (Voice / Text / Context Events)
         │
         ▼
┌──────────────────────┐
│ Wake Word + ASR/STT  │  ← Layer 1
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────┐
│ Personalized Linguistic      │  ← Layer 2
│ Engine (Dialect + Slang +    │
│ Code-Switch + Emotion)       │
└──────────┬───────────────────┘
           │
           ▼
      Canonical Intent
           │
           ▼
┌──────────────────────────────┐
│ Cognitive Engine             │  ← Layer 3
│ ├─ Rule-Based Parser (FIRST)│
│ ├─ LLM fallback (if needed) │
│ ├─ Memory + Context         │
│ ├─ Planner                  │
│ └─ Permission & Safety      │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ Orchestration (Routing)      │  ← Layer 4
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ Android Execution Engine     │  ← Layer 5
└──────────┬───────────────────┘
           │
           ▼
        Result
           │
           ▼
┌──────────────────────────────┐
│ Response Engine              │  ← Layer 6
│ (Dialect Adapter + TTS)      │
└──────────┬───────────────────┘
           │
           ▼
         USER

    ┌──────────────────────┐
    │ Self-Reinforcement   │  ← Layer 7 (background)
    │ (Observe → Learn →   │
    │  Hypothesize → Apply)│
    └──────────────────────┘
         ↕ (feeds back into Layers 2, 3, 6)
