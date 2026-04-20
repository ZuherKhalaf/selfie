# Execution Flow: Was wird WANN ausgeführt?

## WICHTIG: Nicht alles wird linear ausgeführt!

Code-Zeilen 20-79 sind **FUNKTIONEN** die nur bei Bedarf aufgerufen werden.

---

## AUSFÜHRUNGS-REIHENFOLGE

### SCHRITT 1: Zeilen 1-13 (Initialisierung) - WIRD AUSGEFÜHRT
```
0x0:  lui t0,0x11          // t0 = 0x11000
0x4:  addi t0,t0,8         // t0 = 0x11008  
0x8:  addi gp,t0,0         // gp = 0x11008 (Global Pointer setzen)
```
**Was:** Global Pointer (gp) einrichten
**Warum:** gp zeigt auf Daten-Segment (globale Variablen)

```
0xC:  addi a0,zero,0       // a0 = 0
0x10: addi a7,zero,214     // a7 = 214 (brk syscall)
0x14: ecall                // System Call: brk(0) → gibt Heap-Start zurück
```
**Was:** Frage OS "Wo ist mein Heap?"
**Return:** a0 enthält jetzt Heap-Start-Adresse

```
0x18: addi a0,a0,7         // a0 += 7
0x1C: addi t0,zero,8       // t0 = 8
0x20: remu t0,a0,t0        // t0 = a0 % 8
0x24: sub a0,a0,t0         // a0 = (a0 + 7) - (a0 % 8)  → 8-Byte aligned
0x28: addi a7,zero,214     // a7 = 214
0x2C: ecall                // brk(aligned_address)
0x30: sd a0,-8(gp)         // Speichere Heap-Start bei gp-8
```
**Was:** Heap-Adresse auf 8-Byte Grenze ausrichten und speichern
**Warum:** CPU-Effizienz (aligned memory access)

---

### SCHRITT 2: Zeilen 14-19 (Bootstrap Frame) - WIRD AUSGEFÜHRT
```
0x34: ld t0,0(sp)          // Lade Wert vom Stack
0x38: addi sp,sp,-8        // sp -= 8 (Platz schaffen)
0x3C: sd t0,0(sp)          // Speichere zurück
0x40: addi t0,sp,16        // t0 = sp + 16
0x44: sd t0,8(sp)          // Speichere bei sp+8
0x48: jal ra,88[0x13C]     // ⭐ SPRINGE ZU main() bei 0x13C ⭐
```
**Was:** Stack-Frame für main() vorbereiten
**WICHTIG:** Bei **0x48** springt das Programm zu **main()** (Zeile 80)!

---

### SCHRITT 3: main() läuft (Zeilen 80-99) - WIRD AUSGEFÜHRT

Deine main() Funktion läuft jetzt. Bei Ende macht sie `jalr zero,0(ra)`.
Das springt **ZURÜCK** zu Adresse in **ra** (gesetzt bei 0x48).

**ra** enthält: **0x4C** (die Adresse nach dem `jal` Befehl)

---

### SCHRITT 4: Nach main() Return (Zeilen 20-25) - WIRD AUSGEFÜHRT
```
0x4C: addi sp,sp,-8        // Stack anpassen
0x50: sd a0,0(sp)          // Return value von main() sichern
0x54: ld a0,0(sp)          // Wieder laden
0x58: addi sp,sp,8         // Stack cleanup
0x5C: addi a7,zero,93      // a7 = 93 (exit syscall)
0x60: ecall                // exit(return_value) → Programm beenden!
```
**Was:** Beende Programm mit Return-Wert von main()
**PROGRAMM ENDET HIER!**

---

## Zeilen 26-79: Werden NICHT automatisch ausgeführt!

Das sind **FUNKTIONEN** (syscall wrapper):

```
Zeile 26-34 (0x64-0x84):   read() Funktion
Zeile 35-43 (0x88-0xA8):   write() Funktion  
Zeile 44-53 (0xAC-0xD0):   openat() Funktion
Zeile 54-71 (0xD4-0x118):  brk()/malloc() Funktion
Zeile 72-79 (0x11C-0x138): procedure_prologue Funktion
```

**Diese werden NUR ausgeführt wenn:**
- Dein Code `printf()` aufruft → springt zu write()
- Dein Code `malloc()` aufruft → springt zu brk()
- Dein Code Dateien öffnet → springt zu openat()

**In deinem simple.c werden sie NIE aufgerufen!**

---

## VISUALISIERUNG

```
START
  │
  ├─ Zeile 1-13:   Initialisierung (LÄUFT)
  │
  ├─ Zeile 14-19:  Bootstrap (LÄUFT)
  │
  └─ Zeile 19:     jal zu main() ──────┐
                                        │
     Zeile 20-25:  exit() Code          │
        (wartet auf Return)             │
                                        │
     Zeilen 26-79: Funktionen           │
        (werden NIE aufgerufen          │
         in simple.c)                   │
                                        ▼
     Zeile 80-99: main() ◄──── SPRINGT HIERHER
        x = 7 + 3;                      │
        return x;                       │
        jalr zero,0(ra) ────────────────┘
                                        │
                                        ▼
     Zeile 20-25: exit(10) ◄──── SPRINGT ZURÜCK
        ecall
        
ENDE
```

---

## ZUSAMMENFASSUNG

**Linear ausgeführt:**
1. Zeile 1-13 (Init)
2. Zeile 14-19 (Bootstrap)
3. **SPRUNG zu Zeile 80** (main)
4. Zeile 80-99 (main läuft)
5. **SPRUNG zurück zu Zeile 20**
6. Zeile 20-25 (exit)
7. PROGRAMM ENDET

**NIE ausgeführt (in simple.c):**
- Zeilen 26-79 (syscall wrapper Funktionen)

**Wann würden 26-79 ausgeführt?**
Nur wenn dein Code eine dieser Funktionen aufruft!

Beispiel:
```c
int main() {
    printf("Hello");  // → würde write() bei Zeile 35 aufrufen
    return 0;
}
```

---

## Technischer Begriff

Zeilen 26-79 sind die **C-Runtime Library** (minimal version).
Sie existieren im Code, werden aber nur bei Bedarf "angesprungen".

Das ist wie Funktionen in einer Bibliothek:
- Sie sind im Code vorhanden
- Aber werden nur ausgeführt wenn du sie aufrufst
- **In simple.c: kein Aufruf → keine Ausführung**
