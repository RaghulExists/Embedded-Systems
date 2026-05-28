# Module-4: Quick Revision Answers

> Compact version — every essential fact, no filler. Designed to be readable in 20 minutes and writable in an exam.

---

## Q5. ARM Processor Exceptions and Modes (with diagram)

The ARM processor has **7 exceptions** that halt normal execution. Each forces the processor into a specific **privileged mode** so a handler can run safely.

### The 7 Exceptions and Modes they enter

| Exception | Mode | Cause |
|---|---|---|
| **Reset** | Supervisor (SVC) | Power-on / hard reset |
| **Undefined Instruction** | Undefined (UND) | Unknown instruction reaches execute stage |
| **Software Interrupt (SWI)** | Supervisor (SVC) | `SWI` instruction executed |
| **Prefetch Abort** | Abort (ABT) | Memory fault while fetching instruction |
| **Data Abort** | Abort (ABT) | Memory fault while reading/writing data |
| **IRQ** | IRQ | External peripheral asserts `nIRQ` |
| **FIQ** | FIQ | External peripheral asserts `nFIQ` |

### The 7 ARM Modes
- **User (usr)** — non-privileged, normal apps
- **System (sys)** — privileged, manually entered
- **Supervisor (svc)** — Reset, SWI
- **Abort (abt)** — Data/Prefetch Abort
- **Undefined (und)** — Undefined Instruction
- **IRQ** — IRQ exception
- **FIQ** — FIQ exception

User and System are the only modes NOT entered by an exception.

### What hardware does AUTOMATICALLY on any exception (memorise!)
1. **`spsr_<mode> ← cpsr`** (save CPSR)
2. **`lr_<mode> ← pc`** (save return address)
3. **`cpsr ← new mode`** (also disables relevant interrupts)
4. **`pc ← vector address`** (jump to handler)

> Processor **always switches to ARM state** during an exception.

### Diagram
```
                    USER MODE (non-priv)
                         |
        +--------+-------+--------+--------+--------+
        |        |       |        |        |        |
      Reset   Undef    SWI    Pref/Data  IRQ      FIQ
        |        |       |     Abort      |        |
        v        v       v        v       v        v
       SVC      UND     SVC      ABT     IRQ      FIQ
        |        |       |        |       |        |
        +--------+-------+--------+-------+--------+
                         |
              Restore cpsr ← spsr, return
                         |
                         v
                    USER MODE
```

