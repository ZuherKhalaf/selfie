# Alle Ausführungs-Optionen in selfie

## ÜBERSICHT

```bash
./selfie { -c source | -l binary } [ -m | -d | -r | -y ] RAM_in_MB
```

**Ausführungs-Modi:**
- `-m` = mipster (Standard-Emulator)
- `-d` = debugger (mit Trace)
- `-r` = riscu (Minimaler Emulator)
- `-y` = hypster (Hypervisor/Virtualisierung)

---

## METHODE 1: Direkt kompilieren + ausführen

### Standard Ausführung (-m)
```bash
./selfie -c examples/own/simple.c -m 1
```
**Was:** Kompiliert + führt mit mipster aus (1 MB RAM)
**Ausgabe:** Nur Exit Code und Statistiken

### Mit Debugger (-d)
```bash
./selfie -c examples/own/simple.c -d 1
```
**Was:** Kompiliert + führt mit Debugger aus
**Ausgabe:** JEDE INSTRUKTION wird angezeigt

### Mit RISC-U Emulator (-r)
```bash
./selfie -c examples/own/simple.c -r 1
```
**Was:** Kompiliert + führt mit minimalem Emulator aus
**Ausgabe:** Wie -m, aber minimaler

### Mit Hypster (-y)
```bash
./selfie -c examples/own/simple.c -y 1
```
**Was:** Kompiliert + führt in virtualisierter Umgebung
**Ausgabe:** Hypervisor-Modus (fortgeschritten)

---

## METHODE 2: Erst kompilieren, dann laden + ausführen

### Schritt 1: Binary erstellen
```bash
./selfie -c examples/own/simple.c -o examples/own/simple.m
```

### Schritt 2a: Mit mipster ausführen
```bash
./selfie -l examples/own/simple.m -m 1
```

### Schritt 2b: Mit Debugger ausführen
```bash
./selfie -l examples/own/simple.m -d 1
```

### Schritt 2c: Mit RISC-U Emulator
```bash
./selfie -l examples/own/simple.m -r 1
```

### Schritt 2d: Mit Hypster
```bash
./selfie -l examples/own/simple.m -y 1
```

---

## RAM-GRÖSSE ÄNDERN

**Format:** Zahl von 0 bis 4096 (in MB)

```bash
./selfie -c simple.c -m 1      # 1 MB RAM
./selfie -c simple.c -m 2      # 2 MB RAM
./selfie -c simple.c -m 16     # 16 MB RAM
./selfie -c simple.c -m 128    # 128 MB RAM
./selfie -c simple.c -m 4096   # 4 GB RAM (Maximum!)
```

**Wann mehr RAM?**
- Große malloc() Aufrufe
- Viele Daten
- Rekursion (tiefer Stack)

---

## UNTERSCHIEDE DER MODI

### -m (mipster) - Standard
```bash
./selfie -c simple.c -m 1
```
**Eigenschaften:**
- ✅ Vollständiger RISC-U Emulator
- ✅ Syscalls unterstützt
- ✅ Statistiken am Ende
- ❌ Kein Trace/Debug-Output

**Ausgabe:**
```
./selfie: 64-bit mipster executing ...
./selfie: mipster terminating ... with exit code 10
./selfie: summary: 44 executed instructions
```

---

### -d (debugger) - Mit Trace
```bash
./selfie -c simple.c -d 1
```
**Eigenschaften:**
- ✅ Wie mipster
- ✅ Zeigt JEDE Instruktion
- ✅ Zeigt Register-Änderungen
- ✅ Zeigt Memory-Zugriffe

**Ausgabe:**
```
pc==0x10154(~3): addi t0,zero,7: |- t0==0 -> t0==7
pc==0x10158(~3): addi t1,zero,3: |- t1==0 -> t1==3
pc==0x1015C(~3): add t0,t0,t1: t0==7,t1==3 |- t0==7 -> t0==10
```

---

### -r (riscu) - Minimal
```bash
./selfie -c simple.c -r 1
```
**Eigenschaften:**
- ✅ Minimaler Emulator
- ✅ Nur RISC-U Instruktionen
- ⚠️ Weniger Features als mipster

**Wann benutzen:**
- Verstehen wie ein minimaler Emulator funktioniert
- Lernzwecke

---

