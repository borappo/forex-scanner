# Projekt-Hinweise für Claude

## Sprache
- Kommunikation mit dem User: **Deutsch**
- Code, Kommentare, Inputs in der Pine-Datei: Deutsch (Labels) / Englisch (Bezeichner)
- Commit-Messages: **Englisch**, imperativ, knapp (siehe `git log`)

## Projekt
- TradingView Pine Script v6, zwei Indikatoren:
  - `majors-scanner.pine` — scannt 28 Forex-Paare auf dem Chart-Timeframe, klassifiziert
    in TREND / GEGENTREND und zeigt das Ergebnis als Tabelle
  - `majors-overlay.pine` — schlanke Variante: nur Bänder, Breakout-Pfeile und
    Hintergrund für das Chart-Symbol, ohne Screening und ohne `request.security`
- **Geteilter Block:** In beiden Dateien **zeichengleich** — Inputs (MACD, BB),
  Helper (`f_macd`, `f_bb`) und die Darstellungs-Abschnitte Bänder, Pfeile, Hintergrund.
  Änderungen daran immer in beiden Dateien nachziehen, sonst zeigt der Chart anderes
  als die Scanner-Liste. Prüfen per Textvergleich der Abschnitte — muss leer bleiben.
- **Die `indicator()`-Zeile liegt außerhalb der Vergleichsblöcke** und muss separat geprüft
  werden: erlaubt ist dort nur ein anderer Titel, alle Parameter müssen übereinstimmen.
  `max_bars_back = 200` ist Pflicht in beiden — variable History-Offsets (`[off]`,
  `[off + 1]`, siehe `f_entry()`) lösen sonst zur **Laufzeit** „cannot determine the
  referencing length of a series" aus. Das kompiliert sauber durch und fällt erst auf
  dem Chart auf.
- Scanner-exklusiv: Filter- und Einstiegs-Inputs, Symbol-Slots, `f_pip()`, `f_entry()`,
  `f_scan()`, `handle()`, Ergebnis-Tabelle und die Einstiegsanzeige
- Ein Toggle statt zweier Dateien bringt nichts: `request.security`-Calls führt Pine immer aus,
  auch wenn das Ergebnis ungenutzt bleibt — die Ladezeit bliebe identisch
- Kein Build, keine Tests, kein CI — Verifikation läuft über Compile + Chart in TradingView durch den User

## Pine v6 Constraints
- **Hartes Limit: 40 `request.security`-Calls pro Indikator** — bei neuen Symbolen/TFs immer mitzählen
- Anti-Repaint: immer auf die letzte abgeschlossene Bar schauen, **niemals hartcoded `[1]`** —
  das dynamische `off`-Pattern verwenden (`time_close <= timenow ? 0 : 1`, siehe `f_entry()`):
  Markt offen → `[1]` (Vorbar), Markt zu (Wochenende/Feiertag) → `[0]` (aktuelle, bereits finale Bar)
- Drawing-Objekte (`label.new`, `line.new`, `box.new`) auf der **aktuellen** Bar
  (unter `barstate.islast`): `var` + passendes `.delete` verwenden, um Duplikate über die
  Ticks der offenen Bar zu vermeiden — siehe Einstiegslinie und -Label. Auf vergangene Bars
  (`bar_index[1]`) nicht nötig, Pines Auto-Rollback räumt dort auf. Die Breakout-Markierungen
  brauchen es nicht, sie laufen über `plotshape`.

## Trading-Logik
- Klassifikation der letzten abgeschlossenen Kerze in **`f_entry()`** (einzige Definition):
  - **TREND** — farbige Kerze (MACD-Linie und Signal einig), Close innerhalb der BB
  - **GEGENTREND** — Close außerhalb der BB, Kerze neutral ODER erste farbige nach Neutral-Phase
  - `f_scan()` ruft `f_entry()` auf und ergänzt nur die Soft-Filter (je Checkbox):
    A Band-Spread, B Close-Abstand, C max. farbige Kerzen (nur TREND),
    D Signalbar/Vorbar-Größenfaktor (nur GEGENTREND, getrennt neutral/farbig)
- **Einstiegsniveau** (ebenfalls `f_entry()`), immer aus der letzten abgeschlossenen Kerze:
  TREND grün → oberes Band (Buy-Stop), TREND rot → unteres Band (Sell-Stop),
  GEGENTREND → Schlusskurs der Signalkerze, Richtung aus der durchbrochenen Bandseite
- „Tote Zone" (laufender Trend außerhalb BB) ist **bewusst** in keiner Liste — nicht versehentlich einfügen
- **Zwei verschiedene Sichtbarkeitsregeln — nicht angleichen:**
  - Breakout-Pfeile: **ungefiltert**, prüfen nur „Close außerhalb Band". Ein Pfeil ohne
    Listeneintrag ist erwartet (tote Zone, greifender Filter)
  - Einstiegsanzeige: **gefiltert**, erscheint nur, wenn das Chart-Symbol tatsächlich in
    der TREND- oder GEGENTREND-Liste steht (`array.includes` + `brokerMatch`). Sie ist ein
    Handelssignal, kein Strukturhinweis — deshalb muss sie zur Tabelle passen
- Der Einstiegs-Abstand ist als einziges Element **live** (aktualisiert je Tick); das Niveau
  selbst bleibt auf der letzten abgeschlossenen Kerze eingefroren
- Pine-Funktionen dürfen globale Variablen **nicht zuweisen** — nur Array-Inhalte ändern.
  Deshalb sammelt `handle()` die Ergebnisse in Arrays; Werte aus `request.security` lassen
  sich von dort nicht per `:=` in Globals schreiben
- Kommentare in `f_entry()` und `f_scan()` sind die Referenz — bei Logik-Änderungen dort und
  hier mit-aktualisieren

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
