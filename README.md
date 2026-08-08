# Forex Majors Scanner

TradingView-Indikator (Pine Script v6), der 28 Forex-Paare auf dem aktuellen
Chart-Timeframe scannt und in zwei Listen klassifiziert: **TREND** und **GEGENTREND**.

## Funktionsweise

Pro Paar wird die letzte **abgeschlossene** Kerze des Chart-Timeframes ausgewertet
(kein Repaint, auch am Wochenende korrekt):

- **TREND** — MACD bestätigt einen Trend (Linie und Signal einig), Schlusskurs
  innerhalb der Bollinger Bänder → sauberer Trend-Einstieg
- **GEGENTREND** — Schlusskurs außerhalb der Bollinger Bänder bei MACD-Schwäche
  → potenzielles Reversal-Setup

Das Ergebnis erscheint als Tabelle oben rechts im Chart; das aktuelle Chart-Symbol
wird mit ▶ markiert.

## Zwei Varianten

| Datei | Inhalt | Wofür |
|---|---|---|
| `majors-scanner.pine` | Screening der 28 Paare + Tabelle, dazu Bänder, Breakout-Pfeile und Trend-Hintergrund | Der morgendliche Durchlauf über alle Paare |
| `majors-overlay.pine` | Nur Bänder, Breakout-Pfeile und Trend-Hintergrund für das Chart-Symbol | Schnelles Draufschauen auf ein einzelnes Paar — lädt deutlich schneller, da keine `request.security`-Abfragen |

Beide zeichnen dasselbe Chart-Bild; der Darstellungsteil ist in beiden Dateien identisch.
Nicht gleichzeitig auf denselben Chart legen, sonst liegen die Bänder doppelt übereinander.

## Installation

1. Inhalt der gewünschten `.pine`-Datei kopieren
2. In TradingView den Pine Editor öffnen, Code einfügen, „Zum Chart hinzufügen"
3. Auf den Chart anwenden — alle Elemente folgen dem gewählten Chart-Timeframe

## Einstellungen

Der Scanner bringt alle unten aufgeführten Gruppen mit. Das Overlay hat nur
**MACD** und **Bollinger Bänder** — Filter und Symbol-Slots entfallen dort, da sie
ausschließlich das Screening steuern.

- **MACD** — Fast/Slow/Signal-Länge, Signaltyp SMA oder EMA
- **Bollinger Bänder** — Länge und StdDev-Multiplikator
- **Filter Kriterien TREND & GEGENTREND** — fünf Soft-Filter, einzeln per Checkbox
  deaktivierbar: Mindestabstand Close ↔ Außenband (Pips), Mindestabstand
  Mittel- ↔ Außenband (Pips), max. Anzahl farbiger Kerzen für ein TREND-Signal,
  Mindest-Faktor Signalbar/Vorbar (Handelsspanne) — getrennt für neutralen und
  farbigen Gegentrend (nur GEGENTREND-Screening)
- **Darstellung** — Tabellen-Versatz nach links; Sichtbarkeit von Bollinger
  Bändern und Breakout-Pfeilen wird direkt im Style-Tab gesteuert
- **Symbole** — Broker/Datenquelle (SWISSQUOTE, SAXO oder IBKR) und 28 Slots,
  jeweils per Checkbox aktivierbar (Paar ohne Broker-Prefix eintragen, z.B.
  `EURUSD`); AUD- und NZD-Paare sind per Default deaktiviert
