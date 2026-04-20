# Assignment 4: `bitwise-shift-execution` — Detailed Implementation Instructions

Diese Datei ist eine klare Schritt-für-Schritt-Anleitung für Assignment 4 auf Basis der funktionierenden Lösung.

---

## Ziel von Assignment 4

In Assignment 3 wurden `<<` und `>>` nur geparst (Frontend).  
In Assignment 4 müssen sie **korrekt ausgeführt** werden:

1. echte Codegenerierung als `sll`/`srl`
2. Decoder erkennt die Instruktionen richtig
3. Emulator (`mipster`) führt sie aus
4. Disassembler druckt sie korrekt
5. `riscu.md` dokumentiert sie

---

## Dateien, die du ändern musst

- `selfie.c`
- `riscu.md`

---

## Step 1 — Code Generation für `sll`/`srl`

### 1.1 Neue Funct-Codes hinzufügen
Bei den bestehenden `F3_*` und `F7_*` Konstanten in `selfie.c` ergänzen:

```c
uint64_t F3_SLL = 1; // 001
uint64_t F3_SRL = 5; // 101

uint64_t F7_SLL = 0; // 0000000
uint64_t F7_SRL = 0; // 0000000
```

### 1.2 Emitter deklarieren
Bei den `emit_*` Prototypen ergänzen:

```c
void emit_sll(uint64_t rd, uint64_t rs1, uint64_t rs2);
void emit_srl(uint64_t rd, uint64_t rs1, uint64_t rs2);
```

### 1.3 Emitter implementieren
Neben `emit_add`, `emit_sub`, ... ergänzen:

```c
void emit_sll(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_SLL, rs2, rs1, F3_SLL, rd, OP_OP));
}

void emit_srl(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_SRL, rs2, rs1, F3_SRL, rd, OP_OP));
}
```

### 1.4 `compile_shift()` von Platzhalter auf echte Shifts umstellen
In `compile_shift()` ersetzen:

```c
if (operator_symbol == SYM_LSHIFT)
  emit_sll(previous_temporary(), previous_temporary(), current_temporary());
else
  emit_srl(previous_temporary(), previous_temporary(), current_temporary());
```

---

## Step 2 — Interpreter/Execution (Decoder + mipster)

### 2.1 Instruction-IDs erweitern
Bei den Instruction IDs ergänzen:

```c
uint64_t ECALL = 14;
uint64_t SLL   = 15;
uint64_t SRL   = 16;
```

> Wichtig: `ECALL` darf nicht entfernt werden.

### 2.2 Prototypen für Ausführung ergänzen
Bei `do_add()`, `do_sub()`, ... ergänzen:

```c
void do_sll();
void do_srl();
```

### 2.3 Counter ergänzen
Zu den Instruction Countern ergänzen:

- `ic_sll`, `ic_srl`
- `nopc_sll`, `nopc_srl`

Dann ergänzen in:

- `reset_binary_counters()`
- `reset_nop_counters()`
- `get_total_number_of_instructions()`
- `get_total_number_of_nops()`
- `print_instruction_counters()`

### 2.4 `do_sll()` und `do_srl()` implementieren
Analog zu `do_add()`/`do_sub()`:

```c
void do_sll() {
  uint64_t next_rd_value;

  read_register(rs1);
  read_register(rs2);

  if (rd != REG_ZR) {
    next_rd_value = left_shift(*(registers + rs1), *(registers + rs2));

    if (*(registers + rd) != next_rd_value)
      *(registers + rd) = next_rd_value;
    else
      nopc_sll = nopc_sll + 1;
  } else
    nopc_sll = nopc_sll + 1;

  write_register(rd);
  pc = pc + INSTRUCTIONSIZE;
  ic_sll = ic_sll + 1;
}

void do_srl() {
  uint64_t next_rd_value;

  read_register(rs1);
  read_register(rs2);

  if (rd != REG_ZR) {
    next_rd_value = right_shift(*(registers + rs1), *(registers + rs2));

    if (*(registers + rd) != next_rd_value)
      *(registers + rd) = next_rd_value;
    else
      nopc_srl = nopc_srl + 1;
  } else
    nopc_srl = nopc_srl + 1;

  write_register(rd);
  pc = pc + INSTRUCTIONSIZE;
  ic_srl = ic_srl + 1;
}
```

