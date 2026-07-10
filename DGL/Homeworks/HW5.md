## CS281 – Systems Architecture

### Homework 5: AI-Guided Investigation

### The Hidden Cost of a Chain: Carry Propagation and the Subtraction Trick

---

### Overview

This homework is different from a typical problem set. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) as a learning tool to investigate a topic that directly complements what we're covering in lecture and lab — but that we don't have time to go deep on in class.

**The topic:** In Lab 5 you built a 1-bit full adder from basic logic gates and then chained four of them together to make a 4-bit ripple carry adder. It worked correctly in simulation. What you didn't investigate is what that chain costs in terms of speed — and a second question the lab didn't ask: how does hardware perform subtraction using only the adder circuit you just built?

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: Two Questions Lab 5 Doesn't Answer

In Lab 5 you built a ripple carry adder by wiring four full adder cells in sequence, connecting the carry\_out of each stage to the carry\_in of the next. In Logisim the simulation runs instantaneously — you enter inputs, you get outputs, everything looks correct. But real hardware doesn't work that way. Each gate introduces a small delay, and when carry must travel from stage to stage, those delays stack up.

Now consider two things that follow naturally from what you built, but that neither lecture nor the lab addressed:

**First, the timing question.** Your 4-bit adder worked correctly in simulation, but in real hardware the final sum is not valid until the carry has finished rippling through every stage. With four stages that's manageable. But the ALU inside your RISC-V CPU operates on 32-bit values — which means carry would have to ripple through 32 stages. At the clock speeds modern processors run, that chain becomes a serious problem.

Here is a specific question to chase down:

> *You built a ripple carry adder by chaining carry\_out to carry\_in across all four stages. In real hardware, this means the output isn't valid until carry has propagated all the way through. What is this called, and why does it get dramatically worse as the adder gets wider? How does carry lookahead address this — and what does "lookahead" actually mean in this context?*

Getting a clear, concrete answer to that question — well enough that you could explain it to a classmate — is the core of the first thread.

**Second, the subtraction question.** Your adder does one thing: it adds. But processors need to subtract. The obvious solution would be to build a separate subtractor circuit. Real hardware doesn't do that — instead, it reuses the exact adder you built, with a surprisingly simple insight.

Start here: $8 − 5$ is the same as $8 + (−5)$. So if we can represent −5 in binary, we can subtract using nothing but addition. You covered two's complement in lecture — it tells you how to represent negative numbers in binary by inverting all the bits and adding 1. So −5 in two's complement is just: flip every bit of 5, then add 1.

That brings up two hardware questions: how do you flip all the bits of B, and where does the +1 come from?

For the +1, think about the adder you built in Lab 5\. Every full adder cell has three inputs: A, B, and carry_in. When you do ordinary addition you set the carry_in of the very first stage to 0 — that is just a wire tied to ground, and you probably never thought twice about it. But that input does not have to be hardwired. What happens if you set it to 1?

For flipping the bits, here is a hint. You used XOR gates to build your full adder. Recall the XOR truth table: 0 XOR 0 \= 0, 1 XOR 0 \= 1, 0 XOR 1 \= 1, 1 XOR 1 \= 0\. Notice something: when one input to an XOR gate is 0, the output equals the other input unchanged. When one input is 1, the output is the inverse of the other input. In other words, an XOR gate can act as a *controllable inverter* — depending on what you put on one of its inputs.

Here is the specific question to chase down:

> *Imagine you add a single input wire to your 4-bit adder — call it "subtract" — that is either 0 (for addition) or 1 (for subtraction). Each bit of B passes through an XOR gate along with this subtract signal before reaching the adder. When subtract \= 0, what does each XOR gate output? When subtract \= 1, what does it output? Now connect this subtract signal to the carry\_in of the first stage as well. Walk through a concrete example — say 8 minus 5 — and show that with subtract \= 1 the circuit produces the correct answer. What have you just built, and what did it cost in terms of extra hardware?*

These two questions are both rooted in the adder you just built. Your job is to understand what that circuit can and cannot do — and what a hardware designer can add to make it serve double duty.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Explain what **carry propagation delay** is, why it grows with adder width in a ripple carry design, and why that limits maximum clock frequency

2.    Describe conceptually how **carry lookahead** computes multiple carry bits in parallel rather than sequentially, and why that matters for real hardware

3.    Explain **two's complement** representation and how it makes subtraction possible using only an adder circuit

4.    Describe how inverting all bits of B and setting the first stage carry\_in to 1 produces A − B, and what minimal hardware additions an ALU needs to support this

