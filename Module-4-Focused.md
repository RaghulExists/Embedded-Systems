# Module-4: Focused Answers

> Each answer covers exactly what the question asks — nothing more, nothing less.

---

## Q1. Explain Vector Table Offset for Exceptions and Modes with example. Give the priority levels for various exceptions.

### Vector Table

The vector table is a fixed table in memory containing one instruction per exception. When an exception occurs, the ARM core jumps to the corresponding fixed address.

| Offset | Exception | Mode entered |
|---|---|---|
| `0x00` | Reset | Supervisor (SVC) |
| `0x04` | Undefined Instruction | Undefined (UND) |
| `0x08` | Software Interrupt (SWI) | Supervisor (SVC) |
| `0x0C` | Prefetch Abort | Abort (ABT) |
| `0x10` | Data Abort | Abort (ABT) |
| `0x14` | Reserved | — |
| `0x18` | IRQ | IRQ |
| `0x1C` | FIQ | FIQ |

### Example

```assembly
0x00:  LDR pc, reset_addr      ; Reset
0x04:  B   undef_handler       ; Undefined (direct branch)
0x08:  LDR pc, swi_addr        ; SWI
0x0C:  LDR pc, pabt_addr       ; Prefetch Abort
0x10:  LDR pc, dabt_addr       ; Data Abort
0x14:  NOP                     ; Reserved
0x18:  LDR pc, irq_addr        ; IRQ
0x1C:  LDR pc, fiq_addr        ; FIQ
```

The Undefined entry uses a direct PC-relative branch (`B`). Other entries use `LDR pc, [pc, #offset]` to load the handler address from a literal pool, allowing the handler to be placed anywhere in memory.

### Exception Priority Levels

| Priority | Exception | I bit set | F bit set |
|---|---|---|---|
| 1 (highest) | Reset | Yes | Yes |
| 2 | Data Abort | Yes | No |
| 3 | FIQ | Yes | Yes |
| 4 | IRQ | Yes | No |
| 5 | Prefetch Abort | Yes | No |
| 6 (lowest) | SWI / Undefined Instruction | Yes | No |

SWI and Undefined Instruction share the lowest priority because they cannot occur on the same instruction simultaneously.

### Step-by-step

1. **Vector table** is at fixed address `0x00000000` (or `0xFFFF0000`); each slot is 4 bytes (one ARM instruction).
2. When exception fires, hardware sets `pc` to the slot address. The instruction there branches to the actual handler.
3. **FIQ at `0x1C`** (last entry) lets the FIQ handler be placed directly there with no branch — saving cycles.
4. **Priorities** decide which exception is serviced first when two occur simultaneously. Reset always wins; SWI/Undef are lowest.

---

## Q2. Discuss the enabling and disabling of IRQ and FIQ exceptions.

IRQ and FIQ are controlled by two bits in the `cpsr`:
- **I bit** (bit 7, mask `0x80`) — 1 = IRQ disabled
- **F bit** (bit 6, mask `0x40`) — 1 = FIQ disabled

Modification requires a **privileged mode** and uses a 3-instruction sequence (read-modify-write).

### Enabling IRQ (clear I bit)
```assembly
MRS  r1, cpsr           ; copy cpsr into r1
BIC  r1, r1, #0x80      ; clear bit 7
MSR  cpsr_c, r1         ; write back
```

### Disabling IRQ (set I bit)
```assembly
MRS  r1, cpsr
ORR  r1, r1, #0x80      ; set bit 7
MSR  cpsr_c, r1
```

For FIQ, replace `0x80` with `0x40`.

### Step-by-step

