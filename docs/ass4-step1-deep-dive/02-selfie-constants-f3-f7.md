# 02 — `selfie.c`: `F3_*` und `F7_*` für `sll`/`srl`

Quelle: `selfie.c` um Zeilen ~1004-1032

## Relevante Konstanten

```c
uint64_t OP_OP    = 51; // 0110011, R format

uint64_t F3_SLL   = 1;  // 001
uint64_t F3_SRL   = 5;  // 101

uint64_t F7_SLL   = 0;  // 0000000
uint64_t F7_SRL   = 0;  // 0000000
uint64_t F7_DIVU  = 1;  // 0000001
```

## Line-by-line Interpretation

### `OP_OP = 51 (0110011)`
- Das ist der Opcode für RISC-V Register-Register Recheninstruktionen.
- `add/sub/mul/divu/remu/sltu/sll/srl` liegen alle in dieser Klasse.

### `F3_SLL = 1 (001)`
- `funct3=001` kennzeichnet innerhalb von `OP_OP` die Shift-Left-Familie.

### `F3_SRL = 5 (101)`
- `funct3=101` wird für Right-Shift/Division-Familie verwendet.
- Deshalb ist zusätzliche Unterscheidung nötig.

### `F7_SLL = 0`, `F7_SRL = 0`
- Beide nutzen `funct7=0000000`.
- Das ist kein Problem, weil `funct3` bereits verschieden ist (`001` vs `101`).

### Warum Konflikt mit `divu`?
- `divu` nutzt auch `funct3=101`, aber `funct7=0000001`.
- Daher Decoder-Regel: bei `funct3=101` über `funct7` entscheiden:
  - `0` -> `srl`
  - `1` -> `divu`

## Woher kommen diese Zahlen?

Nicht frei gewählt.  
Sie stammen aus der RISC-V ISA-Kodierung (RV64I/M, R-Format-Tabellen).

