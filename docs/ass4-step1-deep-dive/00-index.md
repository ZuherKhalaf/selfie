# Ass4 Step 1 Deep Dive (Bit-Level)

Diese Sammlung ist auf **Assignment 4, Step 1** fokussiert:  
`<<`/`>>` korrekt als `sll`/`srl` erzeugen.

## Empfohlene Reihenfolge

1. [01-riscu-semantics.md](01-riscu-semantics.md)
2. [02-selfie-constants-f3-f7.md](02-selfie-constants-f3-f7.md)
3. [03-selfie-encode-r-format.md](03-selfie-encode-r-format.md)
4. [04-selfie-emitters.md](04-selfie-emitters.md)
5. [05-selfie-decode.md](05-selfie-decode.md)
6. [06-selfie-do-functions.md](06-selfie-do-functions.md)
7. [07-book-reading-map.md](07-book-reading-map.md)
8. [08-riscv-unprivileged-isa-reference.md](08-riscv-unprivileged-isa-reference.md)

## Lernziel

Nach diesen Dateien solltest du erklären können:

- warum `F3_SLL=1`, `F3_SRL=5`, `F7_SLL=0`, `F7_SRL=0`
- was ein Emitter macht
- wie aus C*-Operatoren echte RISC-U Instruktionsbits werden
- warum Decoder und Emulator zusätzlich nötig sind