### -y (hypster) - Virtualisierung
```bash
./selfie -c simple.c -y 1
```
**Eigenschaften:**
- ✅ Hypervisor-Modus
- ✅ Kann mipster IN mipster laufen lassen
- ⚠️ Fortgeschritten!

**Wann benutzen:**
- Virtualisierung studieren
- Verschachtelte Ausführung
- Fortgeschrittene Konzepte

---

## KOMBINATIONEN

### Kompilieren, Assembly speichern, ausführen:
```bash
./selfie -c simple.c -S simple.S -m 1
```

### Binary speichern UND ausführen:
```bash
./selfie -c simple.c -o simple.m -m 1
```

### Kompilieren + Assembly + Binary + Ausführen:
```bash
./selfie -c simple.c -S simple.S -o simple.m -m 1
```

### Mehrere Programme nacheinander:
```bash
./selfie -c prog1.c -m 1 -c prog2.c -m 1
```

---

## EXIT CODE SEHEN

### In Bash:
```bash
./selfie -c simple.c -m 1
echo $?
```
**Ausgabe:** `10` (weil simple.c `return x;` mit x=10 hat)

### Exit Code direkt anzeigen:
```bash
./selfie -c simple.c -m 1 && echo "Exit: $?"
```

---

## OUTPUT FILTERN

### Nur Exit Code:
```bash
./selfie -c simple.c -m 1 2>&1 | grep "exit code"
```

### Nur Statistiken:
```bash
./selfie -c simple.c -m 1 2>&1 | grep -A 10 "summary:"
```

### Debugger nur main():
```bash
./selfie -c simple.c -d 1 2>&1 | grep "~3\|~4\|~5"
```

---

## PRAKTISCHE BEISPIELE

### 1. Schnell testen:
```bash
./selfie -c test.c -m 1
```

### 2. Debuggen:
```bash
./selfie -c test.c -d 1 | less
```

### 3. Binary für später:
```bash
./selfie -c test.c -o test.m
# Später:
./selfie -l test.m -m 1
```

### 4. Performance messen:
```bash
time ./selfie -c benchmark.c -m 16
```

### 5. Memory-Zugriffe checken:
```bash
./selfie -c prog.c -d 1 2>&1 | grep "mem\["
```

---

## HÄUFIGE FEHLER

### 1. Zu wenig RAM
```bash
./selfie -c big_program.c -m 1
# Error: out of memory
```
**Lösung:** Mehr RAM geben
```bash
./selfie -c big_program.c -m 16
```

### 2. Falsche Datei-Endung
```bash
./selfie -l simple.c -m 1     # ❌ FALSCH (.c statt .m)
./selfie -l simple.m -m 1     # ✅ RICHTIG
```

### 3. Ohne RAM-Angabe
```bash
./selfie -c simple.c -m       # ❌ FEHLT RAM
./selfie -c simple.c -m 1     # ✅ RICHTIG
```

---

## ALLE OPTIONEN AUF EINEN BLICK

```bash
# KOMPILIEREN
./selfie -c source.c              # Nur kompilieren
./selfie -c source.c -o out.m     # + Binary speichern
./selfie -c source.c -S out.S     # + Assembly speichern
./selfie -c source.c -s out.s     # + Assembly (kurz)

# AUSFÜHREN (kompiliert + läuft)
./selfie -c source.c -m 1         # Mit mipster
./selfie -c source.c -d 1         # Mit debugger
./selfie -c source.c -r 1         # Mit riscu
./selfie -c source.c -y 1         # Mit hypster

# LADEN + AUSFÜHREN (Binary)
./selfie -l binary.m -m 1         # Mit mipster
./selfie -l binary.m -d 1         # Mit debugger
./selfie -l binary.m -r 1         # Mit riscu
./selfie -l binary.m -y 1         # Mit hypster

# KOMBINIERT
./selfie -c source.c -S out.S -o out.m -m 1
```

---

## ZUSAMMENFASSUNG

**Schnellste Variante (testen):**
```bash
./selfie -c mein_code.c -m 1
```

**Zum Debuggen:**
```bash
./selfie -c mein_code.c -d 1
```

**Für Wiederverwendung:**
```bash
./selfie -c mein_code.c -o mein_code.m
./selfie -l mein_code.m -m 1
```

**RAM anpassen je nach Bedarf:** 1, 2, 4, 8, 16, 32, 64, 128, ... bis 4096 MB
