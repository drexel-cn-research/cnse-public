#### CS281 – Systems Architecture

### Homework 4: AI-Guided Investigation

### The RISC-V Menu: Why Your Simulator Has "IM" in Its Name and What That Letter Cost Someone

---

### Overview

This homework continues the AI-guided investigation format from HW-1, HW-2, and HW-3. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) to explore a question that has been hiding in plain sight all semester — one that Lab4 puts directly in front of you, if you know where to look.

**The topic:** Why RISC-V is designed as a modular family of extensions rather than a single fixed instruction set, what it costs in real silicon to add an instruction, and how the C (compressed) extension adds 16-bit instructions to a processor that was supposed to have fixed-width 32-bit instructions — and does so without repeating x86's variable-length decode problem.

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: A Reveal Hidden in Lab4

Look carefully at what Lab4 asked you to do.

**Part 1** had you implement a multiply(a, b) function using nothing but repeated addition:

```
multiply(a, b) = a + multiply(a, b-1)
```

No multiply instruction. No shortcut. Just add called over and over until the recursion bottoms out. This is slow — multiplying 5 × 200 requires 200 recursive calls and hundreds of executed instructions — but it works. And it works on any RISC-V processor, no matter how stripped-down.

**Part 3** handed you a single instruction:

```
mul rd, rs1, rs2
```

One instruction. Done in one cycle (or close to it). The result you computed in hundreds of steps now happens in essentially no time.

Here is the thing you weren't told during the lab: **those two approaches exist because two different chips are out there.** The mul instruction only works because the course simulator is an **RV32IM** processor — and that "M" is not free. It means the chip designer chose to include hardware multiply circuitry. A different RISC-V chip, one designed for a cheap sensor or a disposable medical device, might have skipped the M extension entirely. On that chip, your Lab4 Part 1 approach is not a warmup exercise — it is the only option, the code that actually ships.

That "M" you've been seeing all semester is a letter in a naming system that describes exactly which optional features a RISC-V chip provides. You've been programming an RV32IM processor. Real commercial chips have names like RV32IMAC or RV32G. Every letter in that string is a deliberate design decision, and every decision has a cost.

This homework investigates two connected questions: why does that modular system exist at all, and what happens when you look at one of its stranger letters — the C extension, which adds 16-bit compressed instructions to a processor that was designed around fixed-width 32-bit instructions?

*Your course simulator is an RV32IM processor. Lab4 Part 1 asked you to implement multiply from scratch, and Lab4 Part 3 handed you the mul instruction. Why do both exist? What does it cost to include hardware multiply in a chip? And if RISC-V is defined by clean fixed-width 32-bit instructions, how does the C extension add 16-bit versions of them without turning into the kind of variable-length nightmare you learned about in HW-1?*

Getting a clear, concrete answer to both threads — in terms you can explain to someone else — is the core of this investigation.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Explain what the letters in a RISC-V designation like **RV32IM** or **RV32IMAC** mean — what each letter represents and why those letters exist as a naming system

2.    Describe at least **two concrete costs** of adding an instruction to a processor's hardware design — touching on silicon area, power consumption, verification burden, or design complexity

3.    Explain why a RISC-V chip designer might deliberately **omit** the M extension, and give a realistic example of a device where bare RV32I is the right choice

4.    Describe what the **C (Compressed) extension** adds, why smaller instruction encoding is valuable in embedded systems, and how it achieves 16-bit instructions without the variable-length decoding problems that make x86 complex

5.    Explain the **two-bit detection mechanism** that lets hardware distinguish a 16-bit C instruction from a full 32-bit instruction

6.    Connect the Lab4 design — Part 1 (software multiply) alongside Part 3 (mul) — to the real-world tradeoff between bare RV32I and RV32IM processors

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the extension ecosystem thread and the C-extension/compressed-instructions thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually reaches the connection between the two threads. A weak log is surface-level questions with no iteration, or questions that only cover one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "What does it mean when a RISC-V chip is called RV32IM? I know it's 32-bit, but what does the I and M stand for, and why are those separate letters rather than just being part of the base design?" *Why I asked this:* Starting point — I used mul in Lab4 Part 3 but never understood why it wasn't just a standard instruction like add.

> *Prompt 2:* "You said the M extension adds hardware multiply. How much does that actually cost in terms of chip area or power? I want to understand why a chip designer would skip it." *Why I asked this:* The first response said it "adds hardware" but didn't explain what that means physically or what the tradeoff is.

> *Prompt 3:* "I've heard that RISC-V has a C extension for compressed 16-bit instructions. But my professor spent a lot of time this semester explaining that fixed-width 32-bit instructions are one of the things that makes RISC-V clean. How do 16-bit instructions fit into that without making things messy?" *Why I asked this:* This seemed like a contradiction and I wanted to understand how they resolved it.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — Why Modular?** Explain why RISC-V uses optional extensions rather than building everything into a single fixed instruction set. What are the real costs of adding an instruction to a processor? Walk through at least two specific costs — for example, silicon area, power consumption, formal verification burden, or time-to-market — and explain concretely what each one means. Then give a realistic scenario where a chip designer would deliberately choose bare RV32I over RV32IM: what kind of device would it be, and why would leaving out hardware multiply be the right call? Finally, contrast this with the x86 approach — what happened when Intel kept adding instructions over 40 years without a modular system?

