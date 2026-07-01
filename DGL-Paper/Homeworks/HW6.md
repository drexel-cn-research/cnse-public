## CS281 – Systems Architecture

### Homework 6: AI-Guided Investigation

### One ALU to Rule Them All? Why Processors Use Specialized Execution Units

---

### Overview

This homework is different from a typical problem set. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) as a learning tool to investigate a topic that builds directly on Lab 6 — but goes somewhere the lab never went.

**The topic:** In Lab 6 you built a clean, capable 32-bit ALU that handles AND, OR, ADD, SUB, and SLT by selecting among operations using a control input. It's a compact and elegant design. The natural follow-up question is: if you can add more cases to the control input, why not extend this same ALU to handle everything a processor needs — floating point arithmetic, multiplication, division, and more? And once you start investigating why that turns out to be harder than it sounds, a second question emerges: what does that mean for how a processor is actually organized?

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: Two Questions Lab 6 Doesn't Answer

In Lab 6 you built your ALU from 1-bit cells, each containing a carry chain, an AND gate, an OR gate, and a mux that selects among the results based on a control signal. That control signal is the key: by changing a single input, you choose which operation the mux passes through to the output. It's an elegant design — every operation you needed (AND, OR, ADD, SUB, SLT) maps cleanly onto the same underlying circuit. And in HW5 you saw how far that cleverness can stretch: subtraction didn't require a separate subtractor at all, just an XOR gate on each B input and a 1 on the first carry\_in, all driven by a single "subtract" control wire.

Now consider two things that follow naturally from that work, but that neither lecture nor the lab addressed.

**First, the extension question.** If the control signal idea worked so well for subtraction, why not keep going? You could add more cases: value 0 for AND, 1 for OR, 2 for ADD, 3 for SUB, 4 for floating point add, 5 for multiply, and so on — one ALU to handle everything the processor needs.

Here is the question to chase down:

*Your ALU computes A \+ B by propagating carry through a chain of 1-bit adder cells. Floating point numbers — the kind stored in your RISC-V F-extension registers — are stored in a completely different format (IEEE 754), where a number is split into a sign bit, an exponent, and a mantissa. When you add two floating point numbers, what steps does the hardware need to perform? Write out those steps and then ask yourself: could any of them be done by your carry-chain adder circuit? Where does the process break down, and what kind of hardware would you actually need?*

Getting a clear answer to that question — specifically what steps floating point addition requires that your ALU simply cannot do — is the core of the first thread.

**Second, the efficiency question.** In your single-cycle CPU, an instruction travels through the datapath from left to right — through the control logic, through the ALU, through memory access, and finally writes its result back to a register. Only after that full journey does the next instruction get to start. It works correctly, but think about what it means for all the hardware an instruction already passed through: once an instruction is halfway across the processor, everything behind it is sitting idle, waiting for it to finish.

Think about doing laundry. Suppose you have two loads. The straightforward approach: wash load 1, wait for it to finish, move it to the dryer, wait for it to dry — and only then start washing load 2\. The washer sits idle the entire time load 1 is drying. Nothing is broken, it just isn't very efficient.

The natural improvement is obvious: as soon as load 1 moves into the dryer, start washing load 2\. The two loads overlap. You didn't buy any new equipment — you just stopped leaving what you already have sitting idle.

Your CPU offers the same opportunity. As soon as an instruction has moved past the left side of the datapath, that circuitry is free. What if the processor started a new instruction through that freed-up hardware right away, so that two instructions are traveling through the processor at the same time — one further along, one just starting? This idea is called **pipelining**, and we'll be covering it formally in class. The intuition is the same as the laundry: don't leave hardware idle when there's work to do.

But even before we get there, you can already see where it leads. If two instructions are moving through the processor at the same time, what happens when they both reach the ALU simultaneously? The ALU is one piece of hardware — it can only process one thing at a time, and it's the most complex and time-consuming part of the circuit. It becomes the natural bottleneck.

