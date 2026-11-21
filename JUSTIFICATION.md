# JUSTIFICATION — the design of the harness, and why

## 1. What the system may do

The system screens synthetic scholarship applications and may only inform a decision: it returns shortlist or reject with a score, and anything in the score band 40-49, any unparseable reply and any reply my extra rule flags goes to a person. It decides on its own only outside that band (342 shortlist and 24 reject in the harness run, 34 of 400 sent to a person).

Controls and where they are in the code (the notebook, Parts 2a-2d):

- blind: `BLIND = ("name", "university")` (Part 2a). Made because Part 1 found a consistent +2.5 score advantage for the flagship university (95% CI +1.1 to +4.1, sign test p = 0.002) and 4 of 4 gender-pair flips favouring the female-coded name, although none of the flip rates stood clear of its floor.
- validate_input: the built-in `validate_profile`, switched on in `Harness(...)`.
- refuse: the built-in `refusal_reason` (purpose, minor, consent) plus my `my_extra_refusal` (Part 2b), which refuses instruction-like text in the name, university and program fields and malformed names.
- log: `log_path="decisions.jsonl"`, one record per call.
- escalate: the band `BORDERLINE = (40, 49)` plus `my_extra_escalation` (Part 2c).

A constraint changed the architecture: I chose to hide the name and the university rather than only flag disagreement between runs. I considered the counterfactual check (re-ask with a swapped name), which doubles the model calls; I did not run it (`RUN_EXTENSION = False`), because blinding costs no extra calls (400 model calls in both runs).

## 2. What it must refuse

It refuses, and the refusals fired in the submitted run (Part 4: 5 of 6 requests refused, 0 model calls each):

- a purpose other than scholarship screening (`purpose_not_allowed`), tested with a request to rank applicants by ethnicity;
- an applicant under 18 (`minor`);
- no consent to automated processing (`no_consent`);
- impossible values (`invalid_input`), tested with GPA 9.9 and exam score -5;
- instruction-like text in a free-text field (`instruction_in_text_field`), tested with a name that contains an instruction to the model.

Which legitimate case each one also blocks: an applicant who does not consent cannot be screened by the system at all; a person whose name contains a digit or a symbol would be refused as `malformed_name`. Of the 400 clean applicants in the measured pairs my rules refused 0, but that sample contains no such names. The cost falls on the honest applicant, who loses a decision. I accept it only if a human can decide instead; that route is not built into the code. A refused request is recorded in `decisions.jsonl` with its reason code, but the code does not tell the applicant or offer a human decision, so I list it under what the design fails to protect against.

## 3. What record it keeps

The audit question: why did this applicant get this outcome, and can someone reconstruct it without re-running the model?

Each record in `decisions.jsonl` (406 records: 400 from the measured run and 6 from Part 4) holds a record id, the application id, the SHA-256 of the exact text the model saw, the fields shown and the fields withheld, the model id and prompt version, the harness settings, the raw reply, the decision and score, the controls fired, the outcome and reason code, whether it went to a person, and a timestamp. For example, a record with `reason_code: borderline` and `escalated_to_human: true` shows a reject at score 40 sent to a person, and a `purpose_not_allowed` record shows `[no model call]` and no decision.

What is left out on purpose: the values of the fields (only their names), so the log is not a second copy of the applicants' data. The raw reply may still quote what the model was shown.

What the record does not give: the applicant's name or university (so an audit cannot check which university an applicant came from), and it does not show whether a person read an escalated case or what they decided. I did not implement a retention period, access control or protection against editing: the log is a plain file that anyone who holds it can change. That is a gap.

## 4. The legal constraints

The instruments I rely on, and the provision for each control, are in `CONTROL_REGISTER` (`control_register.json`). In short:

