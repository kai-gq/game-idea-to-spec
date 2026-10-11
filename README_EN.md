# Game Idea To Spec

Turn a one-line game idea into a ready-to-paste standard game spec for GameForge.

## What is this

Guides a one-sentence request like "make me a shooting / card / puzzle / vision-training mini game" through a few multiple-choice steps, then outputs the standard three-field spec (**Game Name / Gameplay / Art Style**) that can be pasted directly into GameForge's input box.

## Features

- Supports **regular casual mini games** (shooting / card / tower defense / platformer / puzzle, etc.) and **vision-training games for amblyopia / binocular vision** (fine discrimination / simultaneous perception / fusion / stereopsis)
- Multiple-choice interaction throughout — the user just clicks, zero barrier, no forms to fill
- Gameplay questions are **generated dynamically** based on the chosen option; outputs a detailed 400~700 word gameplay description (controls / win & lose conditions / enemy behavior / difficulty progression / scoring & feedback)
- Final output can be pasted straight into GameForge's input box (Game Name / Gameplay / Art Style)

## Directory structure

```
game-idea-to-spec/
├── SKILL.md                  # Main skill file: guidance flow, interaction rules and output format
└── references/
    └── scheme-library.md     # Scheme library: type pool / gameplay skeletons / style pool / vision-training pool
```

## Usage flow

1. The user states a game idea (e.g. "make me a shooting mini game");
2. Complete 4 multiple-choice steps:
   - **Training type**: normal / fine discrimination / simultaneous perception / fusion (stereopsis can be typed in);
   - **Game scheme or color scheme**: normal → pick 1 of 5 gameplay options of the same type; training → red-blue or red-cyan 2-choose-1;
   - **Gameplay details**: 3~4 questions generated dynamically from the chosen option (goal / difficulty / boss / controls, etc.);
   - **Art style**: 4-choose-1 (default flat cartoon for normal games, training-simple for vision training).
3. Output the three-field spec, paste it into GameForge's input box, and hit "Generate".

## Output format

Three independent fields, copied one box at a time:

| Field | Description |
|---|---|
| Game Name | 2~8 characters, matching the gameplay and audience |
| Gameplay | 400~700 word detailed description in one paragraph |
| Art Style | 1~2 style phrases |

## Requirements

- Doubao desktop (relies on the `interaction.ask` multiple-choice component)
- GameForge (the game generation engine that receives the three-field spec)
