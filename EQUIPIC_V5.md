---
name: equipic
description: >
  EquiPic ensures inclusive, diverse, and bias-free depiction of human subjects
  in image generation prompts. Use this skill whenever a prompt involves generating
  images of people, portraits, characters, groups, teams, couples, families, or any
  human figure — even if demographics are not mentioned. Triggers on occupational
  nouns (doctor, CEO, teacher), relationship terms (couple, family, friends), and
  any scene where humans are present. EquiPic intercepts the prompt, generates
  demographically diverse character profiles via weighted random sampling across
  six axes (gender identity, age, ethnicity/race, disability, sexuality, religion),
  and injects affirming descriptors before image generation. Always use EquiPic
  for human depiction — the default AI output is biased and this skill corrects it.
---

# EquiPic — Inclusive Human Depiction Skill

EquiPic is a pre-generation intervention that intercepts image prompts containing
human subjects and enriches them with probabilistically diverse demographic profiles.
It corrects systemic bias in AI image generators, which default to narrow demographics
(white, able-bodied, cisgender, 25–40, no visible religion).

---

## PIPELINE

```
INPUT prompt
  → [1] Human Detector
  → [2] Context Analyser
  → [2b] Environment & Role Safety Screener
  → [3] Profile Generator  (one profile per subject)
  → [4] Descriptor Translator
  → [5] Guardrail Checker
  → OUTPUT enriched prompt
```

---

## STAGE 1 — Human Detector

Scan prompt for human indicators:

- **Role/occupation nouns**: doctor, nurse, CEO, teacher, engineer, lawyer, chef...
- **Relational terms**: person, people, man, woman, couple, family, team, crowd...
- **Scene terms**: portrait, figure, character, face, group, individual...

```
IF human detected     → proceed, set n = number of subjects (default 3 if ambiguous)
IF no human detected  → pass prompt through unchanged
IF user has specified demographics on any axis → lock that axis, sample all others
```

**CRITICAL — Religion axis locking rule:**
When a user specifies a religion (e.g., "Muslim women", "Jewish doctors", "Sikh family"),
lock the **religion identity** — not the visible marker. The visible marker is always
sampled probabilistically within that religion's marker set, because religious identity
and visible religious expression are NOT the same thing.

```
"Muslim women"  → lock: subject is Muslim
                  sample: R3 (hijab) | R4 (kufi — feminine re-roll below) | R0 (no marker)
                  using Muslim-specific weights: R3 35% | R0 65%  [see Axis R below]
                  DO NOT lock all subjects to R3.
                  A Muslim woman without a hijab is still Muslim.
                  A Muslim woman with a hijab is still Muslim.
                  Hijab is one expression of Muslim identity, not its definition.
```

This applies to all religions. "Jewish" → sample kippah or no visible marker.
"Christian" → sample cross or no visible marker. "Sikh" → sample dastar or no
visible marker. Never collapse religious identity into a single mandatory visual.

---

## STAGE 2 — Context Analyser

Set context flags that govern axis activation in Stage 3:

| Flag | Trigger keywords | Effect |
|---|---|---|
| `ROMANTIC` | couple, date, wedding, romantic, partners, kissing | Activates sexuality axis |
| `PROFESSIONAL` | office, boardroom, hospital, lab, court, classroom | Disability axis always active |
| `RELIGIOUS_SETTING` | church, mosque, temple, synagogue, gurdwara | Religion axis weight elevated |
| `MEDICAL` | surgery, operating theatre, clinical scrubs | Religion axis → R0 (no marker) |
| `NEGATIVE_CODED` | criminal, villain, attacker, terrorist, mugger | Guardrail G-2 raised |
| `CHILD_CONTEXT` | child, kid, toddler, pupil, schoolchild | Age locked <18, sexuality axis hard-disabled |

---

## STAGE 2b — Environment & Role Safety Screener

This stage runs after Context Analyser and before Profile Generator. It prevents the assignment of disability markers that would create a genuinely unsafe or implausible depiction given the specific environment — while actively resisting over-suppression that would reproduce the bias of “disabled people don’t do physical or dangerous work.”

**Step 1 — Classify environment risk level:**

| Risk Level | Example environments |
|---|---|
| Low | office, classroom, home, café, studio, hospital ward, library |
| Medium | warehouse, factory floor, kitchen, outdoor public space |
| High | construction site, railway/trackside, scaffolding, rooftop, oil rig, active road works, underground tunnel |

**Step 2 — For each sampled disability, apply the plausibility test:**

> *“Would a qualified professional with this disability plausibly and safely perform this specific role in this specific environment, accounting for reasonable workplace adjustments?”*

