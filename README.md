# CareLoop Clinics — No-Show Guard

## Project overview

CareLoop Clinics operates eight outpatient clinics across six Indian cities. This project analyses appointment data to understand where and when no-shows occur, builds a model to identify appointments at higher risk of being missed, and uses Gemini AI to turn call-centre notes into structured reasons. The goal is to help the clinic prioritise follow-up calls and make better use of appointment slots.

**Business question:** Which appointments are most likely to be missed, why do patients miss them, and what should CareLoop do first?

## Key findings

1. **Part 1 — Data preparation:** The raw file contained 1,020 rows; after cleaning, 990 appointments remained, including 218 no-shows (22.0%).
2. **Part 2 — SQL analysis:** The overall no-show rate was 22.02%; the highest rate was 33.3% at Gomti Nagar Care Point (90 appointments), while the highest-volume clinic in the ≥95-appointment comparison was Kothrud Family Clinic (160 appointments, 22.5%).
3. **Part 3 — Machine learning:** Logistic regression was selected with a training cross-validation ROC-AUC of approximately 0.77; at the 0.35 threshold, test ROC-AUC was approximately 0.79 and test recall was approximately 0.57.
4. **Part 4 — AI-assisted note review:** Gemini analysed the first 10 notes; 5 of 10 linked appointments had no reminder recorded, and both notes classified as `forgot_or_no_reminder` (2 of 2) were linked to appointments with `reminder_sent = No`.

## Business recommendation

CareLoop should pilot an evening-before follow-up process that prioritises appointments with a predicted no-show probability of at least **0.35**. The cleaned dataset contains **990 appointments**, of which **218 (22.0%)** were no-shows. The selected logistic regression model achieved a test **ROC-AUC of approximately 0.79** and **recall of approximately 0.57** at the chosen threshold, flagging about **49 of 198** test appointments for follow-up. Booking lead time is an important operational signal: appointments booked **31+ days** ahead had a **60.5%** no-show rate (26 of 43), compared with **16.2%** for appointments booked 1–7 days ahead (63 of 390). The SQL analysis also found a **33.3%** no-show rate at Gomti Nagar Care Point, although it had only **90 appointments**, so that rate should be interpreted cautiously. In the 10-note illustration, **5 of 10** linked appointments had no reminder recorded, and both notes classified as `forgot_or_no_reminder` were associated with `reminder_sent = No`. The clinic should therefore start with a small pilot: use the model to rank next-day bookings, contact the highest-risk patients within available call capacity, and track attendance and call outcomes. The note sample is too small to justify clinic-wide conclusions about reasons, and the association between reminders and attendance does not establish causation.

## Selected model and threshold

- **Selected model:** Logistic regression with `StandardScaler` in a scikit-learn pipeline.
- **Model-selection method:** Models were compared using 5-fold cross-validation ROC-AUC on the training set. Logistic regression achieved the highest score at approximately **0.77** (the tuned random forest was also approximately 0.77 in the reference run).
- **Selected classification threshold:** **0.35**.
- **Threshold rationale:** Out-of-fold training results gave an F1 score of approximately **0.49** at 0.35, compared with approximately **0.35** at 0.50. Recall increased from approximately **0.25** to **0.47**, which is important because missing a genuinely high-risk appointment can leave a doctor’s slot empty.
- **Final test results:** ROC-AUC approximately **0.79**, recall approximately **0.57**, precision approximately **0.57**, and about **49 of 198** test appointments flagged at the 0.35 threshold.

The threshold should ultimately be set according to the number of follow-up calls the front desk can make each evening and the relative cost of an empty appointment slot.

## Part 4 — AI note analysis

The first 10 follow-up notes were processed using the Gemini model configured in the notebook:

`MODEL_NAME = "gemini-3.5-flash-lite"`

The output was structured as JSON and checked with a Pydantic model. The allowed `root_cause` categories were:

- `forgot_or_no_reminder`
- `distance_or_transport`
- `cost_or_fee`
- `illness_or_emergency`
- `work_or_schedule`
- `other`

The blank N004 note was skipped rather than sent to the API. The expected classifications for the non-blank notes were checked against the assignment's reference outcomes. The note analysis is an illustration based on 9 non-blank notes, not a representative estimate of all missed appointments.

## Limitations

1. **Small note sample:** The 10 notes are an illustration and cannot show the real reasons for missed appointments clinic-wide.
2. **Patient overlap:** The same patient can appear in both the training and test sets, so the test evaluation may be less representative of performance on entirely new patients.
3. **Observational data:** Associations in this dataset do not prove that reminders or other factors cause a change in no-show behaviour.

## Project files

Include the notebooks, source data permitted for submission, and generated deliverables required by the assignment. Key generated files include:

- `rejected_rows.json`
- `appointments_clean.csv`
- `clinics_enriched.csv`
- `load_data.sql`
- `test_risk_scores.csv`
- `notes_analysed.csv`
- `notes_appointments_merged.csv`
- `root_cause_reminder_crosstab.csv`
- `README.md`

## Reproducibility and security

- The project uses `random_state=42` and an 80/20 train/test split (`test_size=0.2`).
- The model threshold is selected using training-set out-of-fold predictions; the test set is used for final reporting only.
- The Gemini API key is entered at runtime and must not be committed to the repository. Clear notebook outputs and search the complete notebook for the key before uploading.
- Results from the GenAI note analysis can vary between runs; the saved `notes_analysed.csv` records the actual model responses used for the final cross-tabulation.

## Disclaimer

This project is for educational and analytical purposes. Model predictions should support, not replace, clinic staff decisions.
