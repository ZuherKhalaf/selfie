# 01 — `riscu.md`: Was die Shift-Instruktionen bedeuten

Quelle: `riscu.md` (u. a. Zeilen 7, 17, 47-49)

## Wichtige Zeilen und Interpretation

### `RISC-U consists of just 16 instructions ...`
- Bedeutung: `sll` und `srl` sind jetzt Teil des offiziellen RISC-U-Satzes.
- Prüfungspunkt: Ohne diese Doku ist Assignment 4 formal unvollständig.

### `RISC-U instructions are encoded in 32 bits`
- Bedeutung: Jede Instruktion ist ein 32-Bit-Wort.
- Konsequenz: `sll`/`srl` müssen in dieses feste Format (R-Format) passen.

### `sll rd,rs1,rs2: rd = rs1 << rs2; pc = pc + 4`
- Semantik: logischer Left-Shift.
- `rd`: Zielregister, `rs1`: Wert, `rs2`: Shift-Menge.
- `pc + 4`: normale sequenzielle Ausführung.

### `srl rd,rs1,rs2: rd = rs1 >> rs2; pc = pc + 4`
- Semantik: logischer Right-Shift.
- In Selfie wird hier die unsigned/rechte Schiebe-Semantik modelliert.

## Warum relevant für Step 1?

Step 1 erzeugt die Instruktion nur.  
Damit du korrekt erzeugen kannst, musst du vorher wissen, **welche Semantik** getroffen werden soll.

