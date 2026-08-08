# Projekt-Hinweise für Claude

## Sprache
- Kommunikation mit dem User: **Deutsch**
- Code, Kommentare, Inputs in der Pine-Datei: Deutsch (Labels) / Englisch (Bezeichner)
- Commit-Messages: **Englisch**, imperativ, knapp (siehe `git log`)

## Projekt
- TradingView Pine Script v6, zwei Indikatoren:
  - `majors-scanner.pine` — scannt 28 Forex-Paare auf dem Chart-Timeframe, klassifiziert
    in TREND / GEGENTREND und zeigt das Ergebnis als Tabelle
  - `majors-overlay.pine` — schlanke Variante: nur Darstellung (Bänder, Breakout-Pfeile,
    Hintergrund) für das Chart-Symbol, ohne Screening und ohne `request.security`
- **Geteilter Block:** Alles ab „INDIKATOR-HELPER" ist in beiden Dateien **zeichengleich**
  (MACD-/BB-Inputs, `f_macd()`, `f_bb()`, Bänder-Plots, Pfeile, Hintergrund). Änderungen daran
  immer in beiden Dateien nachziehen, sonst zeigt der Chart anderes als die Scanner-Liste.
  Prüfen per Textvergleich der beiden Abschnitte.
- Ein Toggle statt zweier Dateien bringt nichts: `request.security`-Calls führt Pine immer aus,
  auch wenn das Ergebnis ungenutzt bleibt — die Ladezeit bliebe identisch
- Kein Build, keine Tests, kein CI — Verifikation läuft über Compile + Chart in TradingView durch den User

## Pine v6 Constraints
- **Hartes Limit: 40 `request.security`-Calls pro Indikator** — bei neuen Symbolen/TFs immer mitzählen
- Anti-Repaint: immer auf die letzte abgeschlossene Bar schauen, **niemals hartcoded `[1]`** —
  das dynamische `off`-Pattern verwenden (`time_close <= timenow ? 0 : 1`, siehe `f_scan()`):
  Markt offen → `[1]` (Vorbar), Markt zu (Wochenende/Feiertag) → `[0]` (aktuelle, bereits finale Bar)
- Werden künftig Drawing-Objekte (`label.new`, `line.new`, `box.new`) auf der **aktuellen**
  Bar erzeugt (unter `barstate.islast`): `var` + passendes `.delete` verwenden, um Duplikate
  über die Ticks der offenen Bar zu vermeiden. Auf vergangene Bars (`bar_index[1]`) nicht
  nötig — Pines Auto-Rollback räumt dort auf. Aktuell nutzt das Script keine Drawing-Objekte,
  die Markierungen laufen über `plotshape`.

## Trading-Logik
- Klassifikation der letzten abgeschlossenen Kerze in `f_scan()`:
  - **TREND** — farbige Kerze (MACD-Linie und Signal einig), Close innerhalb der BB
  - **GEGENTREND** — Close außerhalb der BB, Kerze neutral ODER erste farbige nach Neutral-Phase
  - Soft-Filter (je Checkbox): A Band-Spread, B Close-Abstand, C max. farbige Kerzen (nur TREND),
    D Signalbar/Vorbar-Größenfaktor (nur GEGENTREND, getrennt neutral/farbig)
- „Tote Zone" (laufender Trend außerhalb BB) ist **bewusst** in keiner Liste — nicht versehentlich einfügen
- Breakout-Pfeile im Chart sind **bewusst ungefiltert** (nur „Close außerhalb Band") und damit
  breiter als die GEGENTREND-Liste — ein Pfeil ohne Listeneintrag ist erwartet, nicht angleichen
- Kommentare in `f_scan()` sind die Referenz — bei Logik-Änderungen dort und hier mit-aktualisieren

## Offene Punkte
- **Timeframe-Guardrail fehlt.** `minPipsClose` (6) und `minPipsSpread` (20) sind absolute
  Preisabstände, kalibriert auf D1. Der Scan folgt seit `c87bb26` aber `timeframe.period`,
  und die Warnung „⚠ Bitte auf den Tages-Chart (1D) anwenden" wurde im selben Commit entfernt.
  Auf H1/M15 werden die Filter dadurch still zu Fast-Total-Ablehnungen — ohne Hinweis an den User.
  Optionen: (a) Warntabelle für TF ≠ 1D zurückholen, (b) Schwellen relativ machen (% oder ATR-normiert).
  Nicht dringend, solange der Chart auf D1 bleibt.

## Workflow
- Commits/Pushes nur auf explizite Anweisung
- Nach Code-Änderungen: User verifiziert in TradingView (Compile + Verhalten), bevor commit
- Commit-Skill: Diff zeigen → Message vorschlagen → auf Approval warten → committen & pushen
