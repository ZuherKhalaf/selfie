# Was passiert bei: ./selfie -c simple.c

## Antwort: Nur KOMPILIEREN, KEINE Ausgabe-Datei!

---

## Was passiert INTERN

### 1. **Compiler läuft**
```
selfie compiling examples/own/simple.c to 64-bit RISC-U with 64-bit starc
```

Der Compiler:
- Liest `simple.c`
- Parst den Code (Lexer, Parser)
- Generiert RISC-U Maschinenbefehle
- **Speichert alles IM SPEICHER** (RAM)

### 2. **Statistiken werden ausgegeben**

```
46 characters read in 6 lines and 0 comments
0 global variables, 1 procedures, 0 string literals
1 assignments, 0 while, 0 if, 0 calls, 1 return
```

**Zeigt:**
- Was wurde gelesen
- Was wurde gefunden
- Wie viel Code generiert

### 3. **Code-Profil wird angezeigt**

```
408 bytes generated with 100 instructions and 8 bytes of data

profile: instruction: total(ratio%)
init:    lui: 1(1.00%), addi: 47(47.00%)
memory:  ld: 18(18.00%), sd: 8(8.00%)
...
```

**Zeigt:**
- Wie viele Bytes Code
- Welche Instruktionen verwendet
- Prozentuale Verteilung

### 4. **KEINE DATEI WIRD ERSTELLT!**

Es gibt:
- ❌ Kein `simple.m` (Binary)
- ❌ Kein `simple.s` (Assembly)
- ✅ Nur Statistik-Ausgabe

---

## UNTERSCHIED ZU ANDEREN FLAGS

### Nur kompilieren (KEIN Output):
```bash
./selfie -c simple.c
```
**→ Nur Compiler-Statistiken, keine Datei**

### Kompilieren + Binary speichern:
```bash
./selfie -c simple.c -o simple.m
```
**→ Erstellt `simple.m` (RISC-U Binary)**

### Kompilieren + Assembly speichern:
```bash
./selfie -c simple.c -S simple.S
```
**→ Erstellt `simple.S` (Assembly mit Details)**

```bash
./selfie -c simple.c -s simple.s
```
**→ Erstellt `simple.s` (Assembly ohne Details)**

### Kompilieren + Ausführen:
```bash
./selfie -c simple.c -m 1
```
**→ Kompiliert UND führt sofort aus (1MB RAM)**

### Kompilieren + Debuggen:
```bash
./selfie -c simple.c -d 1
```
**→ Kompiliert UND führt mit Debugger aus**

---

## WARUM NUR KOMPILIEREN?

**Sinnvoll um:**

1. **Syntax zu checken**
   ```bash
   ./selfie -c mein_code.c
   ```
   Zeigt Fehler falls Code ungültig

2. **Code-Statistiken zu sehen**
   - Wie viele Instruktionen?
   - Welche Instruktionen verwendet?
   - Wie viel Speicher braucht der Code?

3. **Performance zu vergleichen**
   ```bash
   ./selfie -c version1.c
   ./selfie -c version2.c
   ```
   Welche Version generiert weniger Code?

---

## WAS BLEIBT IM SPEICHER

Nach `./selfie -c simple.c` ist im RAM:

```
┌─────────────────────────┐
│ Source Code (simple.c)  │ ← Eingelesen
├─────────────────────────┤
│ Binary Code (RISC-U)    │ ← Generiert
├─────────────────────────┤
│ Symbol Table            │ ← Variablen/Funktionen
├─────────────────────────┤
│ String Literals         │ ← Konstante Strings
└─────────────────────────┘
```

**Aber:** Sobald selfie beendet → ALLES WEG!

Deshalb brauchst du `-o` oder `-S` um es zu speichern.

---

## BEISPIEL-SESSION

```bash
# Nur kompilieren - Code prüfen
./selfie -c test.c
# → Gibt Statistiken aus
# → KEINE Datei erstellt

# Falls keine Fehler → Binary erstellen
./selfie -c test.c -o test.m

# Binary ausführen
./selfie -l test.m -m 1
```

---

## COMPILER-STATISTIKEN VERSTEHEN

### Zeilen gelesen:
```
46 characters read in 6 lines and 0 comments
```
- Dein Source Code hat 46 Zeichen
- Verteilt auf 6 Zeilen
- Keine Kommentare

### Code-Elemente:
```
0 global variables, 1 procedures, 0 string literals
1 assignments, 0 while, 0 if, 0 calls, 1 return
```
- 0 globale Variablen
- 1 Funktion (main)
- 0 Strings
- 1 Zuweisung (x = 7 + 3)
- 0 while/if
- 1 return

### Generierter Code:
```
408 bytes generated with 100 instructions and 8 bytes of data
```
- 408 Bytes Gesamt
- 100 RISC-U Instruktionen
- 8 Bytes Daten

### Instruktions-Mix:
```
addi: 47(47.00%)    → Fast die Hälfte sind addi!
ld: 18(18.00%)      → 18% Loads
sd: 8(8.00%)        → 8% Stores
...
```

---

## ZUSAMMENFASSUNG

**Befehl:**
```bash
./selfie -c examples/own/simple.c
```

**Macht:**
1. ✅ Kompiliert den Code
2. ✅ Zeigt Statistiken
3. ✅ Prüft auf Fehler
4. ❌ Erstellt KEINE Datei
5. ❌ Führt NICHT aus

**Nützlich für:**
- Syntax-Check
- Code-Analyse
- Performance-Vergleich
- Lernzwecke (Statistiken studieren)

**Wenn du was behalten willst:**
```bash
./selfie -c simple.c -o simple.m     # Binary
./selfie -c simple.c -S simple.S     # Assembly
./selfie -c simple.c -m 1            # Sofort ausführen
```
