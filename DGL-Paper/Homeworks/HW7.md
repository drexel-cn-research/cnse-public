## CS281 – Systems Architecture

### Homework 7: AI-Guided Investigation

### When Programs Go Wrong: CPU Exceptions and Hardware Fault Handling

---

### Overview

This homework is different from a typical problem set. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) as a learning tool to investigate a topic that builds directly on Lab 7 — but goes somewhere the lab never went.

**The topic:** In Lab 7 you built a single-cycle RISC-V CPU. It fetches an instruction, decodes it, executes it, and moves on to the next one. The design assumes everything goes right: every instruction is valid, every memory access succeeds, every operation completes without incident. Real programs don't work that way. They execute invalid instructions, access memory they aren't allowed to touch, and encounter conditions the hardware cannot handle silently. The CPU needs a way to deal with all of this — and that mechanism turns out to be more interesting than it looks, because it also answers a question from HW3: why does ecall work the way it does?

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: Two Questions Lab 7 Doesn't Answer

In Lab 7 you built a single-cycle CPU that handles a specific set of RISC-V instructions. Your datapath fetches each instruction, the control unit sets the appropriate signals, the ALU performs its operation, and the result is written back. Every path through your circuit assumes that the instruction is legal and that everything the instruction touches is accessible.

Now consider two things that follow naturally from what you built, but that neither lecture nor the lab addressed.

**First, the fault question.** Your CPU has no way to handle something going wrong. But real programs fail in predictable ways: a program might execute an instruction your hardware doesn't recognize, attempt to load from an address it isn't permitted to access, or trigger an arithmetic condition the hardware needs to report. The CPU cannot simply freeze or produce garbage output. It needs to detect the problem, preserve enough state to recover, and transfer control to a piece of code — called an exception handler — that knows what to do next.

Here is the mechanism to chase down:

*Your single-cycle CPU always moves from one instruction to the next by incrementing the PC. But when an exception occurs, the CPU needs to do something very different: save the current PC somewhere safe, record what went wrong, and jump to a handler routine at a known address.*

*In class and lab, we have talked about registers x0–x31 — the general-purpose registers that your programs use to hold values, pass arguments, and store results. But real processors contain additional registers that programmers never touch directly. These extra registers exist to support the processor itself when it needs to reach out to other software — most often the operating system — for help on what to do next. When the CPU encounters something it can't handle on its own, it needs a way to hand off control and provide enough context that the OS can make a decision.*

*RISC-V provides a dedicated family of registers for exactly this purpose, called Control and Status Registers (CSRs). Three of them are central to exception handling: mepc saves the PC of the instruction that caused the exception (so the OS knows where in the program things went wrong), mcause records a code identifying what type of exception occurred (so the OS knows what happened), and mtvec holds the base address of the exception handler (so the CPU knows where to jump). Investigate how this mechanism works: what does the hardware do, step by step, when an exception is detected? What information must be saved, and why? And once the handler finishes, how does the CPU know where to return?*

Getting a clear, step-by-step picture of what the hardware does during an exception — not the software handler, but the hardware actions that happen before the handler even runs — is the core of the first thread.

**Second, the privilege question.** In HW3, you used ecall to request services from the operating system. You saw that ecall causes control to transfer to a handler, and that the handler can do things your user program cannot. At the time, that might have seemed like a software convention. It isn't. The distinction between what user code can do and what the operating system can do is enforced in hardware through privilege modes.

RISC-V defines multiple privilege levels. The most fundamental distinction is between machine mode, which has unrestricted access to all hardware resources including the CSRs you just investigated, and user mode, which cannot directly access those registers or certain memory regions. When your user program executes ecall, the processor doesn't just jump to a different address — it elevates the privilege level, switching from user mode to machine mode, so the handler runs with full hardware access.

Here is the question to chase down:

*Think back to how you have been using ecall in your RISC-V programs all semester. You load a service number into a7, put any arguments into a0 and a1, execute ecall, and things just happen — characters appear on the console, user input arrives, an LED turns on. From your program's perspective, it's a complete black box: set up some registers, make the call, and trust that the result appears. But that abstraction is hiding a lot.*

*Here is the question worth chasing: where does the software that actually performs those actions live, and how does it get enough context to do its job? For straightforward cases like writing to the console, the answer is relatively simple — the parameters your program loaded into registers before the ecall are still there when the handler runs, so the handler reads them and does the work. You saw this pattern in earlier labs.*

*But exceptions are different. When an exception occurs, your program didn't ask for anything. The CPU detected a problem on its own and needs help. There's no a7 value to consult, no argument registers set up by the programmer. Instead, the CPU needs to provide its own context — and that's exactly what the Control and Status Registers (CSRs) introduced in the first thread are for. The handler reads mcause to find out what went wrong and mepc to find out where.*

*This brings up a key question: what stops a user program from writing to those CSRs directly — setting mtvec to point wherever it wants, or overwriting mepc before a handler runs? The answer is that access to the CSRs is restricted by privilege mode, and the hardware enforces that restriction. Investigate: what is the difference between machine mode and user mode in RISC-V? What can machine mode code do that user mode code cannot? When ecall is executed, what does the hardware actually do to enforce the privilege transition — and why does it matter that this boundary is enforced in hardware, not software?*

