# MiniMax Music — Advanced Prompt Methods

MiniMax Music supports two advanced approaches for the Style field. These methods produce highly detailed prompts that leverage the full 2,000-character Style limit.

---

## Method 1: Natural Language (6-Section Analysis)

Write a detailed natural-language description covering these 6 areas. This is the **recommended** approach — paste the full output directly into the Style field.

### Template Structure

#### 1. Song Information Analysis
- **Style**: Describe the subgenre in detail (e.g., "dark indie folk with ambient electronic textures")
- **Emotion Tags**: Core emotion + subtle shifts (e.g., "melancholic yet hopeful, building from quiet resignation to cathartic release")
- **Scenario Tags**: Listening environment (e.g., "late-night solo drive on empty highways, autumn rain")
- **BPM**: Specific tempo and rhythm description (e.g., "78 BPM with a gentle swaying 6/8 feel")

#### 2. Vocal Information Analysis
- **Gender & Timbre**: (e.g., "warm, slightly smoky male baritone with natural grit")
- **Vocal Style**: (e.g., "breathy verse delivery, opening to powerful mixed voice on chorus")
- **Backing Vocals**: Presence and character (e.g., "medium presence, ethereal 'ooh' harmonies on chorus, octave doubling on bridge")
- **Vocal Effects**: Reverb type, pitch processing (e.g., "medium plate reverb, no auto-tune, natural pitch")
- **Harmonic**: Vocal melody alignment (e.g., "minor pentatonic verse, major lift on chorus hook")

#### 3. Vocal Production & Sound Design
- **Spatial Sense**: Proximity in virtual space (e.g., "close and intimate in verse, pulling back slightly in chorus for room")
- **Dynamics Processing**: Volume/impact characteristics (e.g., "gentle compression, allowing natural dynamic range from whisper to full voice")
- **Stereo Field**: Left-right positioning (e.g., "centered lead, wide doubled harmonies on chorus")

#### 4. Instrumental Production & Sound Design
- **Spatial Sense**: Room modeling (e.g., "medium studio room feel, guitars in a slightly larger space")
- **Dynamics Processing**: Intensity control (e.g., "dynamic build from sparse verse to full chorus, controlled peaks")
- **Stereo Field**: Instrument panning (e.g., "acoustic guitar left, electric guitar right, bass and kick centered, synth pads wide")

#### 5. Performance by Structure
Break down each section:
- **[Verse]**: Instruments present, vocal approach, energy level
- **[Chorus]**: What changes, what's added, emotional peak
- **[Bridge]**: Contrast elements, shift in energy
- Include specific playing techniques and dynamics per section

#### 6. Synthesis
- **Core Imagery**: Overall metaphor (e.g., "a conversation with yourself at 3 AM, the room lit by a single lamp")
- **Narrative Analysis**: Why the musical elements serve the emotional intent

### Example: Natural Language Output

```
Dark indie folk with ambient electronic textures, melancholic yet hopeful, building from quiet resignation to cathartic release. Late-night atmosphere, rainy window, solo reflection. 78 BPM with gentle swaying feel.

Warm, slightly smoky male baritone with natural grit. Breathy intimate delivery in verses, opening to powerful mixed voice on chorus. Ethereal female backing vocals with soft "ooh" harmonies appearing on chorus, octave doubling on bridge. Medium plate reverb on vocals, completely natural pitch, no auto-tune.

Vocals close and intimate in verse, pulling slightly back in chorus to let the room breathe. Gentle compression allowing natural whisper-to-full-voice dynamics. Lead centered, backing harmonies panned wide on chorus.

Acoustic guitar fingerpicking left channel, ambient synth pad right channel, subtle bass centered, programmed brush-snare drums slightly left. Medium room feel, natural reverb on acoustic instruments, longer tails on synth pads. Dynamic build from sparse verse to layered chorus.

Verse: sparse fingerpicked acoustic guitar, whispered breathy vocal, distant pad drone, no drums. Pre-chorus: soft brush drums enter, bass note swells, vocal gains strength. Chorus: full arrangement, strumming guitar joins, drums open up, vocal at full power with harmonies, synth arpeggio adds movement. Bridge: stripped to voice and piano only, most vulnerable moment, single vocal line. Final chorus: everything returns with added intensity, vocal ad-libs.

Like watching headlights pass on the ceiling at 3 AM — each element arrives and departs like passing thoughts, building to a moment of clarity where grief becomes acceptance.
```

---

## Method 2: JSON Format

Structured JSON that can be pasted directly into the Style field. Useful for precise, repeatable parameter control.

### JSON Schema

