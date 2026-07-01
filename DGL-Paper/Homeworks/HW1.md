## CS281 – Systems Architecture

#### Homework 1: AI-Guided Investigation
#### The Hidden Cost of Intel/AMD: Complexity and Power

---

### Overview

This homework is different from a typical problem set. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) as a learning tool to investigate a topic that directly complements what we're covering in lecture — but that we don't have time to go deep on in class.

**The topic:** Why modern Intel/AMD processors are so much more power-hungry than ARM or RISC-V processors — and how the internal complexity of the x86-64 Instruction Set Architecture (ISA — the complete specification of what instructions a processor can execute and how they are encoded) is a big part of the reason why.

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: Two Questions Lecture Doesn't Answer

In lecture we've established that RISC-V instructions are simple and fixed-width (32 bits each), and that the processor fetches one instruction at a time, executes it, and moves on. The model is clean and regular.

Now consider two things that follow naturally from that but that we haven't dug into:

**First, the complexity question.** Intel's x86-64 ISA — the Instruction Set Architecture used by virtually every Windows and Linux laptop and desktop — has hundreds of instructions, variable-length encoding (anywhere from 1 to 15 bytes per instruction), and decades of backward compatibility requirements going back to the original 8086 chip from 1978\. Yet modern Intel processors execute these instructions extremely fast. How? It turns out they don't actually execute x86 instructions directly — they secretly translate them into simpler internal operations called **micro-operations (µops, sometimes written "micro-ops")** first. That translation layer adds significant hardware complexity. This style of ISA design — lots of complex, powerful instructions — is called **CISC (Complex Instruction Set Computing)**. RISC-V, by contrast, is a **RISC (Reduced Instruction Set Computing)** architecture — fewer, simpler instructions that hardware can execute directly without translation.

**Second, the power question.** Look up the **TDP (Thermal Design Power — the maximum amount of heat, measured in watts, that a processor's cooling system needs to be able to dissipate during normal operation; it's used as a proxy for power consumption)** of a recent Intel Core i9 processor: somewhere around 125–253 watts depending on the model. Now look up a recent Apple M-series chip or a high-end ARM server chip — you'll find numbers like 15–30 watts for comparable workloads. Why such a dramatic difference? A modern laptop running an Intel chip gets warm and runs its fans hard under load. An iPad running Apple Silicon stays cool and silent. This isn't just a manufacturing story — it's an architecture story.

As you investigate this, here is a specific question to chase down — you don't need an electrical engineering background to understand the answer, but it is the key to unlocking the whole power story:

*Processor clock speeds are measured in GHz — a modern Intel chip might run at 3, 4, or 5 GHz, meaning billions of cycles per second. Why does running a processor at a higher clock speed require more power? And for decades, making transistors smaller seemed to solve that problem automatically — until around 2004–2006, when it suddenly stopped working. What changed, and why did that force the industry to rethink everything?*

Getting a clear answer to that question — in terms you can explain to someone else — is the core of the power thread.

These two questions are connected. Your job is to find out how.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Explain what a micro-operation (µop) is and why Intel/AMD processors use them

2.    Describe roughly where in instruction execution the x86-to-µop translation happens and what hardware it requires

3.    Explain what **Dennard Scaling** (the observation that as transistors shrink, their power consumption shrinks proportionally — allowing clock speeds to increase without increasing total chip power) is, why it ended in the mid-2000s, and what that meant for processor power consumption

4.    Describe the key architectural reasons why RISC processors (ARM, RISC-V) tend to be more power-efficient than CISC processors (x86) running equivalent workloads

5.    Connect both threads back to why RISC-V is designed the way it is — fixed-width instructions, a small and regular instruction set, 32 general-purpose registers

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the micro-ops thread and the power thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually reaches the connection between the two threads. A weak log is surface-level questions with no iteration, or questions that only cover one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "What is a micro-operation in the context of Intel processors?" *Why I asked this:* Starting point — I needed the basic definition before anything else made sense.

