# RISC-U Assembly Erklärung: simple.c

## C-Code
```c
int main() {        // Zeile 1
    int x;          // Zeile 2
    x = 7 + 3;      // Zeile 3
    return x;       // Zeile 4
}                   // Zeile 5
```

---

## Assembly Code - ZEILE FÜR ZEILE

Die main() Funktion beginnt bei Adresse **0x13C** (Zeile 80 in simple.S)

---

### PROLOG (Setup) - Zeilen 80-85

**Zeile 80:** `0x13C(~3): addi sp,sp,-8`
- **Was:** Stack Pointer um 8 Bytes verringern
- **Warum:** Platz für Return Address schaffen
- **sp** zeigt jetzt auf freien Speicher

**Zeile 81:** `0x140(~3): sd ra,0(sp)`
- **Was:** Return Address auf Stack speichern
- **Warum:** Damit wir wissen, wohin wir zurückkehren müssen
- **ra** = wohin springen nach `main()`

**Zeile 82:** `0x144(~3): addi sp,sp,-8`
- **Was:** Nochmal 8 Bytes Platz schaffen
- **Warum:** Für Frame Pointer

**Zeile 83:** `0x148(~3): sd s0,0(sp)`
- **Was:** Frame Pointer (s0) auf Stack speichern
- **Warum:** Alten Wert sichern

**Zeile 84:** `0x14C(~3): addi s0,sp,0`
- **Was:** Frame Pointer = Stack Pointer
- **Warum:** s0 zeigt jetzt auf Anfang unseres Stack-Frames

**Zeile 85:** `0x150(~3): addi sp,sp,-8`
- **Was:** Nochmal 8 Bytes für lokale Variable
- **Warum:** Platz für `int x`

---

### DEIN CODE (x = 7 + 3) - Zeilen 86-89

**Zeile 86:** `0x154(~3): addi t0,zero,7`
- **Was:** `t0 = 0 + 7` → **t0 = 7**
- **C-Code:** Die `7` aus `x = 7 + 3`

**Zeile 87:** `0x158(~3): addi t1,zero,3`
- **Was:** `t1 = 0 + 3` → **t1 = 3**
- **C-Code:** Die `3` aus `x = 7 + 3`

**Zeile 88:** `0x15C(~3): add t0,t0,t1`
- **Was:** `t0 = t0 + t1` → **t0 = 7 + 3 = 10**
- **C-Code:** Das `+` aus `x = 7 + 3`

**Zeile 89:** `0x160(~3): sd t0,-8(s0)`
- **Was:** Speichere t0 bei Adresse `s0 - 8`
- **C-Code:** Das `x =` → Variable x bekommt Wert 10
- **-8(s0)** ist die Position von `x` auf dem Stack

---

### RETURN - Zeilen 90-92

**Zeile 90:** `0x164(~4): ld t0,-8(s0)`
- **Was:** Lade Wert von `x` (bei s0-8) in t0
- **C-Code:** `return x;` → x holen

**Zeile 91:** `0x168(~4): addi a0,t0,0`
- **Was:** `a0 = t0 + 0` → **a0 = 10**
- **Warum:** Return-Wert muss in **a0** sein (Konvention!)

**Zeile 92:** `0x16C(~4): jal zero,2[0x174]`
- **Was:** Springe zu Adresse 0x174 (Epilog)
- **Warum:** Überspringe tote Code-Zeile 93

---

### EPILOG (Cleanup) - Zeilen 94-99

**Zeile 94:** `0x174(~5): addi sp,s0,0`
- **Was:** Stack Pointer = Frame Pointer
- **Warum:** Alle lokalen Variablen "löschen"

**Zeile 95:** `0x178(~5): ld s0,0(sp)`
- **Was:** Alten Frame Pointer vom Stack laden

**Zeile 96:** `0x17C(~5): addi sp,sp,8`
- **Was:** Stack um 8 Bytes erhöhen

**Zeile 97:** `0x180(~5): ld ra,0(sp)`
- **Was:** Return Address vom Stack laden

**Zeile 98:** `0x184(~5): addi sp,sp,8`
- **Was:** Stack nochmal um 8 erhöhen

**Zeile 99:** `0x188(~5): jalr zero,0(ra)`
- **Was:** Springe zu Adresse in **ra**
- **Warum:** **RETURN aus der Funktion!**

---

## Wichtige Register

- **sp**: Stack Pointer (zeigt auf Top des Stacks)
- **ra**: Return Address (wohin zurückkehren)
- **s0**: Frame Pointer (zeigt auf Stack-Frame Anfang)
- **t0, t1**: Temporäre Register (für Berechnungen)
- **a0**: Argument & Return Value Register
- **zero**: Immer 0

---

## Stack Visualisierung

```
Vor main():
sp → [höhere Adresse]

Nach Prolog (Zeile 85):
      [Return Address (ra)]  ← sp+16
      [Alter Frame Pointer]  ← sp+8, s0+8
s0 →  [Frame Start]          ← sp
      [lokale Variable x]    ← sp-8, s0-8
```

---

## Übungsfragen

1. Wo auf dem Stack wird `x` gespeichert?
   → Bei s0-8

2. In welchem Register ist der Return-Wert?
   → In a0

3. Warum wird sp **verringert** statt erhöht beim Prolog?
   → Der Stack wächst nach unten (zu niedrigeren Adressen)

---

## RISC-U Instruktionen - Kurz-Referenz

**Arithmetic:**
- `add rd,rs1,rs2`: rd = rs1 + rs2
- `addi rd,rs1,imm`: rd = rs1 + imm (immediate)

**Memory:**
- `ld rd,imm(rs1)`: rd = memory[rs1 + imm] (load)
- `sd rs2,imm(rs1)`: memory[rs1 + imm] = rs2 (store)

**Control:**
- `jal rd,imm`: rd = pc + 4; pc = pc + imm (jump and link)
- `jalr rd,imm(rs1)`: rd = pc + 4; pc = rs1 + imm (jump and link register)

**Initialization:**
- `lui rd,imm`: rd = imm * 2^12 (load upper immediate)
