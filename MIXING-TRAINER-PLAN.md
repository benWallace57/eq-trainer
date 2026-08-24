# Mixing Trainer — Implementation Plan

## Concept

A web-based mixing trainer that teaches frequency separation and problem-solving using real multitrack stems. The user learns to make instruments sit together by applying EQ to individual stems.

## Stems Available (in `stems/`)

| Stem | File | Key frequencies |
|------|------|-----------------|
| Lead Vocals | lead-vocals.mp3 | Boxiness 300-500Hz, presence 2-5kHz, air 8-12kHz |
| Backing Vocals | backing-vocals.mp3 | Competes with lead in 1-4kHz |
| Piano | piano.mp3 | Wide range, mud 200-400Hz, competes with everything |
| Guitar Main | guitar-main.mp3 | Body 200-500Hz, presence 2-4kHz, masks vocals |
| Guitar Overdub | guitar-overdub.mp3 | Room/air, masks main guitar |
| Melodica | melodica.mp3 | Nasal 800-1.5kHz, fights vocals |
| Overhead | overhead.mp3 | Cymbal wash 3-8kHz, high-end clutter |
| Floor Tom | floor-tom.mp3 | Rumble 60-150Hz, mud 200-400Hz |
| Reference Mix | reference-mix.mp3 | The target — what a good mix sounds like |

## Game Modes

### Mode 1: "Fix the Problem" (guided challenges)

Predefined scenarios with a specific problem to solve:

1. **Muddy bass** — Piano + Floor Tom playing together, 200-400Hz buildup. User must cut mud from one/both.
2. **Boxy vocals** — Lead Vocals solo with 300-500Hz emphasis. User applies corrective EQ.
3. **Guitar masking vocals** — Lead Vocals + Guitar Main together. User carves space for vocals by cutting guitar in 2-4kHz.
4. **Competing guitars** — Guitar Main + Guitar Overdub. User separates them with complementary EQ.
5. **Vocal clarity** — Lead + Backing Vocals. User gives each their own space.
6. **Full mix mud** — Piano + Guitar + Floor Tom. User cleans up the low-mids across all three.

Each challenge:
- Plays the problem stems together
- Describes the issue ("These two instruments are fighting in the mids")
- User applies EQ to one or both stems
- A/B between processed and unprocessed
- Submit → scored on how well the problem was addressed

### Mode 2: "Spot & Fix" (diagnosis + cure)

- Two stems play together with an obvious masking problem
- User first clicks the spectrum to identify the problem frequency range
- Then applies a cut/boost to fix it
- Scored on: correct diagnosis + effective fix
- ELO-based progression (problems get subtler as you improve)

### Mode 3: "Match the Mix" (advanced)

- Multiple stems play together (raw, no EQ)
- Reference mix available to A/B against
- User has EQ on each stem, goal is to get close to the reference
- Scored by comparing the user's combined frequency response to the reference

## Technical Architecture

### File: `mix-trainer.html`

Single self-contained HTML file (same pattern as index.html and identify.html).

### Audio Routing

```
stem1 → stem1EQ (peaking/shelf) → stem1Gain → masterGain → destination
stem2 → stem2EQ (peaking/shelf) → stem2Gain → masterGain → destination
...
```

Each stem gets:
- Independent volume fader (gain node)
- Solo/mute buttons  
- A parametric EQ chain (2-3 bands: low shelf, peaking mid, high shelf)
- Visual EQ curve on a per-stem mini graph

### UI Layout

```
[nav: EQ Trainer | Frequency Identify | Mix Trainer]

[Challenge selector / description]
[A/B: Processed | Bypass | Reference]

[============= Frequency spectrum (combined) =============]

[Stem 1: Lead Vocals    ] [S] [M] [Vol ---o---] [EQ: Lo|Mid|Hi]
[Stem 2: Guitar Main    ] [S] [M] [Vol ---o---] [EQ: Lo|Mid|Hi]

[Submit] [Next Challenge] [Reset]

[Feedback / Score]
```

### EQ Controls Per Stem

Each stem gets 3 bands (clickable/draggable on a mini curve):
- **Low shelf**: freq fixed ~200Hz, gain adjustable ±12dB
- **Mid peak**: freq 200-8000Hz (draggable), gain ±12dB, Q 0.5-4
- **High shelf**: freq fixed ~4000Hz, gain adjustable ±12dB

This keeps it simple while covering all the real mixing moves.

### Scoring (Mode 1 & 2)

Each challenge defines:
- `problemBands`: frequency ranges that should be reduced (e.g., [{stem: 'guitar', freq: 2000, range: [1500, 3500], action: 'cut'}])
- Score based on: did the user cut in the right area, by a reasonable amount, on the right stem?
- Bonus for not over-cutting (preserving tone)

### Scoring (Mode 3)

- Compute combined frequency response of all user-EQ'd stems
- Compare to reference mix frequency response
- Score = average dB difference (same approach as the EQ trainer's curve scoring)

### Challenges Data Structure

```javascript
const challenges = [
    {
        id: 'muddy-piano-tom',
        title: 'Muddy Low-Mids',
        description: 'The piano and floor tom are clashing in the low-mids, making the mix sound muddy.',
        stems: ['piano', 'floor-tom'],
        hints: ['Try cutting around 200-400Hz on one or both'],
        targets: [
            { stem: 'piano', band: 'low', idealGain: -4, tolerance: 3 },
            { stem: 'floor-tom', band: 'mid', idealFreq: 300, idealGain: -3, tolerance: 3 }
        ]
    },
    // ... more challenges
];
```

### Masking Visualization

Show a real-time spectrum analysis overlay where the two stems' energy overlaps — highlight in red/orange where they're competing. This gives visual feedback as the user applies EQ.

Implementation: use AnalyserNode on each stem, compute FFT, find overlapping bins above a threshold.

## Implementation Order

1. **Basic page structure** — nav, challenge selector, stem loader
2. **Audio engine** — load stems, play in sync (loop), per-stem gain + EQ nodes
3. **EQ controls** — per-stem 3-band EQ with draggable curve display
4. **A/B toggle** — bypass all EQ vs hear processed
5. **Challenge system** — define 4-6 challenges, show description, score on submit
6. **Masking visualization** — optional but very helpful visual feedback
7. **Reference comparison** — Mode 3, compare to reference mix

## Nav Links

Add "Mix Trainer" link to nav in `index.html` and `identify.html`.
