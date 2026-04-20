# Assignment 5: `bitwise-and-or-not` — Detailed Implementation Instructions

Diese Datei ist eine klare Schritt-für-Schritt-Anleitung für Assignment 5 auf Basis der funktionierenden Lösung aus `selfie_backup`.

---

## Ziel von Assignment 5

In Assignment 4 wurden `<<` und `>>` bereits vollständig umgesetzt.  
In Assignment 5 müssen bitweise logische Operatoren in C\* und RISC-U ergänzt werden:

1. Scanner und Parser erkennen `&`, `|`, `~`
2. korrekte Präzedenz für `&` und `|`
3. Codegenerierung mit `and`, `or`, `xori`
4. Decoder und Emulator (`mipster`) führen die Instruktionen korrekt aus
5. Disassembler druckt die neuen Instruktionen korrekt
6. `grammar.md` und `riscu.md` dokumentieren die Erweiterung

---

## Dateien, die du ändern musst

- `selfie.c`
- `grammar.md`
- `riscu.md`

---

## Step 1 — Scanner für `&`, `|`, `~`

### 1.1 Zeichen- und Symbolkonstanten ergänzen
**Cursor:** `selfie.c` bei `char CHAR_EXCLAMATION` (ca. L428) und `SYM_LSHIFT` (ca. L467)

In `selfie.c` ergänzen:

```c
char CHAR_AMPERSAND = '&';
char CHAR_PIPE      = '|';
char CHAR_TILDE     = '~';

uint64_t SYM_BITWISEAND = 36; // &
uint64_t SYM_BITWISEOR  = 37; // |
uint64_t SYM_TILDE      = 38; // ~
```

### 1.2 `init_scanner()` erweitern
**Cursor:** `selfie.c` bei `void init_scanner()` (ca. L513)

`SYMBOLS`-Array groß genug machen und Strings eintragen:

```c
SYMBOLS = smalloc((SYM_TILDE + 1) * sizeof(uint64_t*));

*(SYMBOLS + SYM_BITWISEAND) = (uint64_t) "&";
*(SYMBOLS + SYM_BITWISEOR)  = (uint64_t) "|";
*(SYMBOLS + SYM_TILDE)      = (uint64_t) "~";
```

### 1.3 `get_symbol()` erweitern
**Cursor:** `selfie.c` bei `void get_symbol()` (ca. L3787)

Einzelzeichen-Token hinzufügen:

```c
} else if (character == CHAR_AMPERSAND) {
  get_character();
  symbol = SYM_BITWISEAND;
} else if (character == CHAR_PIPE) {
  get_character();
  symbol = SYM_BITWISEOR;
} else if (character == CHAR_TILDE) {
  get_character();
  symbol = SYM_TILDE;
```

---

## Step 2 — Parser und Präzedenz (`&` unter `|`, beide unter Vergleich)

### 2.1 Neue Prototypen ergänzen
**Cursor:** `selfie.c` im Prototypenblock bei `compile_expression/compile_shift` (ca. L729-L733)

Bei den Parser-Prototypen ergänzen:

```c
uint64_t compile_bitwiseor();  // returns type
uint64_t compile_bitwiseand(); // returns type
```

### 2.2 Neue Helper ergänzen
**Cursor:** `selfie.c` bei `uint64_t is_shift()` (ca. L4391)

```c
uint64_t is_bitwiseand() {
  if (symbol == SYM_BITWISEAND)
    return 1;
  else
    return 0;
}

uint64_t is_bitwiseor() {
  if (symbol == SYM_BITWISEOR)
    return 1;
  else
    return 0;
}
```

### 2.3 `compile_expression()` umhängen
**Cursor:** `selfie.c` bei `uint64_t compile_expression()` (ca. L4948)

Statt direkt `compile_shift()` zu verwenden:

```c
ltype = compile_bitwiseor();
...
rtype = compile_bitwiseor();
```

### 2.4 `compile_bitwiseor()` einfügen
**Cursor:** `selfie.c` zwischen `compile_expression()` und `compile_shift()` (ca. L5027-Bereich)

Zwischen `compile_expression()` und `compile_shift()`:

```c
uint64_t compile_bitwiseor() {
  uint64_t ltype;
  uint64_t rtype;

  ltype = compile_bitwiseand();

  while (is_bitwiseor()) {
    get_symbol();

    rtype = compile_bitwiseand();

    if (ltype != rtype)
      type_warning(ltype, rtype);

    ltype = UINT64_T;

    emit_or(previous_temporary(), previous_temporary(), current_temporary());

    tfree(1);
  }

  return ltype;
}
```

