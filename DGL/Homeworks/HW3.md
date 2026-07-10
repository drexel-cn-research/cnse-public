### CS281 – Systems Architecture

### Homework 3: AI-Guided Investigation

### Why Your IoT Code Would Kill a Battery in Days: Polling, Interrupts, and the Hardware That Makes the Difference

---

### Overview

This homework continues the AI-guided investigation format from HW-1 and HW-2. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) to explore a topic that builds directly on what you did in Lab3 — but reveals a fundamental limitation in the approach you used, and the hardware mechanism that real embedded systems use to solve it.

**The topic:** Why polling — the button-checking strategy your Lab3 code uses — is completely impractical for real IoT devices, and how hardware interrupts solve the problem in a way that connects directly to the calling conventions, jump instructions, and function return mechanisms you've been learning in lecture.

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: A Problem Hidden in Your Lab Code

Take a look at the sleep function from the blinky.s sample program in Lab3:

```asm 
sleep:
	mv   t0, a0

sleep_loop:
	addi t0, t0, -1
	bnez t0, sleep_loop
	ret
```
This function doesn't actually sleep. It counts down from a number to zero, executing roughly a thousand instructions that do absolutely nothing useful, just to create a delay. The processor is running at full speed the entire time — fetching instructions, incrementing the program counter, burning power — to accomplish nothing except killing time.

Now look at the button-checking loop from button\_led.s. At its core, it continuously executes ecall 0x122 in a loop, asking "was a button pressed?" on every iteration. If no button is pressed, it loops and asks again. And again. And again — thousands of times per second, burning CPU cycles on a question that only needs answering when something actually changes.

This is called **polling**: repeatedly checking a status value in a loop rather than waiting for the hardware to notify you. For a program running on your laptop for a few seconds, polling is fine. For a real IoT device running on a battery in the field for years, it's a serious problem.

Consider the real-world context of RISC-V. The ISA was explicitly designed for embedded and IoT applications — the kind of systems where a processor runs in a smoke detector, an implanted medical sensor, a key fob, or a wireless temperature monitor. These devices run on small batteries for months or years. A processor constantly polling in a tight loop might draw tens of milliwatts. That same processor doing nothing — genuinely halted, waiting for an event — might draw only a few microwatts. The difference is a factor of 1,000 or more, which translates directly into whether your device lasts a week or a decade on a single charge.

The hardware solution to this problem is the **interrupt** — and understanding how interrupts work reveals a set of mechanisms built into the RISC-V processor that you haven't seen yet, but that connect directly back to things you already know.

As you investigate this, here is the specific question to chase down:

*Your Lab3 code used a sleep loop and a polling loop to simulate delays and check for button presses. In a real IoT device — a smoke detector, a medical implant, a wireless sensor — that approach is unusable. What mechanism does the hardware provide instead? How does the processor know when to stop doing nothing and start handling an event, and what happens inside the CPU in the first microsecond after that event occurs?*

Getting a clear, concrete answer to that question — in terms you can explain to someone else — is the core of this investigation.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Explain what **polling** is and describe a concrete scenario where it is impractical due to power consumption

2.    Explain what a **hardware interrupt** is: how a peripheral device signals the CPU, and what the CPU does in response

3.    Describe at a high level what an **Interrupt Service Routine (ISR)** is and why it must save and restore registers — connecting this to the calling conventions and ra register behavior you learned in lecture

4.    Explain what RISC-V's **WFI (Wait For Interrupt)** instruction does and why it exists as a distinct instruction in the ISA

5.    Give a concrete sense of the power difference between a polling loop and an interrupt-driven idle state, and explain why this matters for battery-powered IoT devices

6.    Recognize that ecall — which you used throughout Lab3 — is itself a form of interrupt (a software-triggered trap), connecting the lab work to the interrupt mechanism you investigated

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the interrupt mechanism thread and the power/IoT thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually reaches the connection between the two threads. A weak log is surface-level questions with no iteration, or questions that only cover one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "What is a hardware interrupt? I've heard the term but I only understand polling, where code checks a value in a loop." *Why I asked this:* Starting point — I needed to understand the basic concept before anything about power made sense.

