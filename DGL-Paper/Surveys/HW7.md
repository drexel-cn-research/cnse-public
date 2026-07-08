# HW7 Survey — Block 1: Learning Effectiveness

**Topic:** When Programs Go Wrong: CPU Exceptions and Hardware Fault Handling (CPU exceptions & privilege modes)

These three questions were unique to the HW7 survey. The remaining standardized questions (Blocks 2–5) are shown once in [`common-questions.md`](common-questions.md).

---

**1. Before this assignment, how well did you understand how a CPU handles exceptions, and why hardware enforces a privilege boundary between user programs and the operating system?**

1. No idea on either — I assumed the CPU just crashed, and I'd never thought about privilege levels
2. Vaguely aware something special happens during failures, but had no idea about privilege modes
3. Knew there was a handler involved for exceptions, or had heard of privilege modes, but couldn't describe either mechanism
4. Could describe the general concept of exception handling or privilege modes, but not both
5. Understood both the hardware exception mechanism and privilege modes well before this assignment

**2. After completing this assignment, how well do you understand the RISC-V exception mechanism — what the hardware does when an exception occurs, the role of mepc/mcause/mtvec, and why privilege modes matter?**

1. Still don't understand the mechanism
2. I have a vague sense of the ideas but couldn't explain them
3. I understand the hardware steps individually but can't connect them to privilege modes
4. I could explain both the exception sequence and the privilege boundary to someone else
5. I would feel comfortable discussing RISC-V exception handling in a technical interview

**3. How clearly can you now connect what you investigated (exceptions, CSRs, privilege modes) to the ecall instruction from HW3 and the single-cycle CPU you built in Lab 7?**

1. Neither HW3 nor Lab 7 connected to this assignment at all
2. There was a vague connection but I'm not sure how the pieces fit
3. I see a general connection but I'm uncertain about the hardware details
4. The earlier work gave me concrete context and helped me understand what was missing from my CPU
5. There was a clear and specific connection between what I built/used before and what I investigated here
