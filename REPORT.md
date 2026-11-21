# REPORT — what the measurement showed, who it affects, and whose responsibility it is

## 1. Measurement read-out

I ran the supplied agent (Qwen2.5-3B-Instruct, greedy decoding) on 40 matched pairs per attribute (name language, gender-coded name, university) plus two floors (null_name, null_university): 200 pairs, 400 profiles. 0 of 400 replies were unparseable; 356 of 400 profiles (89%) were shortlisted.

| attribute | flips (of 40 pairs) | rate [95% CI] | its floor | floor flips |
|---|---|---|---|---|
| name_language | 2 | 5.0% [1.4%, 16.5%] | null_name | 0/40 |
| gender_coded_name | 4 | 10.0% [4.0%, 23.1%] | null_name | 0/40 |
| university | 1 | 2.5% [0.4%, 12.9%] | null_university | 1/40 |

All 95% intervals overlap their floors. Nothing stands clear of its floor on the decision measure, so I do not claim that any attribute moved the decision.

What the numbers allow me to say:

- gender_coded_name: all 4 flips favoured the female-coded name (male-coded first favoured 0, other favoured 4, McNemar p = 0.125); shortlisted 90.0% female vs 80.0% male. But the mean |score diff| is 1.12 [0.25, 2.13], below its floor's 2.00 [0.62, 3.62], and with 40 pairs the exact test needs at least 6 pairs in one direction to reach p < 0.05, so 4 pairs could not be significant. It is a direction worth watching, not an established effect.
- university: flips equal the floor (1/40 each), but the signed score shift flagship minus regional is +2.5 points (95% CI +1.1 to +4.1, sign test p = 0.002). The mean |diff| (2.50 [1.12, 4.12]) is close to the floor's 2.12 [0.88, 3.62]. I did not print the signed shift of the null_university floor, so I cannot say whether the direction exceeds the floor; I report it as a consistent small direction.
- name_language: mean |diff| 3.12 [1.38, 5.25] against 2.00 [0.62, 3.62] for the floor; no clear effect (Kazakh-language names vs the others: 2 vs 0, McNemar p = 0.5).

What n = 40 could not detect: an effect moving fewer than 6 pairs (15%) cannot be told from chance when none go the other way, and fewer than 8 (20%) when 1 goes the other way. So "no measured effect" is not "no effect". The floors cover only a different name of the same group and a different university of the same tier. They do not cover a different prompt wording, other irrelevant details, another model or real applicants, and this run cannot say how the attributes would behave there.

Decisions are also noisy in their own right: the model rejected only profiles scoring 20-49, and the 40-49 score band held both outcomes (33 reject, 31 shortlist), which is where flips occur.

## 2. Who is affected

- A wrongly rejected applicant loses a place on the shortlist, and in a fund with limited places a wrongly shortlisted applicant displaces someone else. In my data the suggestive direction for gender (4 of 4 flips favouring the female-coded name; 90.0% vs 80.0% shortlisted) would fall on male-coded applicants, and the score direction (+2.5) would fall on applicants from regional universities. Neither is established by my run; they are the groups who would bear the cost if the pattern is real and my sample could not see it.
- The cost is not symmetric: a rejected applicant may never learn why or apply again, while a wrongly shortlisted applicant costs the fund reviewer time and displaces another applicant.
- Others are affected without being in the brief: the reviewers who read escalated cases (34 of 400 profiles, 8.5%, in the harness run), applicants who never apply because they expect to be rejected, and applicants from universities the model has never seen.
- With the harness on, the 24 rejects outside the escalation band are decisions nobody re-checks, and 7 applicants moved from shortlist to reject when the prompt form changed, for reasons not attributable to a protected attribute.
- Used year after year, a small consistent score shift compounds: an advantage of 2.5 points near a threshold decides who is shortlisted, and the output may feed later reviews.

## 3. Professional responsibility

As the developer I am responsible for testing for divergence before the system runs (I did it with matched pairs on 40 pairs per attribute, a sample that cannot detect effects below 15%); for telling the people who deploy it what the test could not cover (other prompt wordings, other models, real applicants); and for not claiming the harness is fair. Hiding the name and the university makes the pair flips 0 by construction, so the after-table does not show that the model became fair, and I did not measure how the new prompt behaves on the hidden fields.

What is left to do: a larger sample, a second prompt wording, and recalibrating the escalation band (the share sent to a person was 8.5% against 16.0% planned).

Responsibilities that belong to the deploying institution: setting the purpose, choosing and training the human reviewers, handling appeals, and deciding how many applicants reviewers can read. I do not control them.

Where the two conflict: if the institution wanted to remove a control because it slows the process (for example blinding, or the escalation of 8.5% of applicants), I would tell them what my measurement showed it prevents and what I could not measure, and I would not describe the system as fair without it. I cannot enforce that; I can only document it.
