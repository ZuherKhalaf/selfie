# 06 — `do_*` Funktionen: Ausführung im Emulator

Quelle: `selfie.c` um Zeilen ~9164-9204 (`do_sll`, `do_srl`)

## Relevante Struktur (beide analog)

```c
void do_sll() {
  read_register(rs1);
  read_register(rs2);
  ...
  next_rd_value = left_shift(*(registers + rs1), *(registers + rs2));
  ...
  write_register(rd);
  pc = pc + INSTRUCTIONSIZE;
  ic_sll = ic_sll + 1;
}
```

## Line-by-line Interpretation

1. `read_register(rs1/rs2)`
   - Profiling/Validierung: Quelle wurde gelesen.
2. `if (rd != REG_ZR)`
   - RISC-V-Regel: `x0` (zero) ist write-ignored.
3. `next_rd_value = left_shift(...)` oder `right_shift(...)`
   - tatsächliche Semantik der Instruktion.
4. `if (*(registers + rd) != next_rd_value) ... else nopc_*++`
   - NOP-Effekt wird statistisch erfasst.
5. `write_register(rd)`
   - Profiling/Validierung: Ziel wurde beschrieben.
6. `pc = pc + INSTRUCTIONSIZE`
   - normaler Kontrollfluss zur nächsten Instruktion.
7. `ic_sll++ / ic_srl++`
   - Instruktionszähler für Profilausgabe.

## Wichtig für die Prüfung

- Emitter erzeugt Bits.
- Decoder erkennt Bits.
- `do_*` gibt ihnen Laufzeitsemantik.

Erst alle drei zusammen machen Assignment 4 „vollständig“.