> *Prompt 2:* "You said x86 instructions are decoded into µops — where exactly does that happen, and does every x86 instruction produce the same number of µops?" *Why I asked this:* The first answer was vague about timing and quantity. I wanted something concrete.

> *Prompt 3:* "How much power does the µop translation hardware itself consume? Is that a significant fraction of the chip's total TDP (Thermal Design Power)?" *Why I asked this:* I wanted to bridge from the complexity topic to the power topic.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — The Micro-Op Story** What are micro-operations (µops), why does the x86 CISC (Complex Instruction Set Computing) ISA need them, and what does the hardware that handles this translation look like at a high level? Include one specific example of an x86 instruction that decodes into multiple µops and approximately how many.

**Part B — The Power Story** What is Dennard Scaling, when did it end, and why did that create a power problem for processors? What are the main architectural reasons RISC (Reduced Instruction Set Computing) processors like ARM and RISC-V tend to use less power than x86 processors running equivalent workloads? Include at least one real TDP (Thermal Design Power) comparison between an Intel processor and an ARM or RISC-V chip — look one up to confirm what the AI told you.

**Part C — The Connection** In a short paragraph, explain how the two threads relate. How does the CISC complexity of x86 (and the hardware needed to manage it) contribute to its higher TDP? What does RISC-V give up by not having that complexity, and what does it gain?

---

### Section 3 — Critical Check (20 pts)

Find **one claim the AI made** — in either thread — that felt imprecise, oversimplified, or that you weren't sure was accurate. Describe:

1.    What the AI said

2.    Why it felt off or incomplete to you

3.    What you did to check it (a follow-up prompt asking for more precision, a different query to the same or a different AI, or a quick external source)

4.    What you concluded — was the AI basically right, partially wrong, or misleading?

This section doesn't require you to catch the AI being wrong. It requires you to *try* — to read critically, notice something worth questioning, and follow up on it. That habit is the skill.

---

### Section 4 — Connect It Back (20 pts)

In **3–5 sentences**, answer this question:

Given what you just learned about CISC (Complex Instruction Set Computing) micro-op complexity and the x86 TDP (Thermal Design Power) problem, pick **two specific design choices** in RISC-V — things from the 1a-Intro or 2-RISCV-Programming lectures — and explain how each one is a direct response to the problems you investigated.

Your answer must reference specific features from lecture (for example: the fixed 32-bit instruction width, the 32-register file, the load/store model, the clean separation of R-type and I-type instruction formats). Vague answers like "RISC-V is simpler and uses less power" without specifics earn partial credit only.

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** micro-ops and power; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanations of both threads (Parts A and B) plus a genuine connection (Part C). Requires a specific µop count example and a real TDP comparison. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Two specific RISC-V design choices from lecture, each clearly linked to a problem investigated. Generic "RISC is better" earns half credit at most. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the micro-ops thread:**

*"I'm a CS student learning about RISC-V processors. My professor mentioned that Intel/AMD processors translate x86 instructions into something called micro-operations (µops) before executing them. Can you explain what that means and why they do it? Assume I understand what an instruction and a register are, but I don't know anything about processor pipelines yet."*

**To start the power thread:**

*"Why do Intel Core processors have such a high TDP (Thermal Design Power) compared to ARM processors like the Apple M-series chips? I've heard this is partly an architecture issue, not just a manufacturing issue. What's the architectural explanation?"*

**To understand the frequency-power connection (no EE background needed):**

*"I'm a CS student with no electrical engineering background. Can you explain simply why running a processor at a higher clock speed — say 4 GHz instead of 1 GHz — requires more power? What is physically happening inside the chip that causes this? And why did making transistors smaller used to solve this problem automatically, and what changed around 2004–2006 that made it stop working?"*

**To bridge the two threads:**

*"Is there a connection between the micro-op translation complexity in x86 CISC processors and their higher TDP? How much does the front-end decoding hardware contribute to the overall power budget?"*
