# Speicher zur Laufzeit inspizieren

## Antwort: Benutze den `-d` (Debugger) Flag!

---

## METHODE: Debugger Mode

```bash
./selfie -c examples/own/simple.c -d 1
```

**Was passiert:**
- Jede Instruktion wird einzeln angezeigt
- Du siehst Register-Werte VORHER und NACHHER
- Du siehst Speicher-Zugriffe (Memory reads/writes)

---

## ZEILE 1-3 IM DEBUGGER

### Zeile 1: lui t0,0x11
```
pc==0x10000(~1): lui t0,0x11: |- t0==0x0 -> t0==0x11000
```

**Lesen:**
- `pc==0x10000`: Program Counter bei Adresse 0x10000
- `t0==0x0`: t0 war 0
- `t0==0x11000`: t0 ist jetzt 0x11000 (69632 dezimal)

### Zeile 2: addi t0,t0,8
```
pc==0x10004(~1): addi t0,t0,8: t0==69632(0x11000) |- t0==69632(0x11000) -> t0==69640(0x11008)
```

**Lesen:**
- `t0==69632`: t0 enthielt 0x11000
- `t0==69640`: t0 ist jetzt 0x11008

### Zeile 3: addi gp,t0,0
```
pc==0x10008(~1): addi gp,t0,0: t0==69640(0x11008) |- gp==0x0 -> gp==0x11008
```

**Lesen:**
- `gp==0x0`: gp war 0
- `gp==0x11008`: gp ist jetzt 0x11008

---

## SPEICHER-ZUGRIFFE SEHEN

### Beispiel: Heap-Start wird gespeichert
```
pc==0x10030(~1): sd a0,-8(gp): gp==0x11008,a0==73728(0x12000) |- mem[0x11000]==0 -> mem[0x11000]==a0==73728(0x12000)
```

**Lesen:**
- `sd a0,-8(gp)`: Store a0 bei Adresse (gp-8)
- `gp==0x11008`: gp = 0x11008
- `-8(gp)`: 0x11008 - 8 = **0x11000**
- `mem[0x11000]==0`: Adresse 0x11000 enthielt 0
- `mem[0x11000]==73728`: Adresse 0x11000 enthält jetzt 73728 (0x12000)

**Was wurde gespeichert?**
Bei Adresse **0x11000** steht jetzt **0x12000** (Heap-Start!)

### Beispiel: Variable x wird gespeichert
```
pc==0x10160(~3): sd t0,-8(s0): s0==0xFFFFFFB0,t0==10(0xA) |- mem[0xFFFFFFA8]==0 -> mem[0xFFFFFFA8]==t0==10(0xA)
```

**Lesen:**
- Bei Adresse **0xFFFFFFA8** steht jetzt **10** (das ist x = 7 + 3!)

---

## DEINE BERECHNUNG x = 7 + 3 IM DEBUGGER

```
pc==0x10154(~3): addi t0,zero,7: |- t0==... -> t0==7(0x7)
```
**t0 = 7**

```
pc==0x10158(~3): addi t1,zero,3: |- t1==0(0x0) -> t1==3(0x3)
```
**t1 = 3**

```
pc==0x1015C(~3): add t0,t0,t1: t0==7(0x7),t1==3(0x3) |- t0==7(0x7) -> t0==10(0xA)
```
**t0 = 7 + 3 = 10**

```
pc==0x10160(~3): sd t0,-8(s0): ... |- mem[0xFFFFFFA8]==0 -> mem[0xFFFFFFA8]==t0==10(0xA)
```
**Speichere 10 im Speicher (Variable x)**

---

## DEBUGGER FORMAT VERSTEHEN

```
pc==ADRESSE(~ZEILE): INSTRUKTION: INPUTS |- VORHER -> NACHHER
```

**Beispiel:**
```
pc==0x1015C(~3): add t0,t0,t1: t0==7(0x7),t1==3(0x3) |- t0==7(0x7) -> t0==10(0xA)
```

- `pc==0x1015C`: Program Counter bei 0x1015C
- `(~3)`: Das ist C-Code Zeile 3
- `add t0,t0,t1`: Die Instruktion
- `t0==7,t1==3`: Input-Werte
- `|-`: Separator
- `t0==7`: Alter Wert von t0
- `->`: Wird zu
- `t0==10`: Neuer Wert von t0

---

## SPEICHER-BEREICHE

Der Debugger zeigt dir verschiedene Speicher-Bereiche:

### Code Segment
```
0x10000 - 0x10xxx: Dein Programm-Code (Instruktionen)
```

### Data Segment  
```
0x11000: Heap-Verwaltung (Heap-Start Pointer)
0x11008: Global Pointer Base (gp)
```

### Heap
```
0x12000: Beginn des Heaps (für malloc)
```

### Stack
```
0xFFFFFFxx: Stack (wächst nach unten!)
```

---

## SPEICHER AN BESTIMMTER ADRESSE SEHEN

**Im Debugger Output suchen:**

```bash
./selfie -c simple.c -d 1 2>&1 | grep "mem\[0x11000\]"
```

**Output:**
```
mem[0x11000]==0 -> mem[0x11000]==a0==73728(0x12000)
```

**Bedeutung:** An Adresse 0x11000 steht 0x12000!

---

## ANDERE DEBUGGER-OPTIONEN

```bash
# Mit mehr Speicher (z.B. 4MB)
./selfie -c simple.c -d 4

# Nur main() ausführen (weniger Output)
./selfie -c simple.c -d 1 2>&1 | grep "~3\|~4\|~5"
```

---

## WAS DU IN 0x11000 FINDEST

Basierend auf dem Debugger:

**Adresse 0x11000:**
```
Vor Zeile 13: mem[0x11000] = 0
Nach Zeile 13: mem[0x11000] = 0x12000 (Heap-Start)
```

**Das ist der HEAP-POINTER!**

---

## ZUSAMMENFASSUNG

**Frage:** Wie sehe ich was in 0x11000 steht?

**Antwort:** 
```bash
./selfie -c dein_programm.c -d 1
```

Im Output nach `mem[0x11000]` suchen!

**Was steht da:**
- Heap-Start Adresse (für malloc)
- Wird beim Programm-Start gesetzt
- In simple.c: **0x12000**

---

## PRAKTISCHE ÜBUNG

Führe aus:
```bash
./selfie -c examples/own/simple.c -d 1 | grep "mem\[0x"
```

Das zeigt dir ALLE Memory-Operationen mit Adressen!