### 2.5 `compile_bitwiseand()` einfügen
**Cursor:** `selfie.c` direkt vor `uint64_t compile_shift()` (ca. L5027)

Direkt vor `compile_shift()`:

```c
uint64_t compile_bitwiseand() {
  uint64_t ltype;
  uint64_t rtype;

  ltype = compile_shift();

  while (is_bitwiseand()) {
    get_symbol();

    rtype = compile_shift();

    if (ltype != rtype)
      type_warning(ltype, rtype);

    ltype = UINT64_T;

    emit_and(previous_temporary(), previous_temporary(), current_temporary());

    tfree(1);
  }

  return ltype;
}
```

### 2.6 `~` als Faktorstart erlauben
**Cursor:** `selfie.c` bei `uint64_t is_factor()` (ca. L4320)

In `is_factor()` ergänzen:

```c
else if (symbol == SYM_TILDE)
  return 1;
```

---

## Step 3 — Unary `~` in `compile_factor()`

### 3.1 Optionalen `~`-Teil parsen
**Cursor:** `selfie.c` bei `uint64_t compile_factor()` (ca. L5168)

Analog zu `negative` und `dereference` ein Flag hinzufügen:

```c
if (symbol == SYM_TILDE) {
  bitnot = 1;
  get_symbol();
} else
  bitnot = 0;
```

### 3.2 `~x` als `xori x, -1` emittieren
**Cursor:** `selfie.c` in `compile_factor()` beim Block `if (negative) { ... }`

Nach dem Laden/Typcheck:

```c
if (bitnot) {
  if (type != UINT64_T) {
    type_warning(UINT64_T, type);
    type = UINT64_T;
  }
  // ~x = xori x, -1 (XOR with all-ones)
  emit_xori(current_temporary(), current_temporary(), -1);
}
```

---

## Step 4 — Code Generation + ISA-Konstanten

### 4.1 Instruction-IDs ergänzen
**Cursor:** `selfie.c` bei den Instruction IDs (`SLL`, `SRL`) um ca. L1655

```c
uint64_t AND  = 17; // and
uint64_t OR   = 18; // or
uint64_t XORI = 19; // xori
```

### 4.2 `F3_*` / `F7_*` ergänzen
**Cursor:** `selfie.c` im Funct-Code-Bereich um ca. L1020-L1031

```c
uint64_t F3_AND  = 7; // 111
uint64_t F3_OR   = 6; // 110
uint64_t F3_XORI = 4; // 100

uint64_t F7_AND = 0; // 0000000
uint64_t F7_OR  = 0; // 0000000
```

### 4.3 Emitter deklarieren
**Cursor:** `selfie.c` im Emitter-Prototypenblock bei `emit_sll` (ca. L1088)

```c
void emit_and(uint64_t rd, uint64_t rs1, uint64_t rs2);
void emit_or(uint64_t rd, uint64_t rs1, uint64_t rs2);
void emit_xori(uint64_t rd, uint64_t rs1, uint64_t immediate);
```

### 4.4 Emitter implementieren
**Cursor:** `selfie.c` im Emitter-Implementierungsblock bei `emit_sll` (ca. L7126)

```c
void emit_and(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_AND, rs2, rs1, F3_AND, rd, OP_OP));
  ic_and = ic_and + 1;
}

void emit_or(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_OR, rs2, rs1, F3_OR, rd, OP_OP));
  ic_or = ic_or + 1;
}

void emit_xori(uint64_t rd, uint64_t rs1, uint64_t immediate) {
  emit_instruction(encode_i_format(immediate, rs1, F3_XORI, rd, OP_IMM));
  ic_xori = ic_xori + 1;
}
```

---

## Step 5 — Decoder + Execution (mipster)

### 5.1 Decode für `xori` im `OP_IMM`-Zweig
**Cursor:** `selfie.c` bei `void decode()` (ca. L9974)

```c
if (funct3 == F3_ADDI)
  is = ADDI;
else if (funct3 == F3_XORI)
  is = XORI;
```

### 5.2 Decode für `and`/`or` im `OP_OP`-Zweig
**Cursor:** `selfie.c` in `decode()` im `else if (opcode == OP_OP)`-Block

```c
...
} else if (funct3 == F3_REMU) {
  if (funct7 == F7_REMU)
    is = REMU;
  else if (funct7 == F7_AND)
    is = AND;
} else if (funct3 == F3_OR) {
  if (funct7 == F7_OR)
    is = OR;
}
```

### 5.3 Kritischer Decoder-Fall
`F3_AND == F3_REMU == 7`.  
Wenn `AND` in einem separaten `funct3==7`-Branch nach `REMU` geprüft wird, ist der Branch tot.

