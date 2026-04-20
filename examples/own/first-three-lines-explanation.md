# Die ersten 3 Zeilen - Immer am Anfang!

## Antwort: JA, fast immer gleich (mit kleiner Variation)

---

## ZEILE 1-3: Global Pointer Setup

### Programme OHNE globale Variablen:

**empty.c, simple.c, count.c:**
```assembly
0x0(~1): 0x000112B7: lui t0,0x11        // Zeile 1
0x4(~1): 0x00828293: addi t0,t0,8      // Zeile 2
0x8(~1): 0x00028193: addi gp,t0,0      // Zeile 3
```

### Programme MIT globalen Variablen:

**larger.c (2 globale int Variablen):**
```assembly
0x0(~1): 0x000112B7: lui t0,0x11        // Zeile 1 - GLEICH
0x4(~1): 0x01828293: addi t0,t0,24     // Zeile 2 - ANDERS! (24 statt 8)
0x8(~1): 0x00028193: addi gp,t0,0      // Zeile 3 - GLEICH
```

**Unterschied:** Nur die **Zeile 2** ändert sich (8 vs 24)!

---

## ZEILE-FÜR-ZEILE ERKLÄRUNG

### Zeile 1: `lui t0,0x11`

**Was:** Load Upper Immediate - Lade obere Bits
**Formel:** `t0 = 0x11 * 2^12 = 0x11 * 4096 = 0x11000`
**Warum:** Basis-Adresse für Daten-Segment setzen
**Immer gleich:** ✅ JA

---

### Zeile 2: `addi t0,t0,X` (X variiert!)

**Was:** Add Immediate - Addiere Offset
**empty/simple/count:**
```
addi t0,t0,8     → t0 = 0x11000 + 8 = 0x11008
```

**larger (2 globale Variablen):**
```
addi t0,t0,24    → t0 = 0x11000 + 24 = 0x11018
```

**Warum der Unterschied?**
- Jede globale Variable = 8 Bytes (64-bit int)
- 2 Variablen = 2 × 8 = 16 Bytes
- Plus 8 Bytes Base = 24 Bytes

**Formel:**
```
Offset = 8 + (Anzahl_Globale_Variablen × 8)
```

**Immer gleich:** ❌ NEIN (hängt von globalen Variablen ab)

---

### Zeile 3: `addi gp,t0,0`

**Was:** Kopiere t0 nach gp
**Formel:** `gp = t0 + 0 = t0`
**Warum:** gp (Global Pointer) zeigt jetzt auf Daten-Segment
**Immer gleich:** ✅ JA

---

## WARUM DIESE 3 ZEILEN?

**Zweck:** Global Pointer (gp) Register initialisieren

**gp zeigt auf:**
- Globale Variablen
- String-Literale  
- Konstante Daten
- Heap-Start Verwaltung

---

## BEISPIEL-BERECHNUNG

### Programm ohne globale Variablen:
```
Zeile 1: t0 = 0x11000
Zeile 2: t0 = 0x11000 + 8 = 0x11008
Zeile 3: gp = 0x11008
```

**Memory Layout:**
```
0x11000: [8 Bytes für Heap-Verwaltung]
0x11008: ← gp zeigt hierher (Daten-Segment Start)
```

### Programm mit 2 globalen int:
```
Zeile 1: t0 = 0x11000
Zeile 2: t0 = 0x11000 + 24 = 0x11018
Zeile 3: gp = 0x11018
```

**Memory Layout:**
```
0x11000: [8 Bytes für Heap-Verwaltung]
0x11008: [global_var - 8 Bytes]
0x11010: [another - 8 Bytes]
0x11018: ← gp zeigt hierher (nach den Variablen)
```

---

## RISCU INSTRUKTIONEN ERKLÄRT

### `lui rd,imm`
**Name:** Load Upper Immediate
**Syntax:** `lui t0,0x11`
**Bedeutung:** `rd = imm × 2^12`
**Warum × 4096?** Um große Adressen zu laden (20 Bit immediate)

**Beispiel:**
```
lui t0,0x11
→ t0 = 0x11 × 4096 = 0x11000
```

### `addi rd,rs1,imm`
**Name:** Add Immediate
**Syntax:** `addi t0,t0,8`
**Bedeutung:** `rd = rs1 + imm`

**Beispiel:**
```
addi t0,t0,8
→ t0 = t0 + 8
```

---

## ZUSAMMENFASSUNG

**Zeile 1:** ✅ IMMER GLEICH - Basis 0x11000
**Zeile 2:** ⚠️ VARIIERT - abhängig von globalen Variablen
**Zeile 3:** ✅ IMMER GLEICH - gp = t0

**Diese 3 Zeilen sind das ERSTE was JEDES selfie-Programm macht!**

**Zweck:** Richte Global Pointer ein, damit das Programm auf Daten zugreifen kann.

---

## TESTE SELBST

Erstelle ein Programm mit vielen globalen Variablen:

```c
int a;
int b;
int c;
int d;
int e;

int main() {
  return 0;
}
```

**Vorhersage für Zeile 2:**
```
5 Variablen × 8 Bytes = 40 Bytes
40 + 8 = 48 Bytes
→ addi t0,t0,48
```

Probier es aus!
