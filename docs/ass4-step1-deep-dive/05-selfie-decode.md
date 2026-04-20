# 05 — `decode()`: Von Bits zurück zu Instruktions-ID

Quelle: `selfie.c` um Zeilen ~9972-10023

## Kernidee

`decode()` nimmt `ir` (instruction register) und setzt `is` (instruction selector).  
Das ist der Schritt zwischen „Bits“ und „welche `do_*`-Funktion läuft“.

## Shift-relevante Stelle

```c
} else if (opcode == OP_OP) {
  decode_r_format();

  if (funct3 == F3_ADD) { ... }
  else if (funct3 == F3_REMU) { ... }
  else if (funct3 == F3_SLTU) { ... }
  else if (funct3 == F3_SLL) {
    if (funct7 == F7_SLL)
      is = SLL;
  } else if (funct3 == F3_SRL) {
    if (funct7 == F7_SRL)
      is = SRL;
    else if (funct7 == F7_DIVU)
      is = DIVU;
  }
}
```

## Line-by-line Interpretation

1. `opcode == OP_OP`
   - Nur R-Format-Recheninstruktionen werden hier behandelt.
2. `decode_r_format()`
   - extrahiert `funct7`, `rs2`, `rs1`, `funct3`, `rd`.
3. `funct3 == F3_SLL`
   - Kandidat für `sll`; Bestätigung über `funct7`.
4. `funct3 == F3_SRL`
   - Kandidat für `srl/divu`; Trennung über `funct7`.
5. `is = SLL/SRL/DIVU`
   - Mapping auf interne Instruktions-ID für den Ausführungsschritt.

## Typischer Fehler

`funct3 == F3_DIVU` separat vor `F3_SRL` prüfen -> `srl` fällt durch.  
Grund: `srl` und `divu` teilen `funct3=101`.