**Korrekt:** im gemeinsamen `funct3==7`-Branch über `funct7` trennen:
- `F7_REMU` -> `REMU`
- `F7_AND` -> `AND`

### 5.4 Ausführungsfunktionen ergänzen
**Cursor:** Prototypen bei `void do_sll();` (ca. L1594), Implementierungen bei `void do_sll()` (ca. L9164)

Prototypen:

```c
void do_and();
void do_or();
void do_xori();
```

Implementierung (analog zu anderen ALU-Instruktionen):

```c
// and
next_rd_value = *(registers + rs1) & *(registers + rs2);

// or
next_rd_value = *(registers + rs1) | *(registers + rs2);

// xori (imm ist bereits sign-extended)
next_rd_value = (*(registers + rs1) | imm) - (*(registers + rs1) & imm);
```

### 5.5 Execute-Pfade ergänzen
**Cursor:** `selfie.c` bei `execute()` (ca. L10065), `execute_record()` (ca. L10110), `execute_debug()` (ca. L10176)

In allen drei Funktionen Branches ergänzen:

- `execute()`
- `execute_record()`
- `execute_debug()`

für `AND`, `OR`, `XORI`.

---

## Step 6 — Disassembler + Counter

### 6.1 Instruction- und NOP-Counter ergänzen
**Cursor:** `selfie.c` bei `ic_sll` (ca. L1219) und `nopc_sll` (ca. L1855)

Neue Zähler:

- `ic_and`, `ic_or`, `ic_xori`
- `nopc_and`, `nopc_or`, `nopc_xori`

Dann ergänzen in:

- `reset_binary_counters()`
- `reset_nop_counters()`
- `get_total_number_of_instructions()`
- `get_total_number_of_nops()`
- `print_instruction_counters()`

### 6.2 `MNEMONICS`-Array und Namen erweitern
**Cursor:** `selfie.c` bei `void init_disassembler()` (ca. L1688)

Arraygröße:

```c
MNEMONICS = smalloc((XORI + 1) * sizeof(uint64_t*));
```

Mnemonics:

```c
*(MNEMONICS + AND)  = (uint64_t) "and";
*(MNEMONICS + OR)   = (uint64_t) "or";
*(MNEMONICS + XORI) = (uint64_t) "xori";
```

### 6.3 `print_instruction()` ergänzen
**Cursor:** `selfie.c` bei `uint64_t print_instruction()` (ca. L9694)

- `and`/`or` als R-Format-Recheninstruktionen ausgeben
- `xori` als I-Format-Instruktion ausgeben

---

## Step 7 — `grammar.md` und `riscu.md` anpassen

### 7.1 `grammar.md`
**Cursor:** `grammar.md` bei `expression = ...` (ca. L54) und `factor = ...` (ca. L62)

Produktion auf die neuen Ebenen umstellen:

```ebnf
expression = bitwiseor [ ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) bitwiseor ] .
bitwiseor  = bitwiseand { "|" bitwiseand } .
bitwiseand = shift { "&" shift } .
factor     = [ cast ] [ "-" ] [ "~" ] [ "*" ] ( ... ) .
```

Zusätzlich Symbol-Liste um `&`, `|`, `~` ergänzen.

### 7.2 `riscu.md`
**Cursor:** `riscu.md` bei "RISC-U consists of just ..." (ca. L7) und Section `#### Arithmetic` (ca. L35)

RISC-U um `and`, `or`, `xori` erweitern und Instruktionsanzahl anpassen:

```md
#### Bitwise

`and rd,rs1,rs2`: `rd = rs1 & rs2; pc = pc + 4`

`or rd,rs1,rs2`: `rd = rs1 | rs2; pc = pc + 4`

`xori rd,rs1,imm`: `rd = rs1 ^ imm; pc = pc + 4`
```

---

## Step 8 — Typische Fehler (und Fix)

### Fehler A: `and` wird nie erkannt
Ursache: `F3_AND` kollidiert mit `F3_REMU` (`funct3 == 7`).  
Fix: gemeinsamer Branch mit Unterscheidung über `funct7`.

### Fehler B: `~x` liefert falsche Werte
Ursache: `do_xori()` sign-extended `imm` nochmals, obwohl `decode_i_format()` das schon gemacht hat.  
Fix: in `do_xori()` direkt `imm` verwenden.

### Fehler C: Assembly-/Disassembler-Checks fail
Ursache: Mnemonics oder `print_instruction()` für `and`/`or`/`xori` unvollständig.  
Fix: Step 6 vollständig umsetzen.

---

## Step 9 — Verifikation

Im Selfie-Root:

```bash
make selfie
make self
./grader/self.py bitwise-and-or-not
```

Erwartung: alle Tests passen, `your grade is: 2`.
