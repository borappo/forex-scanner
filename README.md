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

## Einstiegsanzeige

Steht das Chart-Symbol selbst in einer der beiden Listen, markiert eine gestrichelte
Linie das Einstiegsniveau, daneben steht der aktuelle Abstand des Kurses dazu in Pips;
eine Infobox unten rechts fasst Setup, Niveau und Abstand zusammen. Das Niveau stammt
aus der letzten abgeschlossenen Kerze, der Abstand aktualisiert sich live.

| Setup | Einstieg | Order |
|---|---|---|
| TREND grün | oberes Bollinger Band | Buy-Stop |
| TREND rot | unteres Bollinger Band | Sell-Stop |
| GEGENTREND | Schlusskurs der Signalkerze | Stop in Umkehrrichtung |

Hintergrund ist die Stop-Order-Platzierung: liegt das Niveau zu nah am Kurs, wirkt die
Order faktisch wie eine Market-Order und der Kurs kommt ohne Schwung ins Trade. Ab dem
eingestellten Mindestabstand (Default 10 Pips) ist die Anzeige grün, darunter rot; ein
⚠ zeigt an, dass der Kurs das Niveau bereits überschritten hat.

Die Anzeige folgt exakt der Tabelle: Sie erscheint nur für Symbole, die nach allen
Soft-Filtern in der TREND- oder GEGENTREND-Liste gelandet sind, und nur wenn der
Chart-Broker dem eingestellten entspricht. Steht das Chart-Symbol in keiner Liste, wird
nichts gezeichnet. Das unterscheidet sie von den Breakout-Pfeilen, die bewusst
ungefiltert jeden Schlusskurs außerhalb der Bänder markieren.

Nur im Scanner verfügbar — das Overlay hat sie nicht, da sie das Screening voraussetzt.

## Zwei Varianten

| Datei | Inhalt | Wofür |
|---|---|---|
| `majors-scanner.pine` | Screening der 28 Paare + Tabelle, Chart-Elemente und Einstiegsanzeige | Der morgendliche Durchlauf über alle Paare |
| `majors-overlay.pine` | Nur Bänder, Pfeile und Hintergrund für das Chart-Symbol | Schnelles Draufschauen auf ein einzelnes Paar — lädt deutlich schneller, da keine `request.security`-Abfragen |

Bänder, Breakout-Pfeile und Trend-Hintergrund sind in beiden Dateien identisch;
die Einstiegsanzeige gibt es nur im Scanner. Nicht gleichzeitig auf denselben Chart legen,
sonst liegen die Bänder doppelt übereinander.

## Installation

1. Inhalt der gewünschten `.pine`-Datei kopieren
2. In TradingView den Pine Editor öffnen, Code einfügen, „Zum Chart hinzufügen"
3. Auf den Chart anwenden — alle Elemente folgen dem gewählten Chart-Timeframe

## Einstellungen

Der Scanner bringt alle unten aufgeführten Gruppen mit. Das Overlay hat nur **MACD**
und **Bollinger Bänder** — Filter, Einstieg und Symbol-Slots entfallen dort, da sie
ausschließlich das Screening steuern.

- **MACD** — Fast/Slow/Signal-Länge, Signaltyp SMA oder EMA
- **Bollinger Bänder** — Länge und StdDev-Multiplikator
- **Einstieg** — Mindestabstand Kurs ↔ Einstieg (Pips, Default 10) sowie Toggles für
  Einstiegslinie und Infobox
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
