# Projekt-Hinweise für Claude

## Sprache
- Kommunikation mit dem User: **Deutsch**
- Code, Kommentare, Inputs in der Pine-Datei: Deutsch (Labels) / Englisch (Bezeichner)
- Commit-Messages: **Englisch**, imperativ, knapp (siehe `git log`)

## Projekt
- TradingView Pine Script v6 Indikator: `majors-scanner.pine` (einzige Code-Datei)
- Scannt 28 Forex-Paare auf dem Chart-Timeframe und klassifiziert in TREND / GEGENTREND
- Kein Build, keine Tests, kein CI — Verifikation läuft über Compile + Chart in TradingView durch den User

## Pine v6 Constraints
- **Hartes Limit: 40 `request.security`-Calls pro Indikator** — bei neuen Symbolen/TFs immer mitzählen
- Anti-Repaint: immer auf die letzte abgeschlossene Bar schauen, **niemals hartcoded `[1]`** —
  das dynamische `off`-Pattern verwenden (`time_close <= timenow ? 0 : 1`, siehe `f_scan()`):
  Markt offen → `[1]` (Vorbar), Markt zu (Wochenende/Feiertag) → `[0]` (aktuelle, bereits finale Bar)
- Bei Re-Tick-anfälligen Elementen (`label.new` o.ä.) auf der **aktuellen** Bar
  (z.B. `label.new(bar_index, ...)` unter `barstate.islast`): `var` + `label.delete`
  Pattern verwenden, um Duplikate über die Ticks der offenen Bar zu vermeiden.
  Auf vergangene Bars (`bar_index[1]`) ist es nicht nötig — Pines Auto-Rollback
  räumt dort auf.

## Trading-Logik
- Klassifikation der letzten abgeschlossenen Kerze in `f_scan()`:
  - **TREND** — farbige Kerze (MACD-Linie und Signal einig), Close innerhalb der BB
  - **GEGENTREND** — Close außerhalb der BB, Kerze neutral ODER erste farbige nach Neutral-Phase
  - Soft-Filter (je Checkbox): A Band-Spread, B Close-Abstand, C max. farbige Kerzen (nur TREND),
    D Signalbar/Vorbar-Größenfaktor (nur GEGENTREND, getrennt neutral/farbig)
- „Tote Zone" (laufender Trend außerhalb BB) ist **bewusst** in keiner Liste — nicht versehentlich einfügen
- Kommentare in `f_scan()` sind die Referenz — bei Logik-Änderungen dort und hier mit-aktualisieren

## Workflow
- Commits/Pushes nur auf explizite Anweisung
- Nach Code-Änderungen: User verifiziert in TradingView (Compile + Verhalten), bevor commit
- Commit-Skill: Diff zeigen → Message vorschlagen → auf Approval warten → committen & pushen
