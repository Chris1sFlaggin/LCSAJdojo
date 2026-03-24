# LCSAJdump Efficiency CTF Dojo

A [pwn.college](https://pwn.college) dojo showcasing why [lcsajdump](https://github.com/chris-j-akers/lcsajdump)'s forward-scan approach finds ROP gadgets that ropper and ROPgadget structurally cannot.

## The Core Problem

Here's the asymmetry: `lcsajdump` scans **forward** from function entry points and accepts any branch or call instruction as a valid LCSAJ tail. Backward-scan tools like ropper and ROPgadget work the opposite way—they start with a fixed set of seed opcodes and scan backwards. If an instruction doesn't appear in their seed table, it's unreachable by design, no matter how deep you search.

The challenges exploit this limitation by hiding exploitable gadgets behind terminators that backward-scan tools can't seed:

| Terminator | Encoding | Barrier type |
|---|---|---|
| x86-64 `call rel32` | `E8 xx xx xx xx` | Absent from ropper/ROPgadget seed tables at any depth |
| ARM64 `bl imm26` | `94xxxxxx` | Absent from both tools' ARM64 JOP seeds (only `br reg`/`blr reg` are seeded) |
| RISC-V `jal` | — | ropper: no RISC-V support; ROPgadget: finds gadgets but chain exceeds buffer limit |

The anonymous gadget lives in `.text` with no label — invisible not because of section hiding, but because `call rel32` is never in the seed tables. There's no terminator for them to scan backward from, at any depth.

## Modules

### x86-64

Proof of concept challenges for x86-64.

| Challenge | Description |
|---|---|
| RisckyMonke | Can you find the trigger? (`/challenge/RisckyMonke` is setuid root) |

## Dojo Structure

```
dojo.yml              # top-level dojo definition
x86-64/               # x86-64 module
  RisckyMonke/        # challenge binary and description
images/
  LCSAJ.png           # dojo award image
```

## Running Challenges

Challenges are hosted on [pwn.college](https://pwn.college). When you start a challenge, your environment contains:
- `/flag` — readable only by root
- `/challenge/<name>` — setuid root binary

The goal is to exploit the binary to read `/flag`.

## Key Tools

```sh
# Forward scan — finds the anonymous gadget in .text
lcsajdump <binary> --all-exec -v

# Backward scan — misses the anonymous gadget
ropper --file <binary> --type rop
ROPgadget --binary <binary>

# Inspect section layout
readelf -S <binary>
```

## References

- [lcsajdump](https://github.com/chris-j-akers/lcsajdump) — the tool being validated
- [pwncollege/dojo-update](https://github.com/pwncollege/dojo-update) — automatic dojo update via GitHub Actions