**Part B — The Compressed Instruction Trick** Explain what the C (Compressed) extension adds and why it exists. What problem does smaller code size solve for embedded systems — concretely, what does it mean when a device has only 64KB of flash memory and the C extension can reduce code size by 25–30%? Then explain the key design question: how does RISC-V add 16-bit instructions to a processor that was built around 32-bit fixed-width encoding? Describe the two-bit detection mechanism — specifically, what the two least-significant bits of any instruction tell the hardware about how long that instruction is. Why does this avoid the x86 variable-length decode problem you studied in HW-1?

**Part C — The Connection** In a short paragraph, explain how the two threads relate. The M extension and the C extension were both added to solve different problems — M for performance, C for code density — but they are both examples of the same design philosophy. What is that philosophy, and what does it reveal about who RISC-V is designed for? Why does the existence of both extensions in the same chip (RV32IMC) make sense, even though they seem to pull in different directions?

---

### Section 3 — Critical Check (20 pts)

Find **one claim the AI made** — in either thread — that felt imprecise, oversimplified, or that you weren't sure was accurate. Describe:

1.    What the AI said

2.    Why it felt off or incomplete to you

3.    What you did to check it (a follow-up prompt asking for more precision, a different query to the same or a different AI, or a quick external source)

4.    What you concluded — was the AI basically right, partially wrong, or misleading?

This section doesn't require you to catch the AI being wrong. It requires you to *try* — to read critically, notice something worth questioning, and follow up on it. That habit is the skill.

A natural candidate: the AI may make a claim about how much area or power the M extension adds to a chip. Push on that. Is it a fixed number? Does it depend on the implementation? Are there RISC-V chips that implement multiply in a small area vs. a high-performance way?

---

### Section 4 — Connect It Back (20 pts)

In **3–5 sentences**, answer this question:

Lab4 asked you to implement multiply in two different ways: Part 1 used repeated addition via recursion, and Part 3 used the mul instruction. You could not have used mul in Part 3 if the course simulator were a bare RV32I processor. Now that you understand the extension model: **why did your professor ask you to implement multiply by hand in Part 1 at all?** What does that implementation represent in the real world — on what kind of actual device would your Part 1 code be the real, shipping implementation rather than just an exercise? And what would you have to change about Part 3 if the simulator were switched to a bare RV32I chip?

Your answer must engage specifically with Lab4. Vague answers like "the extension model explains why mul exists" earn partial credit only. The strongest answers will connect Part 1's software multiply to a concrete device type (think about who actually ships bare RV32I), explain what RV32IM means as a design decision, and name what would have to change in Part 3\.

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** the extension ecosystem thread and the C-extension thread; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanation of why modular ISA design exists (Part A, with at least two concrete costs) and how the C extension works (Part B, including the two-bit detection mechanism); Part C connects the two threads with the underlying design philosophy. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Engages specifically with Lab4 Parts 1 and 3; identifies a realistic device type for bare RV32I; explains what RV32IM means as a design decision. Generic responses earn half credit at most. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the extension model thread:**

> *"I'm a CS student learning RISC-V assembly. My course simulator is called an RV32IM processor. I know mul is a multiply instruction and I used it in a lab. But my professor also had us implement multiply from scratch using repeated addition, which suggests mul isn't always available. What does the IM in RV32IM mean, and why would a RISC-V chip ever not have hardware multiply?"*

**To dig into the cost of adding instructions:**

> *"You said the M extension adds hardware multiply circuitry. What does that actually cost on a real chip — in terms of silicon area, power consumption, or design verification? I want to understand the engineering tradeoff well enough to explain why a cheap IoT sensor might ship without it."*

**To chase the C extension thread:**

> *"I've been told that one of RISC-V's strengths is fixed-width 32-bit instructions — simpler decode, easier pipelines, no x86 variable-length mess. But I've also heard there's a C extension that adds 16-bit compressed instructions. How does that work without turning RISC-V into the kind of variable-length ISA we were told to avoid?"*

**To nail down the detection mechanism:**

> *"Walk me through exactly how a RISC-V processor with the C extension knows whether the next instruction is 16 bits or 32 bits when it fetches it from memory. What bits does it look at first, and how does this differ from how x86 handles variable-length instructions?"*

**To bring it back to Lab4:**

> *"If I ran my Lab4 Part 3 code — which uses the mul instruction — on a bare RV32I chip with no M extension, what would happen? And what would the Part 1 approach (multiply via repeated addition) look like from that chip's perspective — is that still a valid implementation?"*

