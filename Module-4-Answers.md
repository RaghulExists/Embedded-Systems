# Module-4: Exception and Interrupt Handling — Exam Answers

> Source: Based on your `Module-4.md` notes (textbook: *ARM System Developer's Guide* by Sloss, Symes, Wright — Chapter 9). Every answer is followed by a step-by-step explanation so you understand the "why".

---

## Quick "core idea" you must internalise before reading

The whole module is about **what happens when something interrupts the normal flow of a program** on an ARM processor.

Three things happen in sequence whenever any exception occurs:

1. **Mode switch** — the CPU jumps from whatever mode it was in (usually User mode) into a special *exception mode* (IRQ mode, FIQ mode, Supervisor mode, etc.).
2. **State save** — the CPU automatically saves where it was (`pc → lr`) and how it was feeling (`cpsr → spsr`).
3. **Vector jump** — the CPU jumps to a fixed address in the **vector table**, where the programmer has placed a branch instruction to the appropriate **handler**.

When the handler finishes, it does the reverse: restores `cpsr` from `spsr`, and `pc` from `lr` (with a small offset). That's it. Every question below is just a deeper look at one of these three steps.

---

# Question 5 — ARM Processor Exceptions and Modes (with diagram)

### Answer

The ARM processor has **seven exceptions** that can halt the normal sequential execution of instructions. Each exception, when it occurs, forces the processor into a **specific privileged mode** so that a handler can deal with it safely without disturbing the user program's state.

### The seven ARM exceptions and the modes they enter

| Exception | Mode entered | Cause |
|---|---|---|
| **Reset** | Supervisor (SVC) | Power-on or hard reset of the processor |
| **Undefined Instruction** | Undefined (UND) | An instruction not in the ARM/Thumb set reaches the execute stage and no coprocessor claims it |
| **Software Interrupt (SWI)** | Supervisor (SVC) | The `SWI` instruction is executed (used by user code to request OS services) |
| **Prefetch Abort** | Abort (ABT) | Memory fault while *fetching* an instruction |
| **Data Abort** | Abort (ABT) | Memory fault while *reading/writing* data (bad address or no permission) |
| **IRQ (Interrupt Request)** | IRQ | External peripheral asserts the `nIRQ` pin |
| **FIQ (Fast Interrupt Request)** | FIQ | External peripheral asserts the `nFIQ` pin (for high-speed, low-latency devices) |

### The seven ARM processor modes

ARM has **7 modes** in total. Six of them are entered automatically by exceptions; two cannot be entered by an exception (User and System) — to enter those you must directly modify the `cpsr`.

| Mode | Type | How you enter it |
|---|---|---|
| **User (usr)** | Non-privileged | Normal application code runs here |
| **System (sys)** | Privileged | Manually, by writing to `cpsr` (shares User registers but is privileged) |
| **Supervisor (svc)** | Privileged | On Reset or SWI |
| **Abort (abt)** | Privileged | On Data Abort or Prefetch Abort |
| **Undefined (und)** | Privileged | On Undefined Instruction exception |
| **IRQ (irq)** | Privileged | On IRQ |
| **FIQ (fiq)** | Privileged | On FIQ |

### What the processor does AUTOMATICALLY on entry to any exception (memorise this — it's exam gold)

When an exception causes a mode change, the core automatically:

1. **Saves the `cpsr` to the `spsr` of the new exception mode** → `spsr_<mode> = cpsr`
2. **Saves the `pc` to the `lr` of the new exception mode** → `lr_<mode> = pc`
3. **Sets the `cpsr` to the new exception mode** (this also disables certain interrupts — see Q2)
4. **Sets `pc` to the address of the exception handler** (the corresponding vector-table entry)

> Important: when an exception occurs, the ARM processor **always switches to ARM state** (even if it was running Thumb code), because the vector table contains 32-bit ARM instructions.

### Diagram (text version — draw this in your exam)

```
                        +------------------+
                        |   USER MODE      |  <-- normal app code
                        |   (non-priv)     |
                        +---------+--------+
                                  |
       Exception occurs ----------+----------- (also: write to cpsr -> System mode)
                                  |
       +------+------+------+-----+-----+-----+------+
       |      |      |      |          |     |      |
     Reset  Undef   SWI   Pref-     Data    IRQ    FIQ
       |      |      |    Abort     Abort    |      |
       v      v      v      v         v      v      v
     +---+ +---+ +---+   +-----+   +-----+ +---+ +---+
     |SVC| |UND| |SVC|   | ABT |   | ABT | |IRQ| |FIQ|
     +---+ +---+ +---+   +-----+   +-----+ +---+ +---+
       \      \      \      \          /     /     /
        \      \      \   Each handler does its work,    /
         \      \      \  then restores cpsr <- spsr   /
          \      \      \  and pc <- lr (with offset)/
           +------+------+----------+--------+-----+
                                  |
                                  v
                          back to USER mode
```

(In a real diagram you'd draw a box at the top labelled "User Mode", arrows fanning out to 6 boxes below labelled with the 6 privileged modes, and a return arrow back to User mode.)

### Step-by-step explanation

**Step 1 — Why we need modes at all.**
A user program (running in User mode) cannot be trusted to safely handle a hardware fault, a timer interrupt, or a memory error. So the ARM designers gave the chip *separate privileged modes* — one for each kind of disaster. Each mode has its own *banked* `lr` and `sp` registers (and FIQ even has its own `r8–r12`), so the handler can do its job without trampling on the user program's registers.

**Step 2 — Why exactly 7 modes?**
- 1 for normal use (User)
- 1 privileged sibling of User (System) — useful for OS code that needs full access but doesn't want a separate stack
- 1 each for the 5 things that can go wrong (Reset/SWI share SVC; Data/Prefetch Abort share ABT; Undef gets its own; IRQ gets its own; FIQ gets its own).
- That gives us 7.

**Step 3 — The 4 automatic actions.**
Memorise the order: **CPSR → SPSR, PC → LR, set new CPSR, set new PC**. The CPU does these in hardware in 1–2 cycles. The handler programmer doesn't need to write code for these — they just happen.

**Step 4 — Returning from the exception.**
The handler must reverse what the hardware did:
- Restore `cpsr` from `spsr_<mode>`
- Restore `pc` from `lr_<mode>` (usually with an offset like `−4` or `−8` — see Q1's link-register offset section).

This is typically done with a single instruction like `SUBS pc, lr, #4` (the `S` suffix on `SUB` plus `pc` as destination tells the CPU "also copy `spsr` back into `cpsr`").

---

# Question 1 — Vector Table Offset for Exceptions and Modes (with example + priorities)

### Answer

The **vector table** is a small fixed table in memory (starting at address `0x00000000`, or alternatively `0xFFFF0000` on some systems) that contains **one instruction per exception**. When an exception fires, the ARM core jumps to the corresponding fixed address in this table; the instruction stored there branches to the actual handler.

### The Vector Table — addresses, exceptions, and modes

| Vector Offset | Exception | Mode entered on entry | I bit | F bit |
|---|---|---|---|---|
| `0x00` | Reset | Supervisor (SVC) | 1 (disabled) | 1 (disabled) |
| `0x04` | Undefined Instruction | Undefined (UND) | 1 | unchanged |
| `0x08` | Software Interrupt (SWI) | Supervisor (SVC) | 1 | unchanged |
| `0x0C` | Prefetch Abort | Abort (ABT) | 1 | unchanged |
| `0x10` | Data Abort | Abort (ABT) | 1 | unchanged |
| `0x14` | *Reserved* | — | — | — |
| `0x18` | IRQ | IRQ mode | 1 | unchanged |
| `0x1C` | FIQ | FIQ mode | 1 (disabled) | 1 (disabled) |

> Note that the offsets are **4 bytes apart** (one ARM instruction = 4 bytes). The slot at `0x14` is reserved/unused. FIQ deliberately sits at the **end** (`0x1C`) so that the FIQ handler code can be placed *directly* at that address with no branch needed (saving cycles → faster response).

### The four kinds of branch instructions used in the vector table

The instruction stored at each vector slot is usually one of:

1. **`B <address>`** — a PC-relative branch. Range is ±32 MB. Simple but limited.
2. **`LDR pc, [pc, #offset]`** — loads a 32-bit absolute handler address from a literal pool just after the vector table. Slightly slower (extra memory read) but can branch *anywhere* in the 4 GB address space.
3. **`LDR pc, [pc, #-0xFF0]`** — special form used **only with the VIC PL190 vector interrupt controller**: it directly reads the handler address from address `0xFFFFF030` inside the VIC. Useful for hardware-prioritised IRQs.
4. **`MOV pc, #immediate`** — copies an 8-bit immediate (rotated by an even number of bits) directly into `pc`. Limited alignment but no extra memory read.

### Example 9.1 — A typical vector table (from your notes)

```assembly
        ; Address     Instruction                Comment
0x00:   LDR pc, reset_addr        ; Reset       -> jumps via literal pool
0x04:   B undef_handler           ; Undefined   -> direct PC-relative branch
0x08:   LDR pc, swi_addr          ; SWI
0x0C:   LDR pc, pabt_addr         ; Prefetch Abort
0x10:   LDR pc, dabt_addr         ; Data Abort
0x14:   NOP                       ; Reserved slot
0x18:   LDR pc, irq_addr          ; IRQ
0x1C:   LDR pc, fiq_addr          ; FIQ  (could also place handler code here directly)

; --- literal pool (just after the vector table) ---
reset_addr: .word reset_handler
swi_addr:   .word swi_handler
pabt_addr:  .word pabt_handler
dabt_addr:  .word dabt_handler
irq_addr:   .word irq_handler
fiq_addr:   .word fiq_handler
```

The Undefined Instruction entry uses a direct `B` branch. All other vectors use `LDR pc, [pc, #offset]` to jump to handlers anywhere in memory. **FIQ also uses an `LDR` here**, but a smarter design would place the FIQ handler code directly at `0x1C` to save cycles.

### Exception Priorities (Table 9.3)

When two exceptions occur at the same time, the processor uses this **priority order** to decide which to service first:

| Priority | Exception | I bit set on entry? | F bit set on entry? |
|---|---|---|---|
| **1 (highest)** | Reset | Yes (1) | Yes (1) |
| **2** | Data Abort | Yes (1) | No (unchanged) |
| **3** | FIQ | Yes (1) | Yes (1) |
| **4** | IRQ | Yes (1) | No (unchanged) |
| **5** | Prefetch Abort | Yes (1) | No (unchanged) |
| **6 (lowest)** | SWI / Undefined Instruction (tied) | Yes (1) | No (unchanged) |

Why SWI and Undefined share the lowest priority: they both happen at the **execute** stage of *one* specific instruction. A single instruction cannot be *both* an SWI *and* undefined, so they can never collide — no need to prioritise between them.

### Step-by-step explanation

**Step 1 — Why a vector table?**
The CPU is hardware. When an exception happens, the hardware needs a *fixed, known place* to jump to. Hard-coding the actual handler address into the chip would be inflexible (different OSes have different handlers). Instead, the hardware always jumps to **the same 8 fixed addresses (`0x00`–`0x1C`)**, and the *programmer* fills those slots with branch instructions to wherever the handler actually lives. Best of both worlds.

**Step 2 — Why each offset is 4 bytes apart.**
Every ARM instruction is exactly 4 bytes (32 bits). One instruction per vector slot = 4-byte spacing.

**Step 3 — Why FIQ is at `0x1C` (the very end).**
Because there's no slot after `0x1C` (within the table), the FIQ handler can simply *start* at `0x1C` and run downward into the rest of memory. This avoids the cost of a branch instruction, shaving cycles off the FIQ response time. FIQ is meant to be **fast**, hence the placement.

**Step 4 — Why we need priorities.**
Two exceptions can become pending simultaneously (e.g., a Data Abort *and* an IRQ on the same cycle). The CPU needs a deterministic rule to pick one. The rule is just the priority table above.

**Step 5 — How to remember the priority order.**
- *Reset* must always win — if power is being applied, nothing else matters.
- *Data Abort* is next because if a memory access fails, the CPU is in an inconsistent state and must resolve that before servicing anything else.
- *FIQ > IRQ* because FIQ is the "fast" interrupt — by definition higher priority.
- *Prefetch Abort* below IRQ because it's "less urgent" — the bad instruction won't actually do harm until executed; an IRQ might be a real-time deadline.
- *SWI/Undef* last — they're synchronous, deliberate, and never time-critical.

**Step 6 — The "I bit / F bit" columns.**
The `I` and `F` bits in the `cpsr` are the **disable flags** for IRQ and FIQ respectively (1 = disabled, 0 = enabled). When you enter the FIQ or Reset handler, *both* are auto-disabled (so a higher-speed interrupt can't preempt you while you're saving state). For other exceptions, only the I bit gets set, so FIQ can still preempt them — useful, since FIQ is the highest-speed source.

---

# Question 2 — Enabling and Disabling of IRQ and FIQ Exceptions

### Answer

The ARM processor has two single-bit flags inside the **`cpsr`** that control whether IRQ and FIQ are accepted:

- **I bit** (bit 7 of `cpsr`) — when **1**, IRQ is **disabled** (masked).
- **F bit** (bit 6 of `cpsr`) — when **1**, FIQ is **disabled** (masked).

To change these bits, the processor must be in a **privileged mode** (User mode cannot modify the `cpsr` control field).

### The 3-instruction enable/disable sequence

Both enabling and disabling use the same idea: read `cpsr`, modify the bit, write `cpsr` back. Three instructions are needed:

#### To ENABLE IRQ (clear the I bit)

```assembly
        MRS   r1, cpsr        ; r1  <-  cpsr      (read CPSR)
        BIC   r1, r1, #0x80   ; r1  <-  r1 AND NOT 0x80   (clear bit 7 = I bit)
        MSR   cpsr_c, r1      ; cpsr_c <- r1      (write back the control byte)
```

#### To ENABLE FIQ (clear the F bit)

```assembly
        MRS   r1, cpsr
        BIC   r1, r1, #0x40   ; clear bit 6 = F bit
        MSR   cpsr_c, r1
```

#### To DISABLE IRQ (set the I bit)

```assembly
        MRS   r1, cpsr
        ORR   r1, r1, #0x80   ; set bit 7 = I bit
        MSR   cpsr_c, r1
```

#### To DISABLE FIQ (set the F bit)

```assembly
        MRS   r1, cpsr
        ORR   r1, r1, #0x40   ; set bit 6 = F bit
        MSR   cpsr_c, r1
```

### What each instruction does

| Instruction | Meaning |
|---|---|
| **`MRS r1, cpsr`** | **M**ove from special **R**egister to **S**tandard register: copies `cpsr` into `r1` so we can edit it. |
| **`BIC r1, r1, #mask`** | **B**it **C**lear: `r1 = r1 AND (NOT mask)` — turns OFF the bits in the mask. |
| **`ORR r1, r1, #mask`** | OR: `r1 = r1 OR mask` — turns ON the bits in the mask. |
| **`MSR cpsr_c, r1`** | **M**ove from **S**tandard register to special **R**egister: writes `r1` back into the **`_c`** (control) byte of `cpsr`. |

### What the `_c` postfix means

`cpsr` is 32 bits, divided into 4 named **fields**:

```
  31........24 23........16 15........8 7........0
  +-----------+------------+-----------+----------+
  |   FLG     |   STATUS   |   EXTN    |   CTL    |
  +-----------+------------+-----------+----------+
       _f          _s          _x         _c
```

Bits 7:0 = the **control field** = **`_c`**. The I and F bits live here (bits 7 and 6). So we use `MSR cpsr_c, r1` — telling the CPU "only update the control byte; leave flags etc. untouched".

### Important timing rule (Sloss textbook)

> The interrupt request is enabled or disabled **only once the `MSR` instruction has completed the execute stage of the pipeline**. Interrupts can still be raised or masked *before* the MSR completes.

Translation: don't assume the change is instant. The bit takes effect when MSR retires — usually 1 cycle later.

### Bit values (memorise these)

| Bit | Position | Mask (hex) | Mask (binary) |
|---|---|---|---|
| **F** | bit 6 | `0x40` | `0100 0000` |
| **I** | bit 7 | `0x80` | `1000 0000` |
| Both | bits 6,7 | `0xC0` | `1100 0000` |

### Step-by-step explanation

**Step 1 — Why we can't just write 1 instruction.**
The `cpsr` is a special "system" register, not a regular register like `r0`–`r12`. ARM does not have an instruction like "AND cpsr, cpsr, #~0x80" that operates *directly* on `cpsr`. The only ways to touch `cpsr` are `MRS` (read) and `MSR` (write). So we need: read into a normal register → modify → write back. That's 3 instructions.

**Step 2 — Why `BIC` to enable, `ORR` to disable.**
Remember the sense is *inverted*: bit = 1 means **disabled**.
- To enable, we need to clear the bit → use `BIC` (bit clear).
- To disable, we need to set the bit → use `ORR`.

**Step 3 — Why `0x80` for IRQ and `0x40` for FIQ.**
The I bit is bit 7 → `2^7 = 128 = 0x80`. The F bit is bit 6 → `2^6 = 64 = 0x40`. If you wanted to disable both at once, you'd use `0xC0` (= 0x80 + 0x40).

**Step 4 — Why `cpsr_c` and not just `cpsr`.**
The `_c` postfix tells `MSR` to update only bits 7:0 — the control field. If you wrote to the full `cpsr`, you'd risk corrupting the **N, Z, C, V** condition flags in the top byte, which would break the next conditional instruction. Always use `cpsr_c` for I/F changes.

**Step 5 — Why you must be in a privileged mode.**
User mode is not allowed to modify `cpsr_c`. If a user-mode program runs `MSR cpsr_c, r1`, the write is silently ignored. So enable/disable code lives in OS/handler code (which runs in SVC, IRQ, FIQ, etc.).

---

# Question 6 — IRQ and FIQ Exceptions in ARM Processor (with flow diagram)

### Answer

IRQ (Interrupt Request) and FIQ (Fast Interrupt Request) are the **two external-peripheral interrupts** on the ARM processor. A peripheral asserts the corresponding pin (`nIRQ` or `nFIQ`), and if that interrupt class is unmasked in the `cpsr`, the CPU enters an exception.

### Standard 5-step procedure the hardware performs (memorise — exam answer)

When an IRQ or FIQ exception fires (and is not masked), the processor hardware automatically does:

1. **Mode switch** — the processor changes to the corresponding interrupt request mode (IRQ mode or FIQ mode).
2. **Save CPSR** — the previous mode's `cpsr` is copied into the `spsr` of the new mode (`spsr_irq` or `spsr_fiq`).
3. **Save PC** — the current `pc` is saved into the `lr` of the new mode (`r14_irq` or `r14_fiq`).
4. **Mask interrupts** — for IRQ: only the `I` bit is set (disabling further IRQs); for FIQ: **both `I` and `F` bits are set** (disabling both kinds of interrupts).
5. **Vector branch** — `pc` is set to the vector-table entry (`0x18` for IRQ, `0x1C` for FIQ).

### Differences between IRQ and FIQ

| Feature | IRQ | FIQ |
|---|---|---|
| Vector address | `0x18` | `0x1C` (last entry — handler can sit here directly) |
| Priority | 4th (lower) | 3rd (higher than IRQ) |
| Bits masked on entry | `I` only | `I` *and* `F` |
| Banked registers | `r13_irq`, `r14_irq`, `spsr_irq` | `r8_fiq–r14_fiq`, `spsr_fiq` (5 extra banked registers!) |
| Typical use | General-purpose: timers, UART, etc. | One single high-speed source (e.g., DMA) |
| Latency | Higher | Lower (because of banked regs + position in vector table) |

### Example 9.5 — IRQ flow (3 states)

```
   STATE 1: Running in USER mode
   --------------------------------
       cpsr = User mode, I=0, F=0      (interrupts enabled)
       pc   = address of next user instruction
       
              | IRQ pin asserted by peripheral
              v
              
   STATE 2: Hardware enters IRQ mode (automatic)
   --------------------------------
       spsr_irq <- cpsr (saved User-mode cpsr)
       r14_irq  <- pc   (return address)
       cpsr     <- IRQ mode, I=1 (further IRQs masked)
                        F still =0 (FIQ can still preempt)
       pc       <- 0x18 (IRQ vector)
       
              | LDR pc, irq_addr at 0x18
              v
              
   STATE 3: Software IRQ handler runs
   --------------------------------
       Handler saves context (push r0-r12)
       Identifies which peripheral fired
       Calls the appropriate ISR
       Restores context
       Returns:  SUBS pc, r14, #4
       (this restores cpsr from spsr_irq AND pc from r14_irq - 4)
              |
              v
   Back to STATE 1 (User mode resumes)
```

### Example 9.6 — FIQ flow (same shape, two key differences)

```
   STATE 1: Running in USER mode
       cpsr = User mode, I=0, F=0
       pc   = ...
       
              | FIQ pin asserted
              v
              
   STATE 2: Hardware enters FIQ mode (automatic)
   --------------------------------
       spsr_fiq <- cpsr
       r14_fiq  <- pc
       cpsr     <- FIQ mode, I=1, F=1   <-- BOTH disabled!
       pc       <- 0x1C
       
              v
              
   STATE 3: FIQ handler runs (often AT 0x1C directly)
       NOTE: r8-r12 are banked in FIQ mode -- the handler
             does NOT need to save them. It can use them
             freely as scratch (e.g., for buffer pointers).
       Service the source.
       Return: SUBS pc, r14, #4
              v
   Back to STATE 1
```

### Flow diagram (text version — draw this in the exam)

```
   +----------------------+
   |  Peripheral asserts  |
   |    nIRQ or nFIQ      |
   +----------+-----------+
              |
              v
   +----------------------+        I or F bit
   |   Is the interrupt   |---NO-->  set?         (masked => ignored,
   |     unmasked in      |                       continue normal exec)
   |       cpsr ?         |
   +----------+-----------+
              | YES
              v
   +-----------------------------+
   |  Hardware:                  |
   |   1. spsr_<mode> <- cpsr    |
   |   2. lr_<mode>   <- pc      |
   |   3. cpsr -> mode bits      |
   |      - IRQ : set I          |
   |      - FIQ : set I and F    |
   |   4. pc <- 0x18 (IRQ) or    |
   |               0x1C (FIQ)    |
   +-------------+---------------+
                 |
                 v
   +-----------------------------+
   |  Vector instruction loads   |
   |  pc with handler address    |
   +-------------+---------------+
                 |
                 v
   +-----------------------------+
   |  Handler executes:          |
   |   - save context (push regs)|
   |   - identify source         |
   |   - call ISR                |
   |   - restore context         |
   +-------------+---------------+
                 |
                 v
   +-----------------------------+
   |  Return:                    |
   |   SUBS pc, lr, #4           |
   |   (restores cpsr from spsr  |
   |    AND jumps to lr-4)       |
   +-------------+---------------+
                 |
                 v
        Resume User-mode program
```

### Step-by-step explanation

**Step 1 — Why the hardware (not software) saves `cpsr` and `pc`.**
The instant an interrupt fires, the user program is mid-execution. If software had to save state, *between* "interrupt fired" and "save state" the CPU would already have lost the original `cpsr`. So the hardware does it atomically. By the time *software* (the handler) runs, the state is safely tucked into `spsr_<mode>` and `lr_<mode>`.

**Step 2 — Why FIQ disables BOTH I and F, but IRQ disables only I.**
- FIQ is the highest-priority interrupt the user can configure, so once you're servicing one, you don't want *anything* else preempting you (including another FIQ or an IRQ).
- IRQ is lower priority. While servicing an IRQ, an FIQ should still be able to break in — that's the whole point of FIQ being "fast".

**Step 3 — Why FIQ is at `0x1C`.**
It's the *last* slot in the vector table, so the FIQ handler can be placed *immediately* at `0x1C` with no branch needed → fewer cycles → lower latency.

**Step 4 — Why FIQ banks registers `r8`–`r12`.**
In FIQ mode, the registers `r8`–`r12` are physically separate from the User-mode `r8`–`r12`. So the FIQ handler can write to them *without saving them first*. Saving 5 fewer registers ≈ 10+ fewer cycles per FIQ → faster response. This is a hardware optimisation specific to FIQ.

**Step 5 — Why `SUBS pc, lr, #4` for return.**
- The `lr` was set when the interrupt fired *while* the user instruction was executing. Because of the ARM 3-stage pipeline, by the time the interrupt is taken, `pc` is already 8 bytes ahead of the instruction we want to *resume*. The hardware stored `pc` (which is "interrupted-instruction + 8") into `lr`. We want to re-execute the next instruction after the interrupted one, so we need `lr − 4`.
- The `S` suffix on `SUB` (when destination is `pc`) is special: it tells the CPU "also copy `spsr_<mode>` back into `cpsr`". One instruction does both jobs: restore mode AND restore PC.

**Step 6 — Why this matters for system design.**
- Use IRQ for general peripherals. They share the IRQ handler, which is allowed to be slightly slower.
- Reserve FIQ for the *one* device with the tightest deadline (e.g., a high-rate DMA controller). The banked registers make this device's handler **the fastest path through the chip**.

---

# Question 7 — Non-Nested Interrupt Handler

### Answer

The **non-nested interrupt handler** is the **simplest** scheme. Once an interrupt enters this handler, **all further interrupts remain disabled** until the handler returns to the interrupted task. Only **one** interrupt is serviced at a time.

This makes it unsuitable for complex systems with many interrupt sources of different priorities — but ideal as a starting point for understanding handlers.

### The 6 stages (Figure 9.8)

```
   +--------------------------+
   |  1. DISABLE INTERRUPTS   |
   |     (done by hardware)   |
   |                          |
   |  - Mode  -> IRQ mode     |
   |  - I bit -> 1            |
   |  - cpsr  -> spsr_irq     |
   |  - pc    -> lr_irq       |
   |  - pc    -> 0x18 vector  |
   +-----------+--------------+
               |
               v
   +--------------------------+
   |  2. SAVE CONTEXT         |
   |     (done by software)   |
   |                          |
   |  STMFD sp!, {r0-r12, lr} |
   |  Push the non-banked     |
   |  user regs onto IRQ stack|
   +-----------+--------------+
               |
               v
   +--------------------------+
   |  3. INTERRUPT HANDLER    |
   |  Identify which periph-  |
   |  eral fired (read the    |
   |  interrupt-status reg in |
   |  the controller)         |
   +-----------+--------------+
               |
               v
   +--------------------------+
   |  4. INTERRUPT SERVICE    |
   |     ROUTINE (ISR)        |
   |  Do the actual work for  |
   |  this device, then clear |
   |  its interrupt latch.    |
   +-----------+--------------+
               |
               v
   +--------------------------+
   |  5. RESTORE CONTEXT      |
   |  LDMFD sp!, {r0-r12, lr} |
   +-----------+--------------+
               |
               v
   +--------------------------+
   |  6. ENABLE INTERRUPTS    |
   |     and RETURN           |
   |  cpsr <- spsr_irq        |
   |  pc   <- lr_irq - 4      |
   |  (single instr:          |
   |    SUBS pc, lr, #4 )     |
   +--------------------------+
```

### Key property — interrupts are DISABLED throughout

Because step 6 (re-enable) happens *only at exit*, the I bit is set the entire time the handler is running. **No new interrupt — even a higher-priority one — can preempt this handler.** That's the whole point of "non-nested".

### Step-by-step explanation

**Step 1 — Disable interrupts (automatic).**
When the IRQ exception is raised, the ARM hardware:
- Switches mode → IRQ mode
- Sets the I bit (auto-mask of further IRQs)
- Saves `cpsr` into `spsr_irq`
- Saves `pc` into `lr_irq`
- Loads `pc` with the contents of vector `0x18`
- That instruction (e.g., `LDR pc, irq_addr`) jumps to your handler.

**Step 2 — Save context (manual).**
The hardware saved only `cpsr` and `pc`. But the handler is about to use `r0`, `r1`, etc. — those still hold the *user's* values. We must preserve them. So the first thing the handler does is push the non-banked registers onto the IRQ stack:
```
STMFD sp!, {r0-r12, lr}
```
Note: `sp` (= `r13_irq`) and `spsr_irq` are already banked, so we don't push them.

**Step 3 — Identify the source.**
With many peripherals possibly sharing the IRQ pin, the handler must ask the interrupt controller "who fired?". This is typically a register read like `LDR r0, [r1, #IRQ_STATUS]`.

**Step 4 — Run the ISR.**
The handler branches into the device-specific ISR. The ISR does the real work (read a byte from UART, copy a DMA descriptor, etc.) and **clears the interrupt at the source** (so the peripheral stops asserting `nIRQ`). If the source isn't cleared, as soon as we re-enable interrupts at step 6, we'll immediately re-fire — an infinite loop.

**Step 5 — Restore context.**
Pop the registers we saved in step 2:
```
LDMFD sp!, {r0-r12, lr}
```
The user's register state is back exactly as it was.

**Step 6 — Return and re-enable.**
A single instruction does it:
```
SUBS pc, lr, #4
```
This:
- Computes `pc = lr - 4` → re-runs the user instruction that was interrupted (subtract 4 to undo the pipeline-induced offset).
- Because the destination is `pc` and the suffix is `S`, the CPU also copies `spsr_irq → cpsr` → which restores the User mode AND clears the I bit (because the saved `cpsr` had I=0).

So in this single instruction, both "enable interrupts" and "return to caller" happen atomically.

**Why this scheme is too simple for big systems:**
A long-running ISR will block *all* other interrupts (including a critical FIQ-like emergency) for its full duration. Real systems use the next two schemes (nested, reentrant) to fix this.

---

# Question 3 — Nested Interrupt Handler

### Answer

A **nested interrupt handler** allows a *second* interrupt to be taken **while the first one is still being serviced**. This is done by **re-enabling interrupts before** the first ISR has fully finished — once the original interrupt source has been cleared.

This significantly improves responsiveness in real-time systems but adds complexity (and risk of bugs).

### The two design goals (from your notes)

1. **Respond to interrupts quickly** — don't make the handler wait for asynchronous exceptions, and don't make exceptions wait for the handler.
2. **Don't delay regular synchronous code** more than necessary.

### Why it's harder than non-nested

The handler must protect itself against:

- **Stack overflow** — repeated nested interrupts could keep pushing data onto the IRQ stack until it overflows.
- **`r14_irq` corruption** — if you stay in IRQ mode and a second IRQ fires, the new IRQ will overwrite `r14_irq`, losing the first handler's return address.
- **`spsr_irq` corruption** — same problem.

### How it works (Figure 9.9)

The entry code is identical to the non-nested handler (steps 1–4 are the same). The differences are at **exit time**, where the handler:

1. Tests a **flag** (set by the ISR) that says: *"is more processing needed?"*
2. **If NO** → behave like a non-nested handler and exit.
3. **If YES** → take extra steps:
   - **Re-enable interrupts** by switching out of IRQ mode into SVC or System mode (you cannot just clear the I bit while still in IRQ mode — see below).
   - **Perform a context switch** if needed: this involves *flattening* (emptying) the IRQ stack into the task's stack (typically the SVC stack), because you must not do a context switch while data is still on the IRQ stack.
   - All registers saved on the IRQ stack must be **transferred to the task's stack** (often the SVC stack).
   - Save remaining registers in a reserved memory block on the task stack called a **stack frame**.

### Why we MUST switch out of IRQ mode before re-enabling

If you stay in IRQ mode and re-enable interrupts:
- You execute a `BL subroutine` — this puts the return address into `r14_irq`.
- A new IRQ fires.
- The hardware's "save pc to lr" step **overwrites `r14_irq`** with the new return address.
- The first handler's return address is lost forever → crash.

By switching to SVC mode first, the BL stores its return address into `r14_svc`, and any new IRQ touches a *different* register (`r14_irq`) — no collision.

### Diagram

```
   IRQ fires
      |
      v
   +------------------------+
   | ENTRY (same as         |
   | non-nested handler)    |
   |  - hw saves cpsr, pc   |
   |  - sw saves context    |
   |  - identify source     |
   |  - call ISR            |
   +-----------+------------+
               |
               v
   +------------------------+
   | ISR clears the source  |
   | and sets a flag if     |
   | more work is needed    |
   +-----------+------------+
               |
               v
       check flag
          /     \
       NO        YES
        |          |
        v          v
   +-------+   +----------------------+
   |Restore|   | Switch IRQ -> SVC    |
   |context|   | (or System) mode     |
   |Return |   +-----------+----------+
   +-------+               |
                           v
                  +------------------+
                  | Flatten IRQ stack|
                  | into task stack  |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Re-enable IRQs   |
                  | (clear I bit)    |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Continue further |
                  | processing -- and|
                  | new IRQs may now |
                  | be taken, nesting|
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | When fully done, |
                  | restore context, |
                  | return to user.  |
                  +------------------+
```

### Step-by-step explanation

**Step 1 — Entry is identical to non-nested.**
Hardware saves `cpsr` and `pc`, switches to IRQ mode, jumps via vector. Software saves the user's `r0`–`r12, lr` to the IRQ stack. So far, same as before.

**Step 2 — ISR clears the source.**
The ISR talks to the peripheral and tells it "I got your message — stop asserting `nIRQ`". This is **essential**: if the source still asserts when we re-enable, we'll re-fire instantly.

**Step 3 — ISR sets a "more work needed" flag.**
For example, the ISR moved a byte from a UART, but a higher layer of code (a task) needs to do more — like wake up a sleeping process, or process a buffer. The ISR says "yes, please continue processing" by setting a software flag.

**Step 4 — Handler reads the flag.**
If clear: just unwind and return like a normal non-nested handler.
If set: prepare for nested operation.

**Step 5 — Mode switch from IRQ to SVC.**
Why? Because the IRQ-mode banked registers (`r14_irq`, `spsr_irq`) are *single-copy*. If we re-enable interrupts in IRQ mode and one fires, those registers get overwritten. By going to SVC (or System), we use SVC's banked `r14_svc` instead — and a new IRQ would land back in IRQ mode using `r14_irq`, not stepping on us.

**Step 6 — Flatten the IRQ stack into the SVC (task) stack.**
The IRQ stack is small (one of the design factors mentioned in §9.2.4 of your notes). Repeated nesting would overflow it. So we copy our saved context off the IRQ stack onto the larger task/SVC stack. After this, the IRQ stack is empty and ready to absorb a new nested interrupt.

**Step 7 — Re-enable interrupts.**
Now it's safe. Clear the I bit (using the MRS/BIC/MSR sequence from Q2). From this moment, a new IRQ can fire and be handled — and when it returns, control comes back to us.

**Step 8 — Continue processing, then return.**
Once the deferred work is done, restore everything and return to the user task.

**Why we use this scheme:**
Key benefit: a long ISR no longer blocks all other interrupts. Once the source is cleared and we've safely transferred state to the task stack, urgent new interrupts can be serviced. **But** since this scheme has *no priority filtering*, a low-priority interrupt CAN preempt a high-priority one — that's why we sometimes need the next scheme (reentrant).

---

# Question 4 — Reentrant Interrupt Handler Scheme

### Answer

A **reentrant interrupt handler** is a more advanced scheme that handles **multiple interrupts filtered by priority**. It is essential when *higher-priority interrupts must have lower latency* — a guarantee the plain nested handler cannot give.

The headline difference from a nested handler:

> **In a reentrant handler, interrupts are re-enabled VERY EARLY in the handler — much earlier than in a nested handler — to reduce latency.**

### Where the ISR runs

All ISRs in a reentrant handler are serviced in **SVC, System, Undefined, or Abort mode** — **never in IRQ mode**. The reason: see "BL corruption" below.

### The BL/`r14_irq` corruption problem (the central reason for this scheme's design)

Suppose we re-enable interrupts while still in IRQ mode. Then inside the handler we execute:
```
BL  process_data    ; puts return address into r14
```
This instruction stores its return address in `r14`. Since we are in IRQ mode, that means `r14_irq`. Now, an interrupt fires. The hardware steps "save pc to `lr`" overwrites `r14_irq` with the new return address. **Our subroutine return address is destroyed.** The system crashes.

**Solution:** swap to SVC (or System) mode *before* the BL. Then the BL writes to `r14_svc`, and a new IRQ overwrites `r14_irq` — no collision.

### Two more rules (very important)

1. **You must disable the interrupt at the source** (in the interrupt controller) **before re-enabling interrupts** in the `cpsr`. Otherwise the same source instantly re-fires → race condition or infinite interrupt loop.
2. **Mask out same-or-lower priority interrupts** in the controller. Most controllers have a *mask register* for this. Then re-enable IRQ in `cpsr`. Result: only **higher-priority** interrupts can preempt the current handler.

### How the IRQ stack is used

In a reentrant handler, the IRQ stack is **mostly unused** for the actual ISR work — that runs on the task's SVC stack. The IRQ stack register `r13_irq` is instead made to point to a **small 12-byte structure** that temporarily stores a few registers (typically `r0`–`r2` or similar) on interrupt entry. After mode switch, those values are moved to the task stack, and the IRQ stack is free again.

### Why prioritisation is mandatory

If interrupts are re-enabled but **not** filtered by priority:
- A low-priority interrupt can preempt a high-priority interrupt that is already in mid-service.
- The high-priority interrupt is now blocked by the low-priority one.
- Latency for the high-priority interrupt degrades to the same as the nested handler — defeating the entire point.

So: **reentrant without prioritisation = pointless**. Always mask same-or-lower priorities at entry.

### Step-by-step flow

```
   IRQ fires
      |
      v
   +-----------------------------------+
   | Hardware:                         |
   |  spsr_irq <- cpsr                 |
   |  r14_irq  <- pc                   |
   |  cpsr     <- IRQ mode, I=1        |
   |  pc       <- 0x18                 |
   +----------------+------------------+
                    |
                    v
   +-----------------------------------+
   | Save a few regs to the small      |
   | 12-byte structure pointed at by   |
   | r13_irq (temporary).              |
   +----------------+------------------+
                    |
                    v
   +-----------------------------------+
   | Identify the interrupt source.    |
   | Disable that source (and any      |
   | same-or-lower priority sources)   |
   | in the controller's mask register.|
   +----------------+------------------+
                    |
                    v
   +-----------------------------------+
   | Switch from IRQ mode to SVC (or   |
   | System) mode.                     |
   |   - Move saved context onto task  |
   |     (SVC) stack.                  |
   |   - Now we're using r14_svc, not  |
   |     r14_irq.                      |
   +----------------+------------------+
                    |
                    v
   +-----------------------------------+
   | Re-enable IRQ in cpsr (clear I).  |
   | From now on, ONLY higher-priority |
   | interrupts can preempt us.        |
   +----------------+------------------+
                    |
                    v
   +-----------------------------------+
   | Run the ISR. May call BL, may     |
   | take long. If a higher-priority   |
   | interrupt fires, it preempts and  |
   | nests safely (its r14 also goes   |
   | to its own task stack via the     |
   | same trick).                      |
   +----------------+------------------+
                    |
                    v
   +-----------------------------------+
   | ISR done. Disable IRQ in cpsr.    |
   | Re-enable the source mask.        |
   | Restore context from task stack.  |
   | Switch back to IRQ mode briefly.  |
   | SUBS pc, lr, #4 -> return to user |
   +-----------------------------------+
```

### Step-by-step explanation

**Step 1 — Hardware entry.**
Identical to all other schemes: save `cpsr/pc`, mask IRQ, jump to `0x18`.

**Step 2 — Why we use a 12-byte temporary structure on the IRQ stack.**
We need to store *some* working registers (typically just enough to do the source-identification and mode-switch dance) without using the IRQ stack as a real stack. By pointing `r13_irq` at a fixed 12-byte block, we can park 3 registers there briefly and reclaim them after the mode switch. The IRQ stack is *not* used for the deep ISR call chain — that lives on SVC's stack.

**Step 3 — Disable the source at the controller, BEFORE clearing I in cpsr.**
This is rule #1. If you re-enable I first, the same source instantly re-fires (it's still asserting). You must silence it at the controller first.

**Step 4 — Mask same-or-lower priority sources.**
This is what gives you the "priority filter". Suppose this is a priority-3 interrupt; we mask priorities 3, 4, 5, 6, ... Now only priority 0, 1, 2 (higher) can fire on us. So if we're servicing a fast device, only an *even faster* device can preempt — exactly what we want.

**Step 5 — Switch to SVC mode (the critical move).**
This is the line of code that distinguishes reentrant from nested. Once in SVC:
- Subsequent `BL` instructions write `r14_svc`, not `r14_irq`.
- We can now re-enable IRQ in cpsr safely.

**Step 6 — Re-enable IRQ.**
At this point, latency is minimised: any higher-priority interrupt can be taken almost immediately. This is why the reentrant scheme reduces latency — the re-enable happens **early**, before the heavy ISR work.

**Step 7 — Do the ISR work.**
Long, complex, can call helper functions, can run for many cycles — none of it blocks higher-priority interrupts.

**Step 8 — Tear down.**
Reverse of setup: re-disable IRQ, re-enable the source mask in the controller, restore context from SVC stack, switch back to IRQ mode just long enough to do `SUBS pc, lr, #4`, which simultaneously restores `cpsr` and returns to the user.

### Comparison cheat-sheet (for the exam)

| Feature | Non-nested | Nested | Reentrant |
|---|---|---|---|
| Multiple interrupts at once? | No | Yes | Yes |
| Priority filtering? | No | No | **Yes** |
| When are interrupts re-enabled? | Only at exit | Late (after source cleared, before final processing) | **Early** (after source masked, before main ISR work) |
| Mode where ISR runs | IRQ | IRQ → SVC near exit | Almost entirely in SVC |
| Latency for high-prio interrupts | Worst | Medium | **Best** |
| Complexity | Low | Medium | High |
| BL `r14_irq` corruption risk | None | None | Avoided by switching to SVC |

---

# Final exam survival tips

1. **Always start an answer by stating *what* and *why*, then *how*.** Examiners reward clarity over volume.
2. **If a question asks for a diagram**, draw at least the box-and-arrow flow even if rough — it shows you understand the order of events.
3. **Memorise these "magic numbers"**:
   - I bit mask = `0x80`, F bit mask = `0x40`
   - Vector offsets: Reset `0x00`, Undef `0x04`, SWI `0x08`, PAbt `0x0C`, DAbt `0x10`, IRQ `0x18`, FIQ `0x1C`
   - Return: `SUBS pc, lr, #4` for IRQ/FIQ
4. **The 4 hardware actions on every exception** (save cpsr→spsr, save pc→lr, set new cpsr, set pc to vector) are the *single most asked* fact in this module. Memorise verbatim.
5. **Priority order (highest → lowest)**: Reset → Data Abort → FIQ → IRQ → Prefetch Abort → SWI/Undef. Mnemonic: *"**R**eally **D**oes **F**ix **I**ssues **P**retty **S**oon"* (Reset, Data, FIQ, IRQ, PAbt, SWI).
6. **One-line distinctions you should be able to fire instantly**:
   - IRQ vs FIQ → "FIQ banks `r8`-`r12`, sits at `0x1C`, masks both I and F."
   - Non-nested vs nested → "Nested re-enables interrupts before exit."
   - Nested vs reentrant → "Reentrant filters by priority and runs ISRs in SVC mode."

Good luck for tomorrow — you've got this.
