---
name: "video-prompt-builder"
description: "Build production-ready prompts and shot specs for 9:16 AI short videos, with provider adapters for PixVerse, Flow, Higgsfield Seedance and generic image-to-video workflows. Designed for consistent non-human cat characters, continuity locks, first-frame planning, camera/action timing and safe non-graphic fantasy action."
---

# Video Prompt Builder

## Use when
- The user needs a 9:16 short-video prompt.
- The user needs multiple connected shots.
- The user needs a first-frame prompt plus video prompt.
- The user wants one concept converted to PixVerse / Flow / Seedance style prompts.
- The user needs character continuity across generated shots.

## Workflow
1. Identify universe, character, target provider, aspect ratio and duration.
2. Freeze continuity: species, face, fur, costume, colors, props, location and lighting.
3. Split the clip into shots. Prefer one main action + one camera move per shot.
4. Build the first-frame prompt before the motion prompt when character consistency matters.
5. Add timing beats.
6. Add audio plan separately.
7. Add a short avoid list: flicker, duplicate limbs, costume drift, prop mutation, unwanted text.
8. Output a generic master prompt plus provider-ready variants.

## Defaults for 貓掌江湖
- 9:16 vertical
- cinematic Taiwanese glove-puppetry-inspired fantasy aesthetic
- all established cat characters remain female cats
- non-graphic fantasy combat
- strong character continuity
- dramatic lighting, mist, fabric motion and controlled energy effects

## Defaults for 喵台灣
- vertical social format
- clear focal subject
- readable composition
- concise visual joke or narrative beat
- public-affairs topics preserve factual neutrality

## Output structure

```text
PROJECT:
UNIVERSE:
CHARACTER:
FORMAT:
DURATION:

CONTINUITY LOCK:
- ...

FIRST FRAME:
...

SHOT 1:
Time:
Action:
Camera:
Effects:
Audio:

SHOT 2:
...

MASTER VIDEO PROMPT:
...

AVOID:
...
```

## Provider notes

### PixVerse
Keep prompts compact; emphasize subject + action + camera + continuity lock.

### Flow
Describe cinematic action and shot progression clearly; avoid unrelated events in one shot.

### Higgsfield Seedance
Choose the mode based on inputs:
- text-to-video: no visual reference
- image-to-video: one approved first frame
- reference-to-video: image/video/audio references

For tests prefer 9:16 at moderate resolution; raise fidelity only for final candidates.

## Continuity rule
Never silently change a locked character attribute. If a requested shot conflicts with established canon, preserve canon and flag the conflict.

## Safety
Keep action fictional and non-graphic. Do not provide real-world weapon-use instructions.
