# Repetitive Thought Disengagement

Participants provided self-generated repetitive thought (RT) topics during an interview on a preceding session, which were converted to scanner stimuli. The fMRI task was completed on a subsequent visit, during which they were also instructed in worry disengagement strategies. The task comprised 3 runs of 9 trials each. For each trial, self-generated RT stimuli were presented for ~30s, followed by a 30s disengagement period in which participants completed a competing attention task (the Sustained Attention to Responses Task, SART). Following the SART, participants responded to thought probes including degree of uncontrollability of thoughts (i.e., disengagement success).

# Data included: #

All imaging data was collected on a Siemens MAGNETOM Prisma 3T MRI scanner with a 64-channel radio-frequency (RF) head coil array.

### Anatomical
T1-weighted structural images (1 mm3 voxels) were acquired in the sagittal plane with an isotropic MPRAGE sequence using a parallel imaging factor of 2 (TR=2500, TE=2.90, TI = 1070, flip angle = 8º, voxel size = 1.0mm3, matrix size 256x256).

### Functional
Functional data were collected as T2*-weighted gradient-echo echo-planar images with a multiband acceleration factor of 3 (TR = 2000, TE = 30, flip angle = 79º, echo spacing = 0.62, voxel size = 2.0mm3, matrix size 106x106, transversal orientation, 72 interleaved slices). Three runs were collected, each of duration 14m40s.

### Fieldmap
Two opposite phase-encoding (A>>P, P>>A) field maps were collected with resolution 3.0x3.0x2.0mm and matrix size 72x72.

# Task details: #

Each of the three runs presented all nine idiographic RT cues, counterbalanced with the constraint that low-uncontrollability trials were separated by two high-uncontrollability trials. **The same nine thoughts recur in every run, in a different order each time.**

Each trial comprised a 32s thought block (reminder cue presented), a 34–38s SART block of 17–19 trials, a 2s reflection period, thought probes, and a 2–4s fixation.

**SART.** Go trials presented digits 0–7 and 9; no-go trials presented the digit 8. Each stimulus appeared for 250ms followed by a fixation cross. Participants pressed a button after each stimulus unless it was an 8, in which case they withheld a response. Block length was jittered (17, 18, or 19 trials) to reduce expectancy.

**Thought probes.** At the end of each disengagement period, participants answered six prompts of the form "Just before the signal, to what extent were your thoughts ____?", responding on a visual slider from 0 to 100 using a five-button data glove held in the right hand. Probe 1 ("To what extent were your thoughts repetitive, intrusive, or hard to control?") operationalizes disengagement success in the associated manuscript.

# Privacy: removal of participant-generated reminder phrases #

**This is the most important difference between this version and earlier versions of the dataset.**

During the idiographic interview, participants created their own short reminder phrases as cues. Because these were participant-generated free text, some contained names of other people and other potentially identifying details. **All reminder phrase text has been removed from this release.**

In the task events files, the former `Thought` column has been replaced by `thought_uid`: a within-participant identifier (`t01`–`t09`) that labels which of the participant's nine thoughts a given block refers to, without revealing its content.

Key properties of `thought_uid`:

- It is **stable within a participant across runs**: the same thought carries the same uid in every run.
- It is **arbitrary across participants**: `t01` for one participant has no relationship to `t01` for another.
- It is **not** the same as `thoughtNumber`, which records within-run presentation order and is reshuffled between runs.

The mapping from `thought_uid` back to the original phrases is not distributed.

# Participants: #

Of 103 individuals who completed an initial phone/online screen, 94 underwent a semi-structured diagnostic interview and final eligibility determination, and 61 completed the fMRI study visit.

**This dataset includes all 61 participants who completed the scan.** The associated manuscript analyzes a subset of 57: four participants were excluded from analysis for insufficient responses on the tasks. In addition, two runs from one participant were excluded for motion (>25% of volumes censored). These participants and runs are retained here so that users can apply their own inclusion criteria; the `exclusion` column in `participants.tsv` indicates which participants were excluded and why.

The sample was transdiagnostic and enriched for clinically significant RT; 79% of participants with ADIS-5 data met criteria for at least one DSM-5 disorder.

# Repository structure: #

```
participants.tsv/.json          demographics and summary scores
dataset_description.json
task-PDT_events.json            column definitions for all events files
sub-<label>/ses-01/
    anat/    T1w
    func/    task-PDT and task-restingstate bold, task-PDT events, fieldmaps
phenotype/
    thought_ratings.tsv/.json   pre-scan ratings, one row per (participant, thought)
    adis.tsv/.json              ADIS-5 diagnoses, one column per disorder
    ptq.tsv/.json               PTQ-15 baseline totals and subscales
    ptq_screening.tsv/.json     PTQ-15 from the separate screening administration
    pswq.tsv/.json              PSWQ-16 totals
```

