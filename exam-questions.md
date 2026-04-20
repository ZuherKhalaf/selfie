# Exam Questions

## Frage 4 (Main Question)

„Wie würde ich am Beispiel von `<<` und `>>` erklären, was in Assignment 3 (Frontend) gemacht wurde und was in Assignment 4 (Backend/Execution) zusätzlich nötig war, damit die Operationen nicht nur geparst, sondern auch korrekt als `sll`/`srl` ausgeführt werden?“

## Subfragen zu Step 1 (Code Generation für `sll`/`srl`)

1. Was bedeutet bei RISC-V der gemeinsame `opcode` `0110011` (`OP_OP`) für Register-Register-Instruktionen?
2. Warum reichen `opcode`-Bits alleine nicht aus, um `add`, `sub`, `mul`, `divu`, `sll`, `srl` usw. zu unterscheiden?
3. Welche Rolle haben `funct3` und `funct7` im R-Format genau?
4. Warum ist `F3_SLL = 1` (binär `001`) und `F3_SRL = 5` (binär `101`)?
5. Warum sind `F7_SLL` und `F7_SRL` beide `0` (binär `0000000`)?
6. Warum muss `divu` im Decoder trotz gleichem `funct3` wie `srl` über `funct7` getrennt werden?
7. Wie baut `encode_r_format(funct7, rs2, rs1, funct3, rd, opcode)` die 32 Bits schrittweise auf?
8. Warum bekommt `encode_r_format` die Argument-Reihenfolge `(funct7, rs2, rs1, funct3, rd, opcode)`?
9. Was ist die Aufgabe eines „Emitters“ im Compiler (`emit_add`, `emit_sll`, `emit_srl`)?
10. Warum ruft `compile_shift()` nicht direkt `encode_r_format`, sondern `emit_sll`/`emit_srl` auf?
11. Was ist der Unterschied zwischen „Instruktion erzeugen“ (Emitter) und „Instruktion ausführen“ (`do_*` im Emulator)?
12. Wie erkenne ich im erzeugten Assembler/Binary, dass wirklich `sll`/`srl` statt Platzhalter erzeugt wurden?
13. Welche minimalen Änderungen in Step 1 sind nötig, damit Codegen korrekt ist, auch wenn Decoder/Execution noch fehlen?
14. Welche Fehlerbilder zeigen, dass Step 1 korrekt ist, aber Step 2/3 noch fehlt?
15. Wie würde ich den End-to-End-Pfad in einem Satz erklären: `<<` im Quellcode -> `emit_sll` -> RISC-U-Binärinstruktion?
16. Wie erkennt das System über `opcode = 51` (`OP_OP`, binär `0110011`), dass die Instruktion im R-Format zu dekodieren ist, und welche Schritte passieren danach bis zur Auswahl von `SLL`/`SRL`?
17. Warum reicht ein 7-Bit-`opcode` (max. 128 Muster) in RISC-V nicht aus, um alle Instruktionen eindeutig zu kodieren, und wie lösen `funct3` und `funct7` dieses Problem?
18. Wie lese ich die R-Format-Tabelle (Bitpositionen 31..0) korrekt, und wie ordne ich daraus bei einer 32-Bit-Instruktion die Felder `funct7`, `rs2`, `rs1`, `funct3`, `rd`, `opcode` praktisch zu?
19. Warum entspricht in den Assertions die Bitanzahl direkt der Zweierpotenz im Wertebereich (z. B. `0 <= funct7 < 2^7`, `0 <= rs1 < 2^5`), und wie hängt das mit der Anzahl darstellbarer Muster pro Feld zusammen?
