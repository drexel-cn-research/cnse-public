# Deliberate Gap Learning (DGL)

This repository is a companion to a research project conducted in **CS281: Systems Architecture** at Drexel University. The project introduces and evaluates **Deliberate Gap Learning (DGL)**, an AI-era homework framework designed to preserve educational integrity in courses where generative AI can effortlessly solve conventional computing assignments.

## The Core Idea

DGL reframes homework around topics *intentionally left uncovered in lecture*. Rather than asking students to produce a correct answer, students are asked to investigate an unfamiliar topic through a structured dialogue with an AI assistant — and are graded on the *quality of their investigative process*, not the answer itself.

The framework rests on a simple observation: if an assignment can be completed by pasting the prompt into ChatGPT, it isn't a meaningful learning activity. DGL redesigns the assignment so that using AI *well* — asking follow-up questions, checking understanding, pushing back, refining — is exactly what the rubric rewards. Gaming the system requires roughly the same effort as authentic engagement.

## What's in This Repository

### Homework Assignments

The [`Homeworks/`](Homeworks/) directory contains all seven DGL assignments deployed in CS281, along with a summary table. Each assignment targets a topic bridging foundational lecture material toward more advanced depth — requiring students to use AI as a learning partner rather than a shortcut.

| HW | Topic |
|----|-------|
| [HW1](Homeworks/HW1.md) | CISC/RISC complexity & power |
| [HW2](Homeworks/HW2.md) | Cache hierarchy & memory locality |
| [HW3](Homeworks/HW3.md) | Interrupts & IoT power |
| [HW4](Homeworks/HW4.md) | RISC-V ISA extensions |
| [HW5](Homeworks/HW5.md) | Carry propagation & subtraction |
| [HW6](Homeworks/HW6.md) | Specialized execution units |
| [HW7](Homeworks/HW7.md) | CPU exceptions & privilege modes |

### Example Student Dialogue

[`Dennard-Scaling-Tutorial.md`](Dennard-Scaling-Tutorial.md) is an annotated example of a high-quality DGL dialogue. A student investigates Dennard Scaling across 11 turns — starting from zero background, asking follow-up questions, checking understanding, and progressively refining their mental model. Teaching notes at the end highlight the specific strategies that made the exchange effective.

This file is intended as a model for students beginning their first DGL assignment.

## Key Findings

The framework was evaluated across seven assignments with 105 enrolled students (538 survey responses):

- Deep AI engagement was sustained at **87–96%** with no drift toward low-effort behaviors over the semester
- Pooled Hake normalized gain was **ḡ = 0.46**, indicating medium-high conceptual learning
- Gains were higher for processor architecture units (ḡ = 0.54) than hardware design units (ḡ = 0.34)
- Students who engaged in active, critical dialogue with AI showed significantly higher learning gains than those who accepted initial responses without follow-up

## Contact

Brian Mitchell, Ph.D.  
Department of Computer Science, Drexel University  
https://www.cs.drexel.edu/~bmitchell
