# 04 — Emitter in `selfie.c`: Warum `emit_*` existiert

Quelle: `selfie.c` um Zeilen ~7102-7132

## Relevante Ausschnitte

```c
void emit_add(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_ADD, rs2, rs1, F3_ADD, rd, OP_OP));
  ic_add = ic_add + 1;
}

void emit_sll(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_SLL, rs2, rs1, F3_SLL, rd, OP_OP));
}

void emit_srl(uint64_t rd, uint64_t rs1, uint64_t rs2) {
  emit_instruction(encode_r_format(F7_SRL, rs2, rs1, F3_SRL, rd, OP_OP));
}
```

## Was ist ein Emitter?

Ein Emitter ist eine Codegen-Helferfunktion, die:

1. Instruktionsfelder zusammenstellt (`encode_*_format`)
2. das fertige Instruktionswort in den Code schreibt (`emit_instruction`)
3. optional Profilzähler erhöht

Kurz: **Emitter erzeugen Maschinencode**, sie führen ihn nicht aus.

## Warum `emit_add`, `emit_divu`, `emit_sll`, ...?

- Lesbarkeit: `compile_*`-Funktionen bleiben fachlich klar.
- Wiederverwendung: Encodierungslogik ist zentral.
- Wartbarkeit: Feldänderungen an einer Stelle.

## Warum Step 1 „Emitter-Schritt“ heißt

In Step 1 geht es nur darum, dass aus `<<`/`>>` jetzt die richtigen Bits erzeugt werden.  
Decoder/Execution kommen erst danach.