Now combine that with what you found in Thread 1\. Suppose two instructions are moving through the processor together and both arrive at the ALU at the same moment: one needs integer addition (exactly what your Lab 6 ALU handles), and the other needs floating point addition (which, as you discovered, requires completely different hardware that your carry-chain circuit simply can't do). They can't share the same unit — they need fundamentally different hardware. And if both need to proceed without waiting, the only answer is to have more than one.

Here is the question to chase down:

*Start from the single-cycle world you built in lab: one instruction travels all the way through the processor, then the next one starts. Now imagine a processor where, as one instruction moves past the left side of the datapath, a second instruction begins right behind it — the two traveling through simultaneously, like two loads of laundry in different appliances at once. What would the processor need to check before letting two instructions travel through at the same time? Consider these two cases:*  

```asm
add  t0, t1, t2   	# t0 = t1 + t2
add  t3, t4, t5   	# t3 = t4 + t5
```

*These two instructions don't share any registers — neither uses the other's result. Now consider:*  

```asm
add  t0, t1, t2   	# t0 = t1 + t2
add  t3, t0, t5   	# t3 = t0 + t5  
                    #      ← uses t0 from the instruction above
```

*Here the second instruction needs a result that the first hasn't produced yet. What is the fundamental difference between these two cases, and what does a processor have to figure out before it can safely overlap two instructions? You don't need to go deep into how modern processors solve this — just explore what the core constraint is, and what "independent" means in this context.*

These two threads connect at the end. If you accept that a single ALU can't handle every type of arithmetic — and that a processor could have two instructions moving through it simultaneously, each needing the ALU at the same moment — then the idea of multiple specialized execution units stops being exotic and starts being obvious: an integer ALU for integer arithmetic, a separate floating point unit for IEEE 754 operations, a multiplier for integer multiplication. Each is optimized for one type of work, and each can operate independently when two instructions traveling through the processor at the same time need different things done at once.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Describe the IEEE 754 floating point format at a high level — sign, exponent, mantissa — and explain why it is structurally different from integer binary representation

2.    Identify the key steps required to add two floating point numbers and explain why those steps cannot be performed by a carry-chain integer ALU

3.    Explain why multiplication is also fundamentally different from addition at the circuit level — specifically what it produces that an adder cannot directly handle

4.    Describe at a conceptual level what it means to pipeline instruction execution — starting a new instruction through the processor as soon as the previous one has moved ahead, so that hardware is never left idle

5.    Explain what it means for two instructions to be independent, and why instruction independence is the fundamental constraint that determines whether two instructions can safely execute simultaneously

6.    Connect both threads to the organization of a real processor: why it has separate specialized execution units rather than one combined ALU, and how pipelining makes that specialization even more valuable

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the floating point / multiplication thread and the multiple execution units thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually connects the two threads. A weak log has surface-level questions with no iteration, or only covers one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "I just built a 32-bit integer ALU in Logisim that does AND, OR, ADD, SUB, and SLT using a control signal. Could I just add floating point addition as another case in the same design?" *Why I asked this:* Starting point — I wanted to know if my approach would even work before investigating why it might not.

> *Prompt 2:* "You said floating point addition requires aligning exponents first. What does that mean and why can't my carry-chain adder do that?" *Why I asked this:* The first answer listed the steps but didn't explain why each one was a problem for my specific circuit.

> *Prompt 3:* "If a processor has two separate execution units, how does it know which instructions can run at the same time? What is it checking?" *Why I asked this:* I wanted to understand the constraint before thinking about the solution.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — Why One ALU Can't Do Everything** What is the IEEE 754 floating point format, and what steps does floating point addition require? Identify at least two of those steps and explain specifically why a carry-chain integer ALU cannot perform them. Then briefly explain why multiplication is also structurally different — what does integer multiplication produce that an adder cannot directly handle in a single step?

**Part B — Pipelining and Multiple Execution Units** Describe the basic idea of pipelining in your own words — use the laundry analogy or one of your own. What does a processor gain by overlapping instruction stages? Then explain the key constraint: what does it mean for two instructions to be independent, and why does independence matter before two instructions can safely execute simultaneously? Give a concrete example of an independent pair and a dependent pair. Keep this at the conceptual level; you do not need to describe the full machinery of a modern processor — just the intuition of why overlapping works and what can go wrong.

**Part C — The Connection** In a short paragraph, explain how these two threads relate. If a processor can have two instructions traveling through it at the same time — like two loads of laundry in different appliances — what happens when both instructions arrive at the ALU simultaneously? How does the incompatibility you discovered in Thread 1 (floating point can't use your carry-chain ALU) combine with the insight from Thread 2 (independent instructions could overlap) to explain why real processors have separate, specialized execution units? What does your Lab 6 integer ALU represent in this picture, and what sits alongside it?

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

You built a 32-bit integer ALU in Lab 6 that performs AND, OR, ADD, SUB, and SLT. We haven't covered pipelining in class yet, but based on what you investigated in this assignment, start to envision a world where a processor doesn't execute one instruction completely before starting the next — where multiple instructions can be in different stages of execution at the same time, just like washing one load of laundry while another is drying. In that world, explain why your focused integer ALU is a deliberate design choice rather than a limitation. What would happen to circuit complexity and timing if you tried to fold floating point addition and integer multiplication into the same unit? And what does keeping the integer ALU focused allow a processor designer to do when two independent instructions reach the execute stage at the same time?

Your answer must be specific — reference actual properties of floating point or multiplication that create the incompatibility, and connect your answer to the pipelining intuition, not just "it would be too complicated."

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** threads; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanations of both threads plus a genuine connection in Part C. Requires specific identification of floating point steps that break the integer ALU, and a concrete example of dependent vs. independent instructions. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Answer is grounded in specific properties of floating point or multiplication, not just general statements about complexity. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the floating point thread:**

> *"I just built a 32-bit integer ALU in Logisim that performs AND, OR, ADD, SUB, and SLT using a control signal to select the operation. I'm wondering whether I could extend the same design to handle floating point addition — just add it as another case in the control input. Is that feasible, or is floating point addition fundamentally different from integer addition? I understand carry-chain adders but I don't know anything about floating point hardware yet."*

**To dig into what makes floating point different:**

> *"You listed several steps for floating point addition — can you walk me through why each one is a problem for a carry-chain integer ALU? I want to understand specifically which steps require hardware that is structurally different from what I built."*

**To investigate multiplication:**

> *"Why is integer multiplication also difficult to add to a standard carry-chain ALU? I know multiplication is repeated addition, but I've heard it requires different hardware. What does multiplication produce that makes it fundamentally different from addition at the circuit level?"*

**To start thinking about pipelining and execution overlap:**

> *"My single-cycle RISC-V CPU executes one instruction completely before starting the next — fetch, decode, execute, memory, writeback, then repeat. I've heard that real processors can have multiple instructions in different stages at the same time. What is the basic idea behind that, and what does a processor have to check before it can safely overlap the execution of two instructions?"*

**To dig into instruction independence:**

*"You said two instructions can overlap if they're independent. What exactly makes two instructions independent, and what happens if the processor starts executing an instruction before a previous one has written its result? Give me a concrete example of a dependent pair and an independent pair."*

**To connect to multiple execution units:**

> *"If a processor might have two instructions in the execute stage at the same time, and one needs integer addition while the other needs floating point addition, what does that imply about the hardware? Could both instructions use the same ALU, or do they need separate units?"*