> *Prompt 2:* "You said the processor 'jumps to an interrupt handler' — how does it know where to jump? Is that address hardcoded, or can software configure it?" *Why I asked this:* The first response mentioned an interrupt vector table but didn't explain how it works or who sets it up.

> *Prompt 3:* "When the interrupt handler finishes, how does the processor know where to return to? Is this the same as a normal function return using the ra register, or is it different?" *Why I asked this:* I wanted to connect this to what I already know about jal and ra from lecture — and to understand if the same mechanism applies.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — The Interrupt Mechanism** What is a hardware interrupt, and how is it fundamentally different from polling? Walk through what happens step by step when a hardware event (like a button press) triggers an interrupt on a RISC-V processor: how the peripheral signals the CPU, what the processor does before jumping to the handler, what an **Interrupt Service Routine (ISR)** is and why it must be careful about saving and restoring registers, and how execution returns to wherever it was before the interrupt occurred. Be specific about why the return-from-interrupt mechanism is different from a normal function return using ret.

**Part B — The Power Story** What does RISC-V's **WFI (Wait For Interrupt)** instruction do, and why does it exist? Give a concrete comparison of power consumption between a polling loop and a WFI-based idle state — include real or approximate numbers (current draw in milliamps or power in milliwatts) for an active RISC-V core vs. a halted/sleeping core. Then translate those numbers into something tangible: approximately how long could a device last on a typical small battery (look up or ask the AI for a reasonable battery capacity to use) if it is polling continuously versus if it is interrupt-driven with WFI?

**Part C — The Connection** In a short paragraph, explain how the two threads relate. Why do interrupts make WFI possible? What would happen if a processor executed WFI but had no interrupt configured — would it ever wake up? And here is a connection to bring back from Lab3: the ecall instruction you used to talk to LEDs and buttons is itself a type of interrupt — a software-triggered trap. Explain what that means and how it unifies what you just learned with what you did in lab.

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

Given what you now understand about interrupts and polling, identify **two specific things from Lab3** — code patterns, design decisions, or behaviors you observed — that look different to you now than they did when you were writing the lab. For each one, explain what you understood it to mean during the lab and what you understand it to mean now.

Your answer must be specific to Lab3. References to the sleep function, the button polling loop, the ecall instructions, the bare-metal setup-and-loop structure, or the boundary condition behavior are all fair game. Vague answers like "the lab used polling and now I know interrupts are better" earn partial credit only.

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** the interrupt mechanism and the power/WFI threads; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanation of the interrupt mechanism (Part A) and WFI/power story (Part B) with real or approximate numbers; Part C makes the ecall-as-trap connection explicitly. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Two specific Lab3 references with a genuine before/after framing — what it meant during the lab vs. what it means now. Generic "polling is bad, interrupts are good" earns half credit at most. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the interrupt mechanism thread:**

> *"I'm a CS student learning RISC-V assembly. I just wrote a lab where I checked for button presses by polling in a loop — the code keeps asking 'was a button pressed?' over and over. My professor said real IoT devices can't do this. What is a hardware interrupt, and how is it different from what I did? Assume I understand functions, registers, and how jal stores a return address in ra, but I don't know anything about interrupts yet."*

**To dig into the step-by-step mechanism:**

> *"Walk me through exactly what happens inside a RISC-V processor in the first few microseconds after a button press triggers a hardware interrupt. What does the hardware do automatically, what does software need to set up ahead of time, and how does the processor eventually return to what it was doing before the interrupt?"*

**To explore the power angle:**

> *"What is the WFI (Wait For Interrupt) instruction in RISC-V? How much power does a RISC-V processor actually save by executing WFI compared to spinning in a polling loop? Give me approximate real-world numbers — like milliwatts or microamps — for a typical small RISC-V microcontroller."*

**To chase the ecall connection:**

> *"My professor told me that ecall, which I used to control LEDs in a lab, is actually a type of interrupt. I don't see how — I thought ecall was just how you call OS services, like a function call. How is it related to hardware interrupts?"*
