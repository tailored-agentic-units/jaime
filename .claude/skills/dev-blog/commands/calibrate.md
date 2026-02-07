# Calibrate — Style Profile Generation

## Purpose

Generate or update the writing style profile at `.claude/context/style-profile.md` from analysis of the user's writing samples.

## Prerequisites

- Writing samples: the user provides a directory path or specific file paths
- Default sample location: `.artifacts/` (if it exists)

## Workflow

### Step 1: Collect Samples

Ask the user for the location of their writing samples. Accept:
- A directory path (all `.md`, `.txt`, and `.html` files within)
- Specific file paths
- Pasted content directly in the conversation

Aim for a minimum of 3-5 samples to produce a reliable profile. A mix of technical and non-technical writing produces the best calibration.

### Step 2: Analyze Voice Characteristics

Read each sample and identify patterns across the following dimensions:

**Tone** — Professional vs. casual, confident vs. hedging, warmth level, use of first person (I vs. we)

**Register** — Formality level, audience awareness, jargon handling

**Sentence Structure** — Opening patterns, sentence length variation, use of em dashes/semicolons/parentheticals, declarative vs. interrogative ratio

**Paragraph Construction** — Lead with conclusion vs. build to it, paragraph length, transition style

**Cadence** — Short-long-medium patterns, technical detail followed by restatement

**Vocabulary** — Technical-to-conversational ratio, characteristic phrases, favored words, avoided words

**Formatting** — Header usage, list styles, link conventions, code block usage, table usage

**Structural Patterns** — How updates are organized vs. concepts, section ordering, opening and closing conventions

**Audience Awareness** — Mixed vs. technical audience, impact framing vs. technical framing

### Step 3: Generate Profile

Write the style profile following this section schema:

```markdown
# Writing Style Profile — [Author Name]

Generated from analysis of [N] writing samples.

---

## Voice Characteristics

### Tone
- [Observations]

### Register
- [Observations]

---

## Sentence Structure

### Patterns
- [Observations]

### Paragraph Construction
- [Observations]

### Cadence
- [Observations]

---

## Vocabulary Fingerprint

### Technical-to-Conversational Ratio
- [Observations for updates vs. concepts]

### Characteristic Phrases
- [Quoted phrases the author uses repeatedly]

### Words to Favor
- [Words that appear naturally in the author's writing]

### Words to Avoid
- [Words the author never uses or actively avoids]

---

## Formatting Conventions

### Update Posts
- [Formatting patterns specific to updates]

### Concept Documents
- [Formatting patterns specific to concepts]

---

## Structural Patterns

### Update Posts
1. [Structural order for updates]

### Concept Documents
1. [Structural order for concepts]

---

## Audience Awareness
- [How the author adjusts for different audiences]
```

### Step 4: Write Profile

Save to `.claude/context/style-profile.md`. If the file already exists, ask the user whether to overwrite or merge with the existing profile.

Ensure the directory exists:

```bash
mkdir -p .claude/context
```

## Outcomes

- Style profile written to `.claude/context/style-profile.md`
- Profile calibrated from actual writing samples (not generic defaults)
- Ready for use by `draft` and `publish` commands