```json
{
  "song_information_analysis": {
    "style": "String (e.g., Uplifting Pop, Dark Techno)",
    "emotion_tags": ["String", "String"],
    "scenario_tags": ["String", "String"]
  },
  "vocal_information_analysis": {
    "gender": "String (Male/Female/Instrumental/Androgynous)",
    "vocal_timbre": "String (e.g., Warm, Powerful, Raspy)",
    "vocal_style": "String (e.g., Belting, Whispering, Rapping)",
    "backing_vocals": {
      "presence": "String (High/Medium/Low/None)",
      "timbre": "String (e.g., Choir, Harmonies)"
    },
    "vocal_effects": {
      "reverb": "String (e.g., Large Hall, Dry)",
      "auto_tune": "String (e.g., Heavy, Natural)"
    },
    "harmonic": "String (e.g., Major, Minor)"
  },
  "arrangement_features_analysis": {
    "instrumentation": ["String", "String", "String"],
    "bpm": "Integer or Range"
  },
  "lyrics_content": {
    "language": "String (e.g., English, Chinese)",
    "structured_lyrics": "String (lyrics with [Verse], [Chorus] tags, use \\n for breaks)"
  }
}
```

### Example: JSON Output

```json
{
  "song_information_analysis": {
    "style": "Melancholic indie folk with ambient electronic textures",
    "emotion_tags": ["bittersweet longing", "quiet hope"],
    "scenario_tags": ["late-night drive", "rainy autumn evening"]
  },
  "vocal_information_analysis": {
    "gender": "Male",
    "vocal_timbre": "Warm, slightly smoky baritone with natural grit",
    "vocal_style": "Breathy verses, powerful mixed voice on chorus",
    "backing_vocals": {
      "presence": "Medium",
      "timbre": "Ethereal female harmonies"
    },
    "vocal_effects": {
      "reverb": "Medium plate reverb",
      "auto_tune": "Natural, no correction"
    },
    "harmonic": "Minor with major chorus lift"
  },
  "arrangement_features_analysis": {
    "instrumentation": ["fingerpicked acoustic guitar", "ambient synth pad", "brush drums", "sub-bass"],
    "bpm": 78
  },
  "lyrics_content": {
    "language": "English",
    "structured_lyrics": "[Verse]\nLines written in fading light\nWords that dissolve like morning frost\n\n[Pre-chorus]\nBut something shifts beneath the grey\n\n[Chorus]\nWe carry what we cannot name\nThrough highways lit by passing flame\nAnd somewhere past the rain, the rain\nThe sky remembers how to change"
  }
}
```

---

## Method 3: Reference Song Analysis

For replicating a specific artist's sound or style. Describe the reference in the Style field with detailed production analysis.

### Template

```
In the style of [reference description]: [genre/subgenre]. [Vocal timbre description] with [vocal style]. [Key instruments and their playing techniques]. [Production quality - spatial, dynamics, stereo]. [Emotional atmosphere]. [BPM and time signature]. [Specific sonic characteristics that define the reference].
```

### Example

```
In the style of a late 90s trip-hop ballad: dark, atmospheric downtempo. Smoky, low female vocals with breathy intimate delivery, almost whispered in verses, slightly stronger on chorus. Turntable scratches, deep Rhodes piano chords, programmed slow breakbeat drums with heavy reverb, sub-bass. Medium room reverb on vocals, dry drums, wet atmospheric pads. Melancholic, cinematic noir atmosphere. 75 BPM, 4/4 time. Vintage analog warmth, slightly muddy low-end, dark and moody throughout.
```

---

## Choosing a Method

| Situation | Recommended Method |
|-----------|-------------------|
| General song creation with detailed control | **Method 1: Natural Language** |
| API integration or batch generation | **Method 2: JSON** |
| Replicating a specific sound/artist style | **Method 3: Reference Analysis** |
| Quick, simple generation | Just write a few sentences in Style field |
| Maximum vocal precision | **Method 1** (most vocal detail) |
| Reproducible, structured workflow | **Method 2** (consistent format) |

---

## Tips for All Methods

1. **Use the full 2,000 characters** — More detail gives MiniMax more to work with
2. **Be specific about vocals** — MiniMax excels here; "raspy female alto" >> "female vocals"
3. **Describe the space** — Room size, reverb type, and proximity significantly affect output
4. **Include emotional narrative** — MiniMax responds to evocative language
5. **Specify instrument techniques** — "fingerpicked acoustic guitar" >> "guitar"
6. **Don't contradict yourself** — Consistent mood, energy, and production direction
7. **Match BPM to genre** — Always include tempo for predictable results
8. **Use scenario tags** — Physical context helps the model understand intent