- Law of the Republic of Kazakhstan No. 94-V of 21 May 2013 "On Personal Data and their Protection": Art. 7(4) (processing limited to specific, predefined and lawful purposes), Art. 7(5) (data excessive in relation to the purposes must not be processed), Art. 8(1) (consent). The text I read contains no rule on automated decisions, so I cannot cite this law for escalation.
- Law of the Republic of Kazakhstan No. 230-VIII of 17 November 2025 "On Artificial Intelligence": Art. 6 (exclude discrimination), Art. 8(1) (continuous control over the system), Art. 9 and Art. 17(2) (human autonomy, human correction of decisions), Art. 15(2) (risk management and documentation). It applies if the fund operates in Kazakhstan; whether a fictional fund is such an operator is an assumption I did not verify.
- Benchmarks, not binding on this prototype: Regulation (EU) 2016/679 Art. 22(1) and 22(3), Regulation (EU) 2024/1689 Art. 12(1) and Art. 14(1), and Annex III point 3(a) (systems that determine access or admission to education are classified as high-risk). I did not read Annex III myself; I found it only in a secondary description, so I do not rely on it.

What the law requires of the system and what it leaves to me: purpose, consent and data minimisation are requirements of the Kazakh personal-data law; the minimum age of 18, hiding the name and the university, the score band and my text-field rules are my own design. For the minor rule I found no provision that sets an age limit. For validate_input I found no provision that requires input validation specifically; the nearest is the duty of risk management and reliability measures in Art. 15(2) of Law 230-VIII, and I call it a weak fit.

## 5. The ethical principles

- Fairness and non-discrimination (OECD AI Principles, human-centred values and fairness; UNESCO Recommendation on the Ethics of AI, fairness and non-discrimination): bears on the decision to hide the name and the university.
- Accountability and traceability (OECD AI Principles): bears on `decisions.jsonl`.
- Human oversight (UNESCO Recommendation, human oversight and determination): bears on the escalation to a person.
- Privacy and data protection (UNESCO Recommendation): bears on consent and purpose.

Both documents are recommendations and not laws: they bind no one, and I use them as the source of the principles (OECD Recommendation of the Council on Artificial Intelligence, OECD/LEGAL/0449, adopted 2019 and revised in 2024; UNESCO Recommendation on the Ethics of Artificial Intelligence, adopted 23 November 2021).

Conflict: treating applicants identically (blinding) versus treating them fairly given what I measured. Blinding makes the pair flips 0 by construction, so I can no longer measure whether the model treats the groups differently. I put preventing possible discrimination first and accept that I lose the ability to test it. Ethics demands something the law does not: the name and the university are not in the list of grounds in Art. 6 of Law 230-VIII in so many words, yet I treat them as attributes that must not matter.

## 6. The trade-offs

- blind: no extra model calls (400 in both runs), but 166 of 400 scores changed (mean |delta| 14.01) and 46 of 400 outcomes changed: reject to escalate 22, shortlist to escalate 12, shortlist to reject 7, reject to shortlist 5. Removing lines also changes the form of the prompt, so I cannot say the model relied on the hidden fields. It stops the +2.5 score advantage for the flagship university from acting, and it costs the model the information those fields legitimately carry.
- escalate: 34 of 400 (8.5%) went to a person against 16.0% planned (the band was calibrated on the unblinded scores and does not transfer to the blinded prompt). The 24 rejects outside the band are decisions nobody re-checks. My extra escalation rule never fired in this run, so I have no evidence that it works.
- refuse: no model calls, so no cost in calls; the cost is the honest applicants refused and the missing route to a human.
- log: a cost in storage and in applicants' privacy, limited by keeping field names and not values.

I chose this balance because reading 8.5% of the applicants is a workload a small fund can carry, and because blinding is the cheapest control. I would choose differently if a larger sample showed no effect (then blinding would cost more than it prevents), or if reviewers could not read 8.5% of the applicants.

What the design still fails to protect against: a larger or different effect that 40 pairs could not detect (fewer than 6 pairs, 15%, cannot be told from chance); real applicants and other models; other irrelevant details in the prompt; a log with no retention period, access control or tamper protection; refused applicants who are not told and cannot ask for a human; and an extra escalation rule I never saw fire. Closing these needs a larger sample, a second prompt wording, a signed or append-only log and a defined appeal route; I did not build them because of time.

## AI-use declaration

I used Claude (Anthropic) throughout this assignment. AI explainet whta i shpould do, and helped me to translate the diffucult words and termins. All code, answerrs, files  writed by mysalfe
