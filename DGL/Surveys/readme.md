# DGL Survey Instrument

This directory documents the survey instrument used to collect the data reported in the paper — the exact questions and response scales presented to students after each of the seven DGL assignments. It contains only the instrument itself; no student responses, individual or aggregated, are included here or anywhere in this repository.

## Structure

Each of the seven surveys followed the same two-part design:

1. **Block 1 — assignment-specific questions** (three items per assignment, wording varies by topic): students rate their conceptual understanding of that assignment's specific topic both before and after completing it, and how clearly it connected to the preceding lab. See the per-assignment files below.
2. **Blocks 2–5 — standardized questions** (identical across all seven assignments): AI engagement quality, assignment design, overall satisfaction, and open-ended feedback. See [`common-questions.md`](common-questions.md).

| Assignment | Topic | Block 1 Questions |
|---|---|---|
| HW1 | CISC/RISC complexity & power | [HW1.md](HW1.md) |
| HW2 | Cache hierarchy & memory locality | [HW2.md](HW2.md) |
| HW3 | Interrupts & IoT power | [HW3.md](HW3.md) |
| HW4 | RISC-V ISA extensions | [HW4.md](HW4.md) |
| HW5 | Carry propagation & subtraction | [HW5.md](HW5.md) |
| HW6 | Specialized execution units | [HW6.md](HW6.md) |
| HW7 | CPU exceptions & privilege modes | [HW7.md](HW7.md) |

## Administration

The survey was distributed via Qualtrics after each assignment was submitted. Participation was voluntary and anonymous, incentivized with a small extra-credit opportunity: students received a unique receipt code on survey completion, which they submitted with their homework. See [`receipt-id-gen.md`](receipt-id-gen.md) for how these codes were generated and validated without linking any response to an individual student. This study was conducted under Drexel University IRB approval.

## Response Scales

Every rating question uses a 5-point scale, but the anchor labels are written specifically for that question rather than a generic agree/disagree scale (e.g., 1 = "No idea what these terms were" through 5 = "Understood these concepts well before this assignment"). The full anchor text for every question is shown alongside the question itself in the files linked above — nothing is abbreviated to "1–5."

## Relationship to the Paper

This instrument implements the survey design described in the Methodology section of *Beyond Lecture and Lab: Deliberate Gap Learning as an AI-Era Homework Framework*. Block 1 corresponds to the paper's "conceptual understanding" measure (RQ1); Block 2 corresponds to "AI engagement quality" (RQ2); Blocks 3–5 correspond to "perceived learning value" (RQ3).