### 2.5 Decoder korrekt erweitern (kritisch!)
In `decode()` bei `opcode == OP_OP`:

```c
if (funct3 == F3_ADD) {
  if (funct7 == F7_ADD)
    is = ADD;
  else if (funct7 == F7_SUB)
    is = SUB;
  else if (funct7 == F7_MUL)
    is = MUL;
} else if (funct3 == F3_REMU) {
  if (funct7 == F7_REMU)
    is = REMU;
} else if (funct3 == F3_SLTU) {
  if (funct7 == F7_SLTU)
    is = SLTU;
} else if (funct3 == F3_SLL) {
  if (funct7 == F7_SLL)
    is = SLL;
} else if (funct3 == F3_SRL) {
  if (funct7 == F7_SRL)
    is = SRL;
  else if (funct7 == F7_DIVU)
    is = DIVU;
}
```

**Warum so?**  
`srl` und `divu` teilen sich `funct3 == 5`; Unterscheidung nur über `funct7`:

- `srl`: `funct7 == 0`
- `divu`: `funct7 == 1`

Wenn du vorher schon `funct3 == F3_DIVU` abfängst, wird `srl` nie erreicht.

### 2.6 Execute-Pfade ergänzen
In allen drei Funktionen ergänzen:

- `execute()`
- `execute_record()`
- `execute_debug()`

jeweils Branches für `SLL` und `SRL`.

Für `execute_debug()` ist es korrekt, die bestehenden
`print_add_sub_mul_divu_remu_sltu_*` Helfer mitzubenutzen.

---

## Step 3 — Disassembler Support

### 3.1 MNEMONICS-Array vergrößern
In `init_disassembler()`:

```c
MNEMONICS = smalloc((SRL + 1) * sizeof(uint64_t*));
```

### 3.2 Neue Mnemonics hinzufügen

```c
*(MNEMONICS + SLL) = (uint64_t) "sll";
*(MNEMONICS + SRL) = (uint64_t) "srl";
```

### 3.3 `print_instruction()` erweitern
SLL/SRL wie andere R-Format-Compute-Instruktionen drucken:

```c
else if (is == SLL)
  return print_add_sub_mul_divu_remu_sltu();
else if (is == SRL)
  return print_add_sub_mul_divu_remu_sltu();
```

---

## Step 4 — `riscu.md` anpassen

1. „14 instructions“ auf „16 instructions“ ändern.
2. Neue Sektion einfügen:

```md
#### Shift

`sll rd,rs1,rs2`: `rd = rs1 << rs2; pc = pc + 4`

`srl rd,rs1,rs2`: `rd = rs1 >> rs2; pc = pc + 4`
```

---

## Step 5 — Typische Fehler (und Fix)

### Fehler A: `unknown instruction ... opcode 0x33` bei Right-Shift
Ursache: Decoder-Reihenfolge falsch (`F3_DIVU` vor `F3_SRL`).  
Fix: SRL/DIVU über `funct3==5` + `funct7` disambiguieren (siehe Step 2.5).

### Fehler B: Assembly-format checks fail bei `-s`
Ursache: `print_instruction()` oder `MNEMONICS` für `srl`/`sll` unvollständig.  
Fix: Step 3 vollständig umsetzen.

### Fehler C: Bootstrapping warning wegen Einrückung
Achte auf saubere Einrückung (z. B. `init_memory()`).

---

## Step 6 — Verifikation

Im Selfie-Root:

```bash
make selfie
make self
./grader/self.py bitwise-shift-execution
```

Erwartung: alle Tests passen, `your grade is: 2`.

