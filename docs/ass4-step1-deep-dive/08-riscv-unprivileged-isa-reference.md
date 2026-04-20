# 08 — RISC-V Unprivileged ISA (RV64I/M) Reference Notes

Diese Notiz verbindet Selfie mit der offiziellen ISA-Quelle.

## Was du im RISC-V Dokument suchen solltest

Suchbegriffe:

- `Integer Register-Register Operations`
- `R-type format`
- `opcode 0110011`
- `sll`, `srl`, `divu`
- `funct3`, `funct7`

## Warum das wichtig ist

In Selfie sind Werte wie:

- `OP_OP = 51` (binär `0110011`)
- `F3_SLL = 1` (binär `001`)
- `F3_SRL = 5` (binär `101`)
- `F7_SLL = 0`, `F7_SRL = 0`, `F7_DIVU = 1`

direkt aus dieser ISA-Kodierung abgeleitet.  
Sie sind daher **Spezifikationswerte**, keine Designentscheidung im Assignment.

## Verbindung zu deiner Prüfungserklärung

Wenn du gefragt wirst „Warum genau diese Zahlen?“:

1. Sie kommen aus der RISC-V Kodierungstabelle für R-Format.
2. Selfie übernimmt diese Bitmuster in `F3_*` und `F7_*`.
3. `encode_r_format` setzt die Felder an die korrekten Bitpositionen.
4. `decode()` liest sie zurück und wählt `is = SLL/SRL/DIVU`.

## Typische Prüfungsfalle

`srl` und `divu` wirken wie „gleiches `funct3`“, aber sind über `funct7` verschieden.  
Genau deshalb muss Decoder-Logik mehrstufig sein.

