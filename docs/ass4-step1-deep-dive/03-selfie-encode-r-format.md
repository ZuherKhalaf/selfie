# 03 — `encode_r_format`: Wie die 32 Bits gebaut werden

Quelle: `selfie.c` um Zeilen ~6629-6647

## Originalstruktur

```c
// | funct7 | rs2 | rs1 | funct3 | rd | opcode |
uint64_t encode_r_format(uint64_t funct7, uint64_t rs2, uint64_t rs1,
                         uint64_t funct3, uint64_t rd, uint64_t opcode) {
  return left_shift(
           left_shift(
             left_shift(
               left_shift(
                 left_shift(funct7, 5) + rs2, 5) + rs1, 3) + funct3, 5) + rd,
           7) + opcode;
}
```

## Schrittweise Interpretation

1. `left_shift(funct7, 5) + rs2`
   - macht aus `[funct7]` -> `[funct7|rs2]`
2. wieder `<< 5` und `+ rs1`
   - `[funct7|rs2|rs1]`
3. `<< 3` und `+ funct3`
   - `[funct7|rs2|rs1|funct3]`
4. `<< 5` und `+ rd`
   - `[funct7|rs2|rs1|funct3|rd]`
5. `<< 7` und `+ opcode`
   - `[funct7|rs2|rs1|funct3|rd|opcode]` = vollständige 32-Bit Instruktion

## Warum diese Parameter-Reihenfolge?

Sie folgt genau der Feldreihenfolge im R-Format von links (MSB) nach rechts (LSB).  
Dadurch bleibt `encode_r_format` für alle R-Format-Instruktionen wiederverwendbar.