These two threads connect at the end. The exception mechanism and the privilege system are not separate features — they are two halves of the same design. Exceptions are how the hardware hands control to a trusted handler, and privilege modes are how the hardware ensures that only trusted code can set up that mechanism in the first place.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Describe what a CPU exception is and give at least two concrete examples of conditions that trigger one in RISC-V

2.    Explain the role of mepc, mcause, and mtvec in RISC-V exception handling, and describe the sequence of hardware actions that occur when an exception is detected

3.    Explain what CSRs (Control and Status Registers) are and why they are a separate register file from the general-purpose integer registers

4.    Describe the difference between machine mode and user mode in RISC-V, and explain what the hardware prevents user mode code from doing

5.    Connect the hardware exception mechanism to the ecall instruction from HW3: explain why ecall causes a privilege transition, and how that transition is enforced in hardware rather than software

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the exception mechanism thread and the privilege modes thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually connects the two threads. A weak log has surface-level questions with no iteration, or only covers one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "I just built a single-cycle RISC-V CPU in Logisim. It assumes every instruction is valid and every memory access succeeds. What happens in a real CPU when an instruction fails — like if the instruction code isn't recognized?" *Why I asked this:* Starting point — I needed to understand what the hardware is supposed to do before I could understand how it does it.

> *Prompt 2:* "You said the CPU saves the PC and jumps to a handler. Where does it save the PC, and how does it know where the handler is?" *Why I asked this:* The first answer mentioned saving state but didn't explain the specific mechanism — I wanted to know what hardware registers are involved.

> *Prompt 3:* "You mentioned mtvec holds the handler address. Who writes that value in the first place, and when? Can a user program write to it?" *Why I asked this:* I realized I didn't understand who sets up the exception mechanism — that led me toward the privilege question.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — The Exception Mechanism** What is a CPU exception? Give two concrete examples of conditions that can cause one. Then walk through the sequence of hardware actions that occur when an exception is detected in RISC-V: what happens to the PC, what gets recorded and where, and where does the CPU jump? Explain the role of mepc, mcause, and mtvec. Finally, explain how the CPU returns from the exception handler once the handler finishes.

**Part B — Privilege Modes** What is the difference between machine mode and user mode? Describe at least one thing that machine mode code can do that user mode code cannot. Explain how the hardware enforces this boundary — what prevents a user program from simply accessing the CSRs directly? Connect this back to HW3: when a RISC-V program executes ecall, what does the hardware do to the privilege level, and why does that matter?

**Part C — The Connection** In a short paragraph, explain how these two threads relate. Why does the exception mechanism depend on privilege modes existing in the first place? What would go wrong if user programs could freely read and write mtvec or mepc? And what does this tell you about the relationship between your Lab 7 CPU — which has neither exceptions nor privilege modes — and a CPU capable of running real software?

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

Your Lab 7 CPU implements the "happy path" — it assumes every instruction is valid, every memory access succeeds, and nothing unexpected ever happens. Based on your investigation, describe what would need to be added to your datapath to support even a minimal exception mechanism. At a minimum: what new registers would be needed, what would each one hold, and what signal or condition would trigger saving the current PC? You do not need to design the full system — just identify the key additions and explain the role of each.

Your answer must be specific and grounded in what you investigated — reference actual register names and what each one stores, not just "you'd need extra hardware for fault handling."

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** threads; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanations of both threads plus a genuine connection in Part C. Requires the step-by-step exception sequence and a specific explanation of what privilege mode enforcement prevents. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Answer identifies specific registers by name, states what each holds, and describes what triggers the exception response. Generic answers earn partial credit only. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the exception mechanism thread:**

> *"I just built a single-cycle RISC-V CPU in Logisim for a systems architecture class. The CPU I built assumes every instruction is valid and every memory access works — it only handles the happy path. In a real RISC-V processor, what happens when something goes wrong, like an unrecognized instruction or an illegal memory access? Walk me through what the hardware does, step by step, before any software handler runs."*

**To dig into the CSR mechanism:**

> *"You mentioned that RISC-V saves the PC to a register called mepc and jumps to an address stored in mtvec when an exception occurs. Can you explain what these registers are, why they're separate from the normal integer registers like x0–x31, and who is responsible for setting mtvec in the first place?"*

**To investigate the return from an exception:**

> *"Once the exception handler finishes running, how does the CPU get back to where it was? Is there a specific instruction for this in RISC-V, and does the CPU restore its privilege level when it returns?"*

**To start the privilege mode thread:**

> *"In a homework earlier this semester I used the ecall instruction to make system calls. My professor said ecall causes a privilege transition in hardware. What does that mean? What is the difference between machine mode and user mode in RISC-V, and what does the hardware actually prevent user mode code from doing?"*

**To connect privilege modes to the exception mechanism:**

> *"If a user program could write directly to mtvec, what could go wrong? Why does it matter that only machine mode code can set up the exception handler address?"*
