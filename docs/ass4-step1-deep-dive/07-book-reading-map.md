# 07 — Kirsch/Book Reading Map (gezielt für Ass4 Step 1)

Quelle: `book/README.md` (lokal im Repo)

## Abschnitt A — R-Format verstehen

Anker: um Zeilen ~3044-3057

- erklärt, dass register-register Instruktionen das **R-Format** nutzen
- zeigt das Feldlayout:
  - `funct7 | rs2 | rs1 | funct3 | rd | opcode`
- nennt explizit, dass gleiche `opcode`-Klasse über `funct3/funct7` getrennt wird

**Lernfrage:** Warum ist `opcode` allein zu grob?

## Abschnitt B — Fetch/Decode/Execute Pfad

Anker: um Zeile ~3498

- beschreibt den Kernzyklus:
  - fetch
  - decode
  - execute
- verbindet Theorie mit `selfie.c` Implementierung

**Lernfrage:** An welcher Stelle würdest du einen unknown-instruction Fehler erwarten?

## Abschnitt C — Assignment-spezifischer Teil (`<<`, `>>`)

Anker: um Zeilen ~5039-5049

- Part 1: Scanner/Parser/Präzedenz (Assignment 3)
- Part 2: Codegen + Emulator (`sll`, `srl`) (Assignment 4)
- betont „use what you implement to implement it“

**Lernfrage:** Warum ist Ass3 ohne Ass4 funktional unvollständig?

## Abschnitt D — Wo im Code nachschlagen?

Mit den Suchbegriffen im Buchtext:

- `encode_r_format`
- `decode_r_format` / `decode`
- `emit_add`, `emit_sll`, `emit_srl`
- `do_add`, `do_sll`, `do_srl`
- `OP_OP`, `F3_*`, `F7_*`

## Mini-Lernpfad (30-45 Minuten)

1. R-Format Tabelle lesen
2. `encode_r_format` lesen und einmal manuell Feldpositionen skizzieren
3. `emit_sll`/`emit_srl` vergleichen mit `emit_add`
4. `decode()` Fall `funct3=101` und `funct7`-Trennung prüfen
5. `do_sll`/`do_srl` mit `do_add` vergleichen