5.    Connect both findings to ALU design: why real 32-bit ALUs use carry lookahead rather than ripple carry, and why a single adder circuit can handle both addition and subtraction

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the carry propagation thread and the subtraction thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually connects the two threads. A weak log has surface-level questions with no iteration, or only covers one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "I just built a 4-bit ripple carry adder in Logisim by chaining four full adder cells. My professor mentioned there's a timing problem with this design. What is carry propagation delay?" *Why I asked this:* Starting point — I needed to understand what the problem actually is before anything else made sense.

> *Prompt 2:* "You said carry ripples through each stage sequentially — how much worse does this get as the adder gets wider? Is a 32-bit adder eight times slower than a 4-bit one?" *Why I asked this:* I wanted to understand the scaling, since our RISC-V CPU does 32-bit arithmetic.

> *Prompt 3:* "You mentioned carry lookahead as the solution. I understand it's faster, but I don't understand what it actually computes differently. Can you explain the mechanism?" *Why I asked this:* The explanation said carry lookahead computes carries in parallel, but I didn't understand what that meant concretely.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — The Carry Propagation Story** What is carry propagation delay, why does it grow with adder width, and what does that mean for the maximum clock frequency of a circuit? Describe conceptually how carry lookahead addresses this. Include one concrete example: what is the worst-case 4-bit addition that forces carry to ripple through all four stages of your Lab 5 adder?

**Part B — The Subtraction Story** What is two's complement, and how does it make subtraction possible using only an adder? Explain specifically what happens when you set Binvert \= 1 and CarryIn \= 1 in a 1-bit ALU cell: what does Binvert do to the B input, and why does the result equal A − B? Verify your explanation with a concrete 4-bit example — pick two numbers, show the subtraction, and confirm the two's complement method gives the same answer.

**Part C — The Connection** In a short paragraph, explain how these two threads relate to the ALU design you will encounter in Lab 6\. Why does the 1-bit ALU cell have Binvert and CarryIn inputs? And why would a real 32-bit ALU use carry lookahead rather than the ripple carry approach you used in Lab 5?

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

In Lab 5 you built a 4-bit ripple carry adder from four 1-bit full adder cells, chaining carry\_out to carry\_in at each stage. Based on your investigation, pick **two specific things** a hardware designer would need to add or change to make that adder circuit support both addition and subtraction, and explain the reasoning behind each one.

Your answer must be specific and grounded in what you investigated — for example, what needs to happen to the B input, what needs to change about the first stage's carry\_in, and why. Vague answers like "you need extra hardware" earn partial credit only.

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** carry propagation and subtraction; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanations of both threads plus a genuine connection in Part C. Requires a concrete worst-case carry example and a worked numeric subtraction example. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Two specific connections to the Lab 6 ALU cell, each grounded in actual signal names or structural decisions. Generic answers earn half credit at most. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the carry propagation thread:**

> *"I'm a CS student and I just built a 4-bit ripple carry adder in Logisim by chaining four full adder cells together, connecting carry\_out to carry\_in at each stage. My professor mentioned there's a timing problem with this design in real hardware. Can you explain what carry propagation delay is and why it becomes a serious problem as the adder gets wider? Assume I understand logic gates and binary addition, but I don't have an electrical engineering background."*

**To dig into carry lookahead:**

> *"You explained that carry propagation delay gets worse as the adder gets wider. I've heard that carry lookahead solves this — but I don't understand what it actually computes differently from a ripple carry adder. Can you explain the mechanism concretely? What does 'lookahead' mean in this context?"*

**To start the subtraction thread:**

> *"My ripple carry adder can only add. I know from lecture that A − B equals A \+ (−B), and that two's complement gives you −B by flipping all bits and adding 1\. So in theory I could compute A − B using my adder — but I need to flip all the bits of B and get a \+1 into the circuit somewhere. My adder has a carry\_in at the first stage that I've just been hardwiring to 0\. Is there something useful I can do with that?"*

**To push toward the XOR insight:**

> *"OK so setting the first carry\_in to 1 gives me the \+1. But I still need to flip all the bits of B. I've been told to think about XOR gates — can you explain how an XOR gate could act as a controllable inverter, and how I'd use that here?"*

**To confirm with a concrete example:**

> *"Can you walk me through 8 minus 5 using this approach? Show me the bit-level steps: what the B bits look like after the XOR gates when the subtract signal is 1, what goes into the adder, and what comes out."*
