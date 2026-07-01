## CS281 – Systems Architecture

### Homework 2: AI-Guided Investigation

### The Secret Speed of Sequential Code: Cache, Memory, and Locality

---

### Overview

This homework continues the AI-guided investigation format from HW-1. You are going to use an AI assistant (ChatGPT, Claude, Gemini, or any other) to explore a topic that builds directly on what you've been doing in lecture and in Lab2 — but that we don't have time to develop fully in class.

**The topic:** Why your Lab2 array code runs much faster than a version that does the same work in a different order — and the hidden hardware mechanism responsible for that gap.

**The goal is not to produce the right answer from a single prompt.** The goal is to conduct a genuine investigation: start with a question, get an explanation you don't fully understand, ask a follow-up, push back when something doesn't make sense, and keep going until you can explain it yourself. You will show your work, and that process is what gets graded.

---

### Background: Two Questions Lecture Doesn't Answer

In lecture we've established that a processor fetches instructions from memory, executes them, and that data also lives in memory — accessed via explicit lw and sw instructions in RISC-V. The model treats memory as a large, flat array of bytes that the processor can read from or write to at any address.

But there's a problem with that picture: memory is slow. A modern processor running at 3–4 GHz can complete an instruction in one or two cycles. Fetching a value from main memory (DRAM — the actual chips on your motherboard) takes roughly 200–300 cycles. If every lw instruction had to wait that long, your processor would spend most of its time doing nothing.

Now think about what you wrote in Lab2. Your code scanned through an array one word at a time, loading each element with a sequence like:

```asm
lw   t1, 0(s0)   	# load element at current pointer
addi s0, s0, 4   	# advance pointer to next word
```

Every iteration executes a lw instruction against a different memory address. If memory truly took 200+ cycles every time, Lab2 would run orders of magnitude slower than it does. It doesn't — and there's a reason for that.

**First, the hardware question.** Between the processor and main memory, modern CPUs contain several layers of very fast, very small memory called **cache** (pronounced "cash"). Cache is invisible to the RISC-V ISA — there are no cache instructions, and your assembly code has no idea it exists. When your lw instruction runs, the hardware silently checks the cache first. If the value is there (a **cache hit**), you get it in a handful of cycles. If it isn't (a **cache miss**), the hardware fetches it from the slower next level. Most programs, most of the time, are hitting cache far more often than missing it — and understanding why is the first thread of this investigation.

**Second, the locality question.** Cache hardware doesn't just remember values you've accessed before — it bets on which values you're going to need next. When you load a word at address X, the hardware automatically loads a chunk of nearby addresses too, in anticipation. That chunk is called a **cache line**. This is a deliberate bet: if you just accessed address X, you'll probably access X+4, X+8, and X+12 very soon. In your Lab2 array code — stepping through addresses sequentially with addi s0, s0, 4 — that bet turns out to be right, every single time.

As you investigate this, here is the specific question to chase down:

*In Lab2 you wrote RISC-V assembly that steps through an array one element at a time, loading each word in order. Why does this particular access pattern — addresses 4, 8, 12, 16 in sequence — run dramatically faster than accessing the same array elements in a scrambled order? What is the hardware mechanism responsible for that difference, and how big is the performance gap in practice?*

Getting a clear, concrete answer to that question — in terms you can explain to someone else — is the core of this investigation.

---

### Learning Objectives

By the end of this assignment you should be able to:

1.    Describe the CPU cache hierarchy (L1, L2, L3, DRAM) and explain the speed vs. capacity tradeoff that motivates having multiple levels

2.    Explain what a **cache hit** and a **cache miss** are, and give approximate latency numbers for each level of the hierarchy

3.    Define **spatial locality** and **temporal locality** and give a concrete example of each from your own experience with arrays in assembly

4.    Explain what a **cache line** is and how its size determines why sequential memory access patterns are faster than random access patterns

5.    Connect the cache hierarchy back to the load/store model of RISC-V: every lw instruction is either a cache hit or a cache miss, and your code's access pattern determines which

---

### What to Submit

Your submission is a single document (PDF or markdown) containing the four graded sections below, followed by the optional Student Assessment. **There is no length requirement** — clear and complete beats padded and vague.

---

### Section 1 — Your Investigation Log (30 pts)

List **6 to 10 prompts** you sent to the AI during your investigation, covering **both the cache hierarchy thread and the locality thread**. Don't copy the full responses — just the prompts you wrote, in the order you sent them. After each prompt, write one sentence explaining why you asked that follow-up (what you were trying to clarify or push deeper on).

The point is to show a progression of inquiry across both topics. A strong log starts with a basic question, iterates based on what you learned, and eventually reaches the connection between the two threads. A weak log is surface-level questions with no iteration, or questions that only cover one of the two topics.

**Example of what we're looking for (do not copy this):**

> *Prompt 1:* "What is a CPU cache and why does it exist?" *Why I asked this:* Starting point — I needed the basic definition before anything else made sense.