### Step-by-step
1. Exception occurs → hardware does the 4 automatic steps.
2. CPU enters privileged mode → handler runs with full access, separate banked registers (so user state isn't trashed).
3. Handler does its work.
4. Handler returns with `SUBS pc, lr, #offset` → restores both `cpsr` and `pc` atomically.

---

## Q1. Vector Table Offsets, Exceptions, and Priorities

The **vector table** is a fixed table at address `0x00000000` (or `0xFFFF0000`) with **one instruction per exception**. The hardware jumps to the corresponding fixed address; that instruction branches to the actual handler.

### Vector Table

| Offset | Exception | Mode | I bit | F bit |
|---|---|---|---|---|
| `0x00` | Reset | SVC | 1 | 1 |
| `0x04` | Undefined | UND | 1 | unchanged |
| `0x08` | SWI | SVC | 1 | unchanged |
| `0x0C` | Prefetch Abort | ABT | 1 | unchanged |
| `0x10` | Data Abort | ABT | 1 | unchanged |
| `0x14` | *Reserved* | — | — | — |
| `0x18` | IRQ | IRQ | 1 | unchanged |
| `0x1C` | FIQ | FIQ | 1 | 1 |

Each slot is 4 bytes apart (= 1 ARM instruction). FIQ is at the end so the handler can sit *directly* at `0x1C` with no branch — saving cycles.

### Branch instructions used in the table
- **`B <address>`** — PC-relative branch (±32 MB).
- **`LDR pc, [pc, #offset]`** — load 32-bit absolute address from literal pool. Anywhere in 4 GB.
- **`LDR pc, [pc, #-0xFF0]`** — special, only with VIC PL190 controller.
- **`MOV pc, #immediate`** — limited 8-bit rotated immediate.

### Example 9.1
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

### Priorities (highest → lowest)

| Priority | Exception | I set? | F set? |
|---|---|---|---|
| 1 | **Reset** | Yes | Yes |
| 2 | **Data Abort** | Yes | No |
| 3 | **FIQ** | Yes | Yes |
| 4 | **IRQ** | Yes | No |
| 5 | **Prefetch Abort** | Yes | No |
| 6 | **SWI / Undefined** (tied) | Yes | No |

> SWI and Undef share lowest priority — they happen on the same execute stage and a single instruction can't be both, so they never collide.

### Step-by-step
1. **Why a vector table?** Hardware needs fixed jump addresses; programmer fills in the branches.
2. **Why FIQ at `0x1C`?** End of table → handler can start right there → faster.
3. **Why priorities?** Two exceptions can be pending simultaneously; CPU needs deterministic order.
4. **Memory aid:** *"Really Does Fix Issues Pretty Soon"* → Reset, Data, FIQ, IRQ, PAbt, SWI.

---

## Q2. Enabling and Disabling IRQ and FIQ Exceptions

Two bits in `cpsr` control interrupts:
- **I bit** (bit 7, mask `0x80`) — 1 = IRQ disabled
- **F bit** (bit 6, mask `0x40`) — 1 = FIQ disabled

Must be in a **privileged mode** to modify these.

### The 3-instruction sequence

#### ENABLE IRQ (clear I bit)
```assembly
MRS  r1, cpsr           ; read cpsr
BIC  r1, r1, #0x80      ; clear bit 7
MSR  cpsr_c, r1         ; write back
```

#### DISABLE IRQ (set I bit)
```assembly
MRS  r1, cpsr
ORR  r1, r1, #0x80      ; set bit 7
MSR  cpsr_c, r1
```

#### Enable/disable FIQ — same code with `0x40` instead of `0x80`.

### What each instruction does
- **MRS** — Move from special Register to Standard register (read cpsr).
- **BIC** — Bit Clear: `r1 = r1 AND NOT mask` (turns OFF bits).
- **ORR** — OR: turns ON bits.
- **MSR** — Move from Standard to special Register (write cpsr).
- **`_c` postfix** — only update the **control byte** (bits 7:0), so flags (N,Z,C,V) aren't corrupted.

### Important rule
The interrupt is enabled/disabled **only after MSR completes its execute stage**.

### Step-by-step
1. **Why 3 instructions?** ARM has no instruction that operates on `cpsr` directly. Must read → modify → write.
2. **Why BIC to enable?** Sense is inverted: bit=1 means disabled, so to enable we clear it.
3. **Why `0x80` and `0x40`?** I bit is bit 7 (= 2⁷ = 0x80). F bit is bit 6 (= 2⁶ = 0x40).
4. **Why `cpsr_c`?** Updates only control byte; protects flags in upper bytes.

---

## Q6. IRQ and FIQ Exceptions (with flow diagram)

IRQ and FIQ are the two external-peripheral interrupts. Peripheral asserts `nIRQ`/`nFIQ`; if unmasked, CPU enters exception.

### Hardware procedure (5 steps)
1. Switch to corresponding mode (IRQ or FIQ).
2. `spsr_<mode> ← cpsr`.
3. `lr_<mode> ← pc`.
4. Mask interrupts:
   - **IRQ:** sets I bit only.
   - **FIQ:** sets BOTH I and F bits.
5. `pc ← 0x18` (IRQ) or `0x1C` (FIQ).

### IRQ vs FIQ

| Feature | IRQ | FIQ |
|---|---|---|
| Vector | `0x18` | `0x1C` |
| Priority | 4th | 3rd (higher) |
| Bits masked | I only | I and F |
| Banked registers | `r13_irq, r14_irq, spsr_irq` | `r8_fiq–r14_fiq, spsr_fiq` (5 extra!) |
| Use | General peripherals | Single fast device (e.g., DMA) |
| Latency | Higher | Lower |

### Why FIQ is faster
- Banked `r8`–`r12` → handler doesn't save them → fewer cycles.
- Sits at end of vector table → handler can be placed directly at `0x1C`.

### Flow diagram
```
   Peripheral asserts nIRQ / nFIQ
              |
              v
       Is interrupt unmasked?  --NO--> ignored
              | YES
              v
       HARDWARE:
       spsr_<mode> ← cpsr
       lr_<mode>   ← pc
       cpsr → mode + masks (I, or I+F for FIQ)
       pc ← 0x18 (IRQ) or 0x1C (FIQ)
              |
              v
       Vector instruction loads pc with handler addr
              |
              v
       HANDLER:
       save context → identify source → ISR → restore
              |
              v
       Return: SUBS pc, lr, #4
       (restores cpsr from spsr AND jumps to lr-4)
              |
              v
       Resume User mode
```

### Why `SUBS pc, lr, #4`?
- `lr` was set to (interrupted instruction + 8) due to ARM's pipeline.
- We want to re-execute the next instruction → need `lr − 4`.
- `S` suffix + `pc` destination = "also restore `cpsr` from `spsr`". One instruction, both jobs.

### Step-by-step
1. Hardware saves state atomically (software couldn't do it fast enough).
2. FIQ disables both I and F because nothing should preempt the highest-priority interrupt.
3. IRQ disables only I so FIQ can still preempt — that's FIQ's whole point.
4. FIQ banks 5 extra registers → fastest path through chip.

---

## Q7. Non-Nested Interrupt Handler

The **simplest** handler. All further interrupts stay disabled until the handler returns. Only **one** interrupt at a time. Not suitable for complex systems.

### The 6 stages
1. **Disable interrupts** (hardware) — mode→IRQ, I=1, save cpsr/pc, pc→0x18.
2. **Save context** (software) — `STMFD sp!, {r0-r12, lr}`.
3. **Interrupt handler** — identify which peripheral fired.
4. **ISR** — service the device, **clear the interrupt at source**.
5. **Restore context** — `LDMFD sp!, {r0-r12, lr}`.
6. **Enable interrupts and return** — `SUBS pc, lr, #4`.

### Diagram
```
Hardware: disable interrupts, switch mode, save state, jump to vector
                          |
                          v
                    Save context
                    (push r0-r12, lr)
                          |
                          v
                    Identify source
                          |
                          v
                    Run ISR
                    (clear interrupt at source)
                          |
                          v
                    Restore context
                          |
                          v
                    SUBS pc, lr, #4
                    (restores cpsr + returns)
```

### Key property
**I bit stays set the entire time.** No interrupt — even higher priority — can preempt.

### Step-by-step
1. Hardware does the 4 auto steps.
2. Handler must save user's `r0–r12` since hardware only saved cpsr/pc.
3. **Must clear interrupt at source** before re-enabling, else it fires again instantly (infinite loop).
4. Single instruction `SUBS pc, lr, #4` does the return AND restores cpsr.
5. **Limitation:** long ISR blocks all other interrupts → unsuitable for real-time systems.

---

## Q3. Nested Interrupt Handler

Allows a **second interrupt while first is being serviced**. Re-enables interrupts before the first ISR fully completes (after source has been cleared).

### Two design goals
1. Respond to interrupts quickly.
2. Don't delay regular synchronous code.

### Risks the handler must protect against
- **Stack overflow** — repeated nesting fills the IRQ stack.
- **`r14_irq` corruption** — new IRQ overwrites first handler's return address.
- **`spsr_irq` corruption** — same problem.

### How it works
- Entry is identical to non-nested handler.
- At exit, handler tests a flag set by ISR:
  - **NO more processing** → exit normally.
  - **YES more processing** →
    - Switch from IRQ mode to **SVC or System mode**.
    - **Flatten IRQ stack** onto task stack (typically SVC stack).
    - Re-enable interrupts.
    - Continue processing.

### Why switch out of IRQ mode before re-enabling?
If you stay in IRQ mode and execute `BL`, the return address goes to `r14_irq`. A new IRQ then overwrites `r14_irq` → **return address lost → crash**. Switching to SVC means `BL` uses `r14_svc`, safe from new IRQs.

### Diagram
```
   IRQ fires
        |
        v
   Entry (same as non-nested):
   save state, save context, identify, call ISR
        |
        v
   ISR clears source, sets "more work" flag
        |
        v
   Check flag
   /         \
  NO          YES
   |           |
   v           v
 Restore    Switch IRQ → SVC mode
 Return     Flatten IRQ stack to SVC stack
            Re-enable IRQs
            Continue processing
            Restore, return
```

### Step-by-step
1. Hardware entry — identical to non-nested.
2. ISR clears source so re-enabling doesn't immediately re-fire.
3. Switch to SVC mode → safe to re-enable interrupts (BL now uses `r14_svc`).
4. Flatten IRQ stack → prevents overflow on deep nesting.
5. Re-enable I bit → new interrupts allowed.
6. **Limitation:** no priority filtering → low-priority IRQ can preempt high-priority IRQ.

---

## Q4. Reentrant Interrupt Handler

Handles **multiple interrupts filtered by priority** — needed when higher-priority interrupts must have lower latency.

### Key difference from nested
**Interrupts re-enabled VERY EARLY** in the handler, drastically reducing latency.

### Where ISRs run
**Always in SVC, System, Undefined, or Abort mode** — never IRQ mode.

### The BL/`r14_irq` corruption problem
Same issue as nested: `BL` in IRQ mode writes `r14_irq`; new IRQ overwrites it. Solution: switch to SVC before any BL.

### Two critical rules
1. **Disable interrupt at source** (in controller) **before** re-enabling I bit in cpsr — else infinite loop.
2. **Mask same-or-lower priority interrupts** in controller — only higher priorities can preempt.

### IRQ stack usage
Mostly unused. `r13_irq` points to a small **12-byte structure** for temporarily storing a few registers on entry. Real ISR work happens on SVC stack.

### Why prioritisation is mandatory
Without it, a low-priority interrupt can preempt a high-priority one → latency degrades to nested level → defeats the purpose.

### Flow
```
   IRQ fires → hardware entry
              |
              v
   Save a few regs to 12-byte structure (IRQ stack)
              |
              v
   Identify source, mask same-or-lower priorities in controller
              |
              v
   Switch IRQ → SVC mode, move context to task stack
              |
              v
   RE-ENABLE IRQ (clear I bit) — early!
   Only higher-priority can now preempt
              |
              v
   Run full ISR (long, can use BL safely)
              |
              v
   Disable IRQ, re-enable source mask
              |
              v
   Restore context, switch back to IRQ mode briefly
              |
              v
   SUBS pc, lr, #4 → return
```

### Step-by-step
1. Hardware entry — same as always.
2. 12-byte temp structure on IRQ stack stores ~3 registers briefly.
3. **Disable source FIRST**, then mask lower priorities.
4. Switch to SVC mode → BL now safely uses `r14_svc`.
5. Re-enable IRQ early → only higher priorities preempt.
6. Run long ISR safely.
7. Tear down in reverse.

### Comparison Table (very useful for exam)

| Feature | Non-nested | Nested | Reentrant |
|---|---|---|---|
| Multiple interrupts? | No | Yes | Yes |
| Priority filtering? | No | No | **Yes** |
| When re-enabled? | Only at exit | Late | **Early** |
| ISR runs in | IRQ mode | IRQ→SVC late | SVC mostly |
| High-prio latency | Worst | Medium | **Best** |
| Complexity | Low | Medium | High |

---

## Exam Survival Cheat-Sheet

### Magic numbers to memorise
- I bit = `0x80` (bit 7), F bit = `0x40` (bit 6)
- Vector offsets: `0x00` Reset, `0x04` Undef, `0x08` SWI, `0x0C` PAbt, `0x10` DAbt, `0x18` IRQ, `0x1C` FIQ
- Return: `SUBS pc, lr, #4`

### The 4 hardware actions on every exception (the most asked fact!)
1. `spsr_<mode> ← cpsr`
2. `lr_<mode> ← pc`
3. `cpsr ← new mode + masks`
4. `pc ← vector`

### Priority order mnemonic
**"Really Does Fix Issues Pretty Soon"** → Reset, Data Abort, FIQ, IRQ, PAbt, SWI/Undef

### One-line distinctions
- **IRQ vs FIQ:** FIQ banks `r8–r12`, sits at `0x1C`, masks both I and F.
- **Non-nested vs Nested:** Nested re-enables interrupts before exit.
- **Nested vs Reentrant:** Reentrant filters by priority and runs ISRs in SVC mode.

You've got this!