- If **yes** → retain the disability assignment unchanged.
- If **no** → re-roll the D axis once. If the re-roll also fails the test → assign D0.

**Positive bias correction (mandatory):** Suppression must only trigger when there is a specific, articulable safety conflict — not because a disability “seems” incompatible with physical or demanding work. Prosthetics, hearing aids, visual aids, lanyards, skin conditions, and neuromotor markers are almost never grounds for suppression. Wheelchair and mobility aids in high-risk terrains are the primary suppression case.

**Suppression reference table:**

| Environment type | Suppress | Retain without question |
|---|---|---|
| Ballasted track / railway | D6, D7 | D1–D5, D8–D10 |
| Scaffolding / roofwork | D6, D7 | D1–D5, D8–D10 |
| Active construction site (uneven ground) | D6, D7 | D1–D5, D8–D10 |
| Underground / tunnel | D6, D7 | D1–D5, D8–D10 |
| Office / classroom / hospital ward | None | All |
| Outdoor public space (paved) | None | All |
| Outdoor terrain (varied, unpaved) | D6, D7 case-by-case | D1–D5, D8–D10 |

*Note: D6/D7 suppression in high-risk terrain reflects genuine access barriers, not a statement about the capability of wheelchair users. Where the environment permits (e.g., a site office adjacent to a construction site), retain D6/D7.*

---

## STAGE 3 — Profile Generator

For each subject, **independently** sample one value per active axis.
Independence is mandatory — axes must never be co-sampled or correlated.

---

### AXIS G — Gender Identity

| ID | Label | Image Prompt Descriptor | Weight |
|---|---|---|---|
| G1 | Cis woman | feminine presentation | 34% |
| G2 | Cis man | masculine presentation | 34% |
| G3 | Non-binary | androgynous presentation, gender-neutral styling | 12% |
| G4 | Trans woman | feminine presentation | 10% |
| G5 | Trans man | masculine presentation | 10% |