# Joining phenotype data to the events files: #

`phenotype/thought_ratings.tsv` is keyed on `participant_id` + `thought_uid`, the same pair that appears in every `*_events.tsv`. To attach each participant's pre-scan ratings for a given thought to the corresponding scanner blocks, merge on those two columns.

```python
import pandas as pd
events = pd.read_csv("sub-302/ses-01/func/sub-302_ses-01_task-PDT_run-01_events.tsv",
                     sep="\t")
ratings = pd.read_csv("phenotype/thought_ratings.tsv", sep="\t")
events["participant_id"] = "sub-302"
merged = events.merge(ratings, on=["participant_id", "thought_uid"], how="left")
```

# Pre-scan thought ratings: #

Before scanning, participants rated each of their nine thoughts on 36 items across four families, each on a 0 (not at all) to 9 (very much) Likert scale:

- `content_*` (13 items) — "The thought is about…"
- `form_*` (4 items) — "What form does the thought take?"
- `quality_*` (8 items) — "How ____ is the thought?"
- `emotion_*` (11 items) — "The thought makes me feel…?"

Full item wording is in `phenotype/thought_ratings.json`.

**Uncontrollability.** Two derived columns are provided: `dyscontrol_raw`, the mean of `quality_repetitive`, `quality_intrusive`, and `quality_uncontrollable` over non-missing items; and `dyscontrol_imputed`, the same with missing items imputed as 0.

Approximately 26% of thought-characteristic responses were missing because the survey software left non-endorsed items blank rather than recording a 0. Imputation was tested at every possible value (0–9); internal consistency was highest at 0 (Cronbach's α = 0.89) and lowest at 9 (0.81), supporting 0 as the intended value. `dyscontrol_imputed` is the variable used in the associated manuscript; results did not substantially differ between imputed and non-imputed data.

Note that `phenotype/thought_ratings.tsv` may contain rows for thoughts that were rated but never scanned; these will not match any `thought_uid` in the events files.

# Questionnaires and diagnoses: #

**Diagnoses.** Diagnostic status was established with the Anxiety and Related Disorders Interview Schedule for DSM-5 (ADIS-5; Brown & Barlow, 2014), administered at the first study visit by trained post-baccalaureate and graduate student diagnosticians. Final diagnoses were determined in weekly consensus meetings led by the PI. `participants.tsv` reports `n_diagnoses`; per-disorder binary columns are in `phenotype/adis.tsv`. Clinician severity ratings, which also capture subclinical presentations, are not included.

Diagnostic data, race, and ethnicity are unavailable for three participants, all of whom were excluded from the primary analysis sample.

**Trait scores.** The PTQ was administered twice, and both are released:

- `phenotype/ptq.tsv` — the **baseline** battery, collected at the first study visit. This is the administration contemporaneous with the scan session, and its total appears as `ptq_total` in `participants.tsv`.
- `phenotype/ptq_screening.tsv` — the **screening** administration, collected before enrollment and used to stratify recruitment by RT severity, along with the assigned severity stratum.

The two correlate approximately r = 0.76 but differ for individual participants, so they should not be treated as interchangeable.

The PSWQ-16 is in `phenotype/pswq.tsv`. Both PTQ and PSWQ totals are prorated: if at least 80% of items in a (sub)scale were completed, the total is the mean of completed items times the number of items; otherwise `n/a`. Item-level responses are not distributed.

# Preregistration: #

This dataset corresponds to pre-registered analyses which can be found here: https://osf.io/qy4df.

Analysis code: https://github.com/ceciwestbrook/RTDisengagement_public

# Data collection and acknowledgements: #

Data were collected at the CMU-Pitt Brain Imaging Data Generation & Education (BRIDGE) Center in Pittsburgh, PA, USA.
Funding was provided by the National Institute of Mental Health K01MH116328 (PI: Hallion).
All study procedures were reviewed and approved by the University of Pittsburgh Institutional Review Board.

# Authors: #

Cecilia A. Westbrook1, Brittany Alberts2, M. Kathleen Caulfield3, Mary Blendermann2, Peter J. Gianaros12, Jonathan Smallwood4, Jennifer S. Silk12, and Lauren S. Hallion5
1Department of Psychiatry, University of Pittsburgh, Pittsburgh, PA, USA
2Department of Psychology, University of Pittsburgh, Pittsburgh, PA, USA
3Reason Research, Philadelphia, PA, USA
4Department of Psychology, Queen's University, Kingston, ON, CA
5Department of Psychology, University of Regina, Regina, SK, CA
