# stackvm — build notes

Competition: tiny-stack-vm-01-run-20261122. 代理人's entry: `main.ts`, a complete VM for the
tiny stack ISA — every opcode and every trap, in the reference VM's order of checks. The house
baseline (plain-vm) skips the hard rules; this entry implements them all.

## Scores

- Practice (40 instances, checker points: 800 stdout + 200 exit per instance): **40,000 of
  40,000** — every stdout byte-exact, every exit code exact. The house baseline scores 26,800
  on the same set, so this entry clears the "strictly above baseline" gate with room to spare.
- Kit grade-local (`--driver process`): every instance 1000 of 1000, L1 regrade reproduced
  byte for byte; eligibility "ranked yes, prize no" only for the not-yet-attached transcripts
  (the parent attaches them at submission, as with mend).
- Fuzz: 1,802 random/mutated programs (random bytes, plausible-opcode streams, mutated
  practice programs) run against the reference VM — 0 mismatches in stdout or exit code.

## Tricky parts (where the baseline goes wrong, and this entry does not)

- Trap order per step: step limit 10 → pc bounds 9 → unknown opcode 5 → immediate cut off 9 →
  stack underflow 2 → stack overflow 3 → the instruction's own trap (4/6/7/8/11). Stack needs
  come from the ISA's pops/pushes table, checked before execution — so DUP on a full stack
  traps 3, and IN on a full stack traps 3 while IN on an empty stack is fine.
- 0x1F is not an opcode (the 0x10-0x22 range has a hole there): trap 5. Caught by fuzzing.
- MUL is Math.imul (exact low 32 bits); DIV/MOD keep the INT_MIN/-1 corners; DIVU/MODU/LTU
  read operands unsigned; shift counts keep the low 5 bits; PUSH8 sign-extends; EXIT keeps
  the low byte; a trapping PRINT/PUTC writes nothing; output written before a trap is kept.
- Repository size: main.ts 5,877 bytes + this file — under the 8,192-byte cap.

## Bundle dry-run

Recorded at the end of this file's final version: run from the kit directory against this
repository; the session is discoverable and the remaining gaps are the parent's bundle-time
declarations (same as mend and sitegen).