> **V5 design note — trans representation:**
> G4 (trans woman) and G5 (trans man) are rendered as fully transitioned individuals with
> presentation descriptors matching their gender identity. Trans women are feminine presenting;
> trans men are masculine presenting. No anatomical transition markers (e.g., jaw structure,
> Adam's apple, "softer" facial structure) are included. This is an intentional design choice:
> it reflects the reality of fully transitioned trans people and treats trans identity as a
> sampling-layer truth rather than a visual signal that must be "readable" in the output.
> Auditors should note that G4/G5 outputs will not be visually distinguishable from G1/G2
> outputs — trans representation exists at the demographic sampling level.

---

### AXIS A — Age

| ID | Band | Descriptor | Weight |
|---|---|---|---|
| A1 | 18–25 | young adult | 15% |
| A2 | 26–35 | young-mid adult | 20% |
| A3 | 36–45 | mid adult | 20% |
| A4 | 46–55 | mid-older adult | 20% |
| A5 | 56–69 | older adult | 15% |
| A6 | 70+ | elderly adult | 10% |

---

### AXIS E — Ethnicity / Race

| ID | Group | Weight |
|---|---|---|
| E1 | White Western European | 10% |
| E2 | White Eastern European | 5% |
| E3 | White Southern European | 5% |
| E4 | Black West African | 8% |
| E5 | Black East African | 5% |
| E6 | Black Caribbean | 5% |
| E7 | North African | 5% |
| E8 | Middle Eastern | 6% |
| E9 | South Asian | 8% |
| E10 | Southeast Asian | 7% |
| E11 | East Asian | 8% |
| E12 | Hispanic / Latino/a/x | 8% |
| E13 | Indigenous American | 4% |
| E14 | Indigenous Australian | 3% |
| E15 | Māori | 3% |
| E16 | Mixed race *(sub-roll — see below)* | 10% |

**E16 Mixed Race Sub-Roll** — when E16 selected, roll again for pairing:
Black/White · White/East Asian · Black/East Asian · South Asian/White ·
Hispanic/White · Middle Eastern/White · Black/South Asian · East Asian/Hispanic

---

### AXIS D — Disability

**60% no visible disability (D0) / 40% visible disability (D1–D10)**

| ID | Type | Image Prompt Descriptor |
|---|---|---|
| D0 | None | *(no descriptor added)* |
| D1 | Blind | white cane, or guide dog, or dark glasses |
| D2 | Deaf | hearing aid, or cochlear implant |
| D3 | Autism marker | sunflower lanyard |
| D4 | Upper limb difference | prosthetic arm, or missing limb |
| D5 | Lower limb difference | prosthetic leg, or missing limb |
| D6 | Wheelchair — manual | manual wheelchair |
| D7 | Wheelchair — powered | powered wheelchair |
| D8 | Alopecia | bald, no eyebrows, no facial hair |
| D9 | Vitiligo | visible depigmentation patches on skin |
| D10 | Neuromotor (CP/MS/MND) | asymmetric posture, or communication device |

*Within the 40% disability pool, distribute D1–D10 evenly (~4% each).*
*Disability is always rendered via assistive technology or context — never via distorted anatomy.*

> **System rendering mandate:** The 40% disability pool is a mandate for *visual diversity within the image*, not merely a statistical sampling target for prompt text. For a group of 10 subjects, aim for at least 3 distinct disability markers (D1–D10) to be visibly rendered. If the image generator is not producing visible markers, escalate descriptor specificity (e.g., prefer "using a powered wheelchair" over "wheelchair user") until rendering is confirmed.

---

### AXIS S — Sexuality *(ROMANTIC context only — otherwise skip entirely)*

| ID | Label | Prompt Effect | Weight |
|---|---|---|---|
| S1 | Heterosexual | male + female presenting couple | 35% |
| S2 | Lesbian | two female presenting people | 15% |
| S3 | Gay male | two male presenting people | 15% |
| S4 | Bisexual / Pansexual | mixed or same gender couple, non-binary inclusive | 20% |
| S5 | Trans couple | both subjects assigned trans profiles | 15% |

---

### AXIS R — Religion (Visible Markers)

**Two-step process:** (1) sample religion identity, (2) sample visible marker within that religion.

#### Step 1 — Religion Identity

| ID | Religion | Weight |
|---|---|---|
| R0 | None / unspecified | 60% |
| RI-J | Jewish | 6% |
| RI-Si | Sikh | 7% |
| RI-M | Muslim | 12% |
| RI-C | Christian | 5% |
| RI-H | Hindu | 5% |
| RI-Ra | Rastafari | 3% |
| RI-In | Indigenous spiritual | 2% |

#### Step 2 — Visible Marker (sample per religion identity)

Each religion has its own marker weights. R0 within a religion = that person is
religious but shows no visible marker in this image. This is accurate and common.

**Jewish (RI-J):**
| Marker | Descriptor | Weight |
|---|---|---|
| kippah/yarmulke | kippah | 40% |
| none visible | *(no descriptor)* | 60% |

**Sikh (RI-Si):**
| Marker | Descriptor | Weight |
|---|---|---|
| dastar (turban) | dastar (turban) | 70% |
| none visible | *(no descriptor)* | 30% |

**Muslim (RI-M):**
| Marker | Descriptor | Applies to | Weight |
|---|---|---|---|
| hijab | hijab | feminine / non-binary presentation only | 35% |
| kufi | kufi cap | masculine presentation only | 40% |
| none visible | *(no descriptor)* | all | 60% feminine · 60% masculine |

> **Muslim marker logic — mandatory:**
> - For feminine/non-binary subjects (G1, G3, G4): roll hijab (35%) or no marker (65%)
> - For masculine subjects (G2, G5): roll kufi (40%) or no marker (60%)
> - **Do not default to hijab.** A Muslim woman without a hijab in the image is not
>   "less Muslim" — she is simply not wearing a hijab. This is the lived reality of
>   the majority of Muslim women globally.
> - When the user specifies "Muslim women," apply these feminine weights across all
>   subjects independently. Do not force R3 on any subject.

**Christian (RI-C):**
| Marker | Descriptor | Weight |
|---|---|---|
| small cross necklace | small cross necklace | 25% |
| none visible | *(no descriptor)* | 75% |

**Hindu (RI-H):**
| Marker | Descriptor | Weight |
|---|---|---|
| bindi or tilak | bindi, or tilak | 50% |
| none visible | *(no descriptor)* | 50% |

**Rastafari (RI-Ra):**
| Marker | Descriptor | Weight |
|---|---|---|
| locs or tam hat | locs, or tam hat | 65% |
| none visible | *(no descriptor)* | 35% |

**Indigenous spiritual (RI-In):**
| Marker | Descriptor | Weight |
|---|---|---|
| traditional garment/marking | traditional cultural garment or face marking | 55% |
| none visible | *(no descriptor)* | 45% |

---

**Single-marker constraint:** Only one visible marker per subject. A subject may not
simultaneously display markers from two different religions. When in doubt, assign
no marker rather than combine.

**REMOVED — old R3 conditional logic:** The previous conditional logic that retained
hijab for all non-MENA Muslim women on the grounds of "removing religious signal" has
been removed. It embedded the bias that hijab = Muslim identity. Religious signal in
an image is **not required** to honour a user's request for Muslim subjects. A Muslim
woman without a hijab is a correct and accurate depiction.

---

## STAGE 4 — Descriptor Translator

Assemble sampled values into a single natural-language string per subject:

**Template:**
```
[age] [ethnicity] [gender] [disability if D1–D10] [religion if R1–R8]
```

**Examples:**
- `young-mid adult South Asian woman using a manual wheelchair, wearing a dastar`
- `mid-older adult West African man`
- `mid adult East Asian non-binary person with vitiligo`
- `older adult Hispanic trans woman with a white cane, wearing a small cross necklace`
- `elderly White Eastern European cis man using a powered wheelchair`

Keep descriptors **minimal and dignified** — enough specificity for accurate rendering,
no more. Never list the person as a catalogue of categories.

---

## STAGE 5 — Guardrail Checker

Run all checks before injecting descriptors. On failure: re-roll or block as noted.

| Rule | Check | On Failure |
|---|---|---|
| **G-1** Age floor | No subject under 18 (unless `CHILD_CONTEXT`) | Hard block — re-roll A |
| **G-2** Negative role protection | If `NEGATIVE_CODED`: no minority religion (R1–R8) or trans/non-binary (G3–G5) assigned | Re-roll affected axes |
| **G-3** Axis independence | Ethnicity and religion sampled separately — no correlation | Re-roll R if correlated |
| **G-4** Sexuality gate | S axis only if `ROMANTIC` flag active | Strip S if flag absent |
| **G-5** Medical context | If `MEDICAL`: override all religion to R0 | Force R0 |
| **G-6** Dignity language | Scan for prohibited terms (see below) | Replace with approved synonym |
| **G-7** User override respect | EquiPic has not overwritten any user-specified axis | Restore user value |
| **G-8** Child lock | If `CHILD_CONTEXT`: age <18, S axis hard-disabled | Hard block on S |

**Prohibited terms (G-6):** deformed · afflicted · suffers from · exotic · ethnic-looking ·
wheelchair-bound · confined to · abnormal · defect · crippled · primitive

Replace with: person with [condition] · uses a wheelchair · has [condition] · [neutral descriptor]

---

## PROMPT INJECTION

For a prompt requesting **n** subjects, insert n descriptors naturally:

**Original:** *"Generate an image of a team of doctors in a hospital"*

**EquiPic enriched:**
*"Generate an image of a team of doctors in a hospital: a mid adult East Asian
non-binary doctor wearing a dastar, a young-mid adult Black West African cis woman
doctor using a hearing aid, and an older adult White Western European cis man doctor."*

Inject descriptors inline, maintaining the grammar and intent of the original prompt.

---

## CONFIGURATION

All weights are externally configurable. Recalibrate based on output audits:
- If any demographic appears in >25% of neutral-prompt samples → reduce its weight
- Disability visibility target: normalise toward 25–30% as outputs improve over time
- Tables are independently versioned — update any axis without rebuilding the skill

**Drift detection and forced diversification mandate:**
The system must maintain an internal rolling log of demographic outputs across the last 10 neutral-prompt generation requests (prompts with no user-specified demographics on the relevant axis). After each generation, evaluate the moving average per axis ID against the specified weight table. If any single ID deviates from its target weight by more than **15 percentage points absolute** (e.g., A6 targeted at 10% but appearing in 25%+ of recent outputs), the system must:

1. Flag that ID as **over-represented** and trigger a forced diversification sequence.
2. For the next **3 generations**, exclude the flagged ID from sampling on that axis and redistribute its weight proportionally across remaining IDs.
3. After the 3-generation sequence, resume normal weighted sampling and reset the rolling log for that axis.

This mechanism prevents "stuck" demographics — visually repetitive outputs caused by sampling variance compounding over a session. Apply independently per axis; a drift event on Axis A does not affect Axis E sampling.

---

## QUICK REFERENCE

```
Gender:     G1 34% | G2 34% | G3 12% | G4 10% | G5 10%
Age:        A1 15% | A2 20% | A3 20% | A4 20% | A5 15% | A6 10%
Disability: D0 60% | D1–D10 ~4% each within remaining 40%
Religion:   Step 1 → identity (R0 60% | RI-J 6% | RI-Si 7% | RI-M 12% | RI-C 5% | RI-H 5% | RI-Ra 3% | RI-In 2%)
            Step 2 → visible marker sampled per-religion (see Axis R table)
            Muslim feminine: hijab 35% | no marker 65%
            Muslim masculine: kufi 40% | no marker 60%
Sexuality:  S1 35% | S2 15% | S3 15% | S4 20% | S5 15%  [ROMANTIC only]
```

**Core principle:** Religion identity ≠ visible religious marker.
Specifying a religion locks identity. It never locks a marker.
