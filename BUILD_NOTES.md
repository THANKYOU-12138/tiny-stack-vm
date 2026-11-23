# stackvm — build notes

Competition: tiny-stack-vm-01-run-20261122. 代理人's entry: `main.ts`, a complete VM for the
tiny stack ISA — every opcode and trap, in the reference VM's order of checks. The house
baseline (plain-vm) skips the hard rules; this entry implements them all.

## Scores

- Practice (40 instances; 800 stdout + 200 exit points each): **40,000 of 40,000** — every
  stdout byte-exact, every exit code exact. The house baseline scores 26,800 on the same
  set, so the "strictly above baseline" gate is cleared with room to spare.
- Kit grade-local (`--driver process`): every instance 1000 of 1000, L1 regrade reproduced
  byte for byte; "prize no" only for the not-yet-attached transcripts (the parent attaches
  them at submission, as with mend).
- Fuzz: 1,802 random/mutated programs run against the reference VM — 0 mismatches.

## Tricky parts

- Trap order per step: step limit 10 → pc bounds 9 → unknown opcode 5 → immediate cut off
  9 → underflow 2 → overflow 3 → the instruction's own trap (4/6/7/8/11). Stack needs are
  checked before execution: DUP or IN on a full stack traps 3; IN on an empty stack is fine.
- 0x1F is not an opcode (a hole in 0x10-0x22): trap 5. Caught by fuzzing.
- MUL is Math.imul; DIV/MOD keep the INT_MIN/-1 corners; DIVU/MODU/LTU read operands
  unsigned; shift counts keep the low 5 bits; PUSH8 sign-extends; EXIT keeps the low byte;
  a trapping PRINT/PUTC writes nothing; output before a trap is kept.
- Size: main.ts 5,877 bytes + this file stay under the 8,192-byte repo cap.

## Bundle dry-run

From the kit directory (`bundle --repo ~/.makefun/work/stackvm-bot --dry-run`): **the build
session is discoverable** — 1 Muse app session (subagent a375d186-…, also holding the
sitegen build), 61 turns, 206 lines kept, AGENTS.md + system prompt collected. A real
bundle is not writable from inside this run; the parent closes the same gaps as with mend
at bundle time: `--mcp`, `--provider`/`--effort`, and the muse.db usage rows for agents
bff0c90e-… (parent) and a375d186-… (subagent), bundled from the top-level session.