1. **MRS** copies `cpsr` into a general register (cpsr can't be modified directly).
2. **BIC** clears bits (to enable, since the bit being clear means enabled). **ORR** sets bits (to disable).
3. **MSR cpsr_c** writes back only the control byte (`_c` = bits 7:0), protecting the flag bits in upper bytes.
4. The change takes effect only after MSR completes the execute stage of the pipeline.

---

## Q3. Explain Nested Interrupt Handler in Embedded systems.

A **nested interrupt handler** allows a second interrupt to be taken while the first is still being serviced. This is achieved by re-enabling interrupts before the handler has fully serviced the current interrupt.

### Goals
1. Respond to interrupts quickly without forcing them to wait.
2. Avoid delaying regular synchronous code while servicing interrupts.

### Risks the handler must protect against
- **Stack overflow** from deeply nested interrupts.
- **`r14_irq` corruption** — a new IRQ overwrites the first handler's return address.
- **`spsr_irq` corruption** — same issue.

### How it works
The entry code is identical to a non-nested handler. The difference is at exit: the handler tests a flag set by the ISR.
- If **no further processing needed** → exit normally.
- If **further processing needed**:
  - Switch from IRQ mode to SVC or System mode.
  - Flatten the IRQ stack onto the task stack.
  - Re-enable interrupts.
  - Continue processing.

### Why switch out of IRQ mode before re-enabling
If interrupts are re-enabled while still in IRQ mode and a `BL` is executed, the return address is stored in `r14_irq`. A new IRQ would overwrite `r14_irq`, destroying the return address. Switching to SVC mode first means `BL` uses `r14_svc`, which is safe.

### Step-by-step

1. Hardware does standard exception entry (saves cpsr/pc, masks IRQ, jumps to vector).
2. Handler saves context, identifies source, calls ISR.
3. ISR clears the source and sets a flag if more processing is required.
4. Handler checks the flag.
5. If yes: switch to SVC mode, transfer IRQ stack contents onto SVC stack, re-enable IRQs, continue.
6. Once done, restore context and return.

---

## Q4. Explain Reentrant Interrupt Handler scheme in Embedded systems.

A **reentrant interrupt handler** handles multiple interrupts filtered by **priority**, ensuring higher-priority interrupts have lower latency. It differs from a nested handler in that **interrupts are re-enabled early** in the handler.

### Key rules
- All ISRs must be serviced in **SVC, System, Undefined, or Abort mode** — not IRQ mode.
- The interrupt source must be **disabled at the controller** before re-enabling interrupts in cpsr (otherwise infinite re-entry occurs).
- Same-or-lower priority interrupts must be **masked in the controller** so only higher-priority ones can preempt.

### Why not run ISRs in IRQ mode
If a `BL` is executed in IRQ mode after re-enabling interrupts, the return address goes to `r14_irq`. A new IRQ would overwrite it, destroying the return address. Switching to SVC means `BL` uses `r14_svc`, safe from new IRQs.

### IRQ stack usage
Interrupt servicing happens on the task's SVC stack. The IRQ stack register `r13_irq` only points to a small 12-byte structure used to temporarily store registers on interrupt entry.

### Importance of prioritisation
If interrupts are not prioritised, lower-priority interrupts can preempt higher-priority ones. The system latency would degrade to that of a nested handler, defeating the purpose.

### Step-by-step

1. Hardware does standard exception entry.
2. Handler stores a few registers in the 12-byte structure on the IRQ stack.
3. Identifies the source; disables it and masks lower priorities in the controller.
4. Switches from IRQ to SVC mode; transfers context to the SVC stack.
5. Re-enables IRQs early — only higher-priority interrupts can now preempt.
6. Runs the ISR. Higher-priority interrupts can preempt safely.
7. On completion, disables IRQs, re-enables source mask, restores context, and returns.

---

## Q5. Discuss the ARM processor exceptions and modes with the help of a diagram.

The ARM processor has **7 exceptions** that halt normal execution. Each exception causes the core to enter a **specific privileged mode**. ARM has **7 modes** in total — User and System are the only modes not entered by an exception.

### Exceptions and Modes

| Exception | Mode entered |
|---|---|
| Reset | Supervisor (SVC) |
| Undefined Instruction | Undefined (UND) |
| Software Interrupt (SWI) | Supervisor (SVC) |
| Prefetch Abort | Abort (ABT) |
| Data Abort | Abort (ABT) |
| IRQ | IRQ |
| FIQ | FIQ |

### What hardware does on entry to any exception
1. Saves `cpsr` to `spsr` of the new mode.
2. Saves `pc` to `lr` of the new mode.
3. Sets `cpsr` to the new mode.
4. Sets `pc` to the address of the exception handler.

The processor always switches to ARM state when an exception occurs.

### Diagram

```
                    USER MODE (non-privileged)
                           |
      +--------+--------+--+----+--------+--------+
      |        |        |       |        |        |
    Reset    Undef     SWI    Abort     IRQ      FIQ
      |        |        |     (Pref/    |        |
      v        v        v      Data)    v        v
     SVC      UND      SVC      |      IRQ      FIQ
                                v       
                               ABT       
                                |
                                v
              Handler restores cpsr ← spsr
              and returns to User Mode
```

### Step-by-step

1. User-mode code runs normally (non-privileged).
2. An exception occurs → hardware performs the 4 automatic steps.
3. CPU enters the corresponding privileged mode with separate banked `lr` and `sp` (FIQ also banks `r8`–`r12`), so the handler doesn't disturb user state.
4. Handler runs, then returns using an instruction like `SUBS pc, lr, #4`, which restores `cpsr` from `spsr` and returns to User mode.

---

## Q6. Explain IRQ and FIQ exceptions in ARM Processor with Flow diagram.

IRQ (Interrupt Request) and FIQ (Fast Interrupt Request) are external-peripheral interrupts. A peripheral asserts the `nIRQ` or `nFIQ` pin; if unmasked, the CPU enters the corresponding exception.

### Hardware procedure (on entry)
1. Processor changes to the corresponding interrupt mode.
2. `spsr_<mode> ← cpsr`.
3. `lr_<mode> ← pc`.
4. Mask interrupts:
   - **IRQ**: sets I bit only.
   - **FIQ**: sets both I and F bits.
5. `pc ← 0x18` (IRQ) or `0x1C` (FIQ).

### Differences

| Feature | IRQ | FIQ |
|---|---|---|
| Vector | `0x18` | `0x1C` (last entry) |
| Priority | 4th | 3rd (higher) |
| Masks on entry | I only | I and F |
| Banked registers | `r13_irq`, `r14_irq`, `spsr_irq` | `r8_fiq`–`r14_fiq`, `spsr_fiq` |
| Use case | General peripherals | Single fast device (e.g., DMA) |

FIQ is faster because it banks 5 extra registers (handler doesn't save them) and sits at the end of the vector table (handler can be placed directly at `0x1C`).

### Flow Diagram

```
   Peripheral asserts nIRQ or nFIQ
              |
              v
       Is interrupt unmasked? --NO--> ignored
              | YES
              v
   HARDWARE:
   spsr_<mode> ← cpsr
   lr_<mode>   ← pc
   cpsr → mode bits set, mask interrupts
          (IRQ: set I; FIQ: set I and F)
   pc → 0x18 (IRQ) or 0x1C (FIQ)
              |
              v
   Vector instruction loads pc with handler address
              |
              v
   HANDLER:
   - save context
   - identify source
   - run ISR (clear source)
   - restore context
              |
              v
   Return: SUBS pc, lr, #4
   (restores cpsr ← spsr and pc ← lr-4)
              |
              v
       Resume User-mode program
```

### Step-by-step

1. Peripheral asserts the pin → CPU checks the corresponding mask bit.
2. If unmasked, hardware does the 5-step entry automatically.
3. Vector instruction at `0x18` or `0x1C` jumps to the handler.
4. Handler saves context, services the source, restores context.
5. `SUBS pc, lr, #4` returns to user code (subtract 4 to compensate for the pipeline offset; `S` suffix restores `cpsr` from `spsr`).

---

## Q7. Explain Non-Nested Interrupt Handler in Embedded systems.

A **non-nested interrupt handler** is the simplest scheme: interrupts are disabled until control returns to the interrupted task. Only one interrupt is serviced at a time. Not suitable for systems needing multiple priority levels.

### The 6 stages

1. **Disable interrupts** (hardware) — IRQ exception sets I bit, switches mode, copies cpsr→spsr_irq, pc→lr_irq, jumps to vector `0x18`.
2. **Save context** (software) — handler pushes non-banked registers onto the IRQ stack: `STMFD sp!, {r0-r12, lr}`.
3. **Interrupt handler** — identifies which peripheral fired.
4. **ISR** — services the source and clears the interrupt at the source.
5. **Restore context** — `LDMFD sp!, {r0-r12, lr}`.
6. **Enable interrupts and return** — `SUBS pc, lr, #4` restores `cpsr` from `spsr_irq` and returns.

### Diagram

```
   IRQ fires
        |
        v
   1. Hardware: disable IRQ, save cpsr/pc, jump to 0x18
        |
        v
   2. Save context (push r0-r12, lr)
        |
        v
   3. Identify source
        |
        v
   4. Run ISR (clear source)
        |
        v
   5. Restore context (pop r0-r12, lr)
        |
        v
   6. SUBS pc, lr, #4 → return + restore cpsr
```

### Key property
The I bit stays set the entire time the handler runs. **No interrupt — even higher priority — can preempt it.**

### Step-by-step

1. Hardware automatically performs the 4 entry actions and disables further IRQs.
2. Handler saves user registers because hardware only saved cpsr/pc.
3. Handler identifies which device fired by reading the interrupt controller.
4. ISR services the device and clears its interrupt latch (otherwise it would re-fire immediately on exit).
5. Handler restores registers.
6. `SUBS pc, lr, #4` simultaneously returns and restores cpsr (re-enabling interrupts via the saved spsr).

---

## Prompt for other modules

To get the same focused-answer style for other modules, use this prompt:

> I have a test tomorrow on **Module-X** and don't know much about the topic. I have these questions and want you to answer them **strictly based on what each question asks** — no extra content, no filler, no unnecessary background. Read each question carefully and only include what is directly relevant.
>
> For each answer:
> - Use the notes in my repo (`Module-X.md`) as the primary source.
> - If something isn't in my notes but is essential to answer the question correctly, you may use your knowledge — but say so honestly.
> - End each answer with a short **step-by-step** explanation of the key reasoning.
> - Keep answers compact but complete enough to write in an exam.
> - Use tables and diagrams where they help clarity, but don't add them just for show.
> - Do **not** add comparison tables, mnemonics, or "exam tips" unless I explicitly ask.
> - If you are not 100% sure of an answer, say so honestly rather than guessing.
>
> Save the result to a new file: `Module-X-Focused.md` and push it to a new branch on GitHub.
>
> Here are the questions:
> [paste your questions]