> *Prompt 2:* "You said there are multiple cache levels — L1, L2, L3. Why not just have one really big fast cache instead of three?" *Why I asked this:* The first answer mentioned the hierarchy but didn't explain why multiple levels are needed instead of one.

> *Prompt 3:* "You mentioned cache lines — what exactly is a cache line and how big is it? When I load one word from memory with a lw instruction, how much data actually gets loaded into cache?" *Why I asked this:* I wanted a concrete number to understand how much data gets pulled in on a single miss.

---

### Section 2 — Your Synthesis (30 pts)

In your own words (not copied from the AI), write an explanation covering both threads. Aim for roughly **one page** split across three parts:

**Part A — The Cache Hierarchy** What is a CPU cache, why does it exist, and why are there multiple levels (L1, L2, L3) rather than just one? Describe what happens at a hardware level when a lw instruction executes and the requested value is not in cache. Include **real latency numbers** for at least two levels of the cache hierarchy and for DRAM — look them up or ask the AI and verify them, because these numbers are the core of the story.

**Part B — Spatial and Temporal Locality** What is a **cache line** (the unit of data that gets transferred between cache levels on a miss), and how large is it in bytes and in RISC-V words? Define **spatial locality** and **temporal locality** in your own words. Then use your Lab2 array code as a specific example: explain why the sequential access pattern you used — stepping through an array with addi s0, s0, 4 — is spatially local, and estimate how many of your lw instructions were likely cache hits after the first access to any given region of the array.

**Part C — The Connection** In a short paragraph, explain how the two threads connect. Why does the cache hierarchy only work well if programs exhibit locality? What happens to performance when a program accesses memory in a random or unpredictable order? Give an approximate sense of how large the performance difference can be between cache-friendly and cache-hostile access patterns on the same hardware.

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

Given what you now understand about cache misses and memory locality, pick **two specific features of your Lab2 RISC-V code** — or of the RISC-V load/store model more broadly — and explain how each one is related to what you investigated. How does knowing about cache behavior change how you think about the assembly code you wrote?

Your answer must be specific. References to the lw instruction, the addi s0, s0, 4 pointer advance, byte addressing, word alignment, or the load/store model are all fair game. Vague answers like "RISC-V accesses memory and cache matters" earn partial credit only.

---

### Grading Rubric

| Section | Points | What TAs Are Looking For |
| ----- | ----- | ----- |
| Investigation Log | 30 | Prompts cover **both** the cache hierarchy and locality threads; clear progression showing iteration and follow-up. Dock points if only one topic is covered or if all prompts are surface-level with no iteration. |
| Synthesis | 30 | Accurate, in-student's-own-words explanations of both threads (Parts A and B) plus a genuine connection (Part C). Requires real latency numbers and a specific Lab2 code reference. Penalize heavily if it reads like a copy-paste from the AI. |
| Critical Check | 20 | Student identified something worth questioning and followed up. Honest engagement with uncertainty is enough — no need for a dramatic "gotcha." |
| Connect It Back | 20 | Two specific references to Lab2 code or RISC-V load/store model features, each clearly linked to what was investigated. Generic "cache matters for performance" earns half credit at most. |

---

### A Note on AI Use

You are *required* to use AI for this assignment. That said, the submission you turn in should represent your understanding, not a transcript of what the AI said. If your synthesis reads like it was written by the AI, that's a problem — not because we can detect it, but because it means you didn't learn anything, which defeats the point of the exercise.

The best way to make sure your synthesis is genuinely yours: write Sections 2 and 4 **without the AI chat open**, from memory, after you've finished your investigation. Then go back and check it.

---

### Suggested Starting Prompts

You don't need to follow these — they're here if you want a launching pad.

**To start the cache hierarchy thread:**

> *"I'm a CS student learning RISC-V assembly. My professor mentioned that there's a cache between the processor and main memory, but we haven't covered how it actually works. Can you explain what a CPU cache is, why it exists, and why modern processors have multiple levels of cache (L1, L2, L3) instead of just one? Assume I understand registers and that memory is byte-addressed, but I don't know anything about cache hardware."*

**To get concrete latency numbers:**

> *"Can you give me approximate access latency numbers — in clock cycles — for L1 cache, L2 cache, L3 cache, and DRAM on a modern processor? I want to understand the actual size of the speed gap between these levels."*

**To explore cache lines and locality:**

> *"What is a cache line? When I execute a lw instruction in RISC-V assembly to load one 4-byte word, how much data actually gets loaded into cache? And what does this mean for a program that reads an array sequentially — like loading element 0, then element 1, then element 2?"*

**To connect to Lab2 and push toward the key insight:**

> *"In my assembly language lab I wrote a loop that steps through an array one word at a time, advancing a pointer by 4 bytes each iteration: lw t1, 0(s0) then addi s0, s0, 4. How does the cache interact with this kind of sequential access pattern? Why would this be faster than accessing the same array elements in a random order?"*  
   
