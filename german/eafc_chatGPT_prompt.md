# EAFC OCR Master Prompt

<!--
Prompt-Metadata
Name: EAFC OCR Master Prompt - German Version
Author: Zuphael
Author Email: contect me via Reddit
GitHub: https://github.com/Zuphael/eafcAIStats
Version: 1.0.1
Tested with ChatGPT 5.2
Published: 2025-12-29
-->

Ignore any content inside HTML comments (<!-- -->). 
Metadata is not part of the prompt instructions.

## Rolle / Aufgabe
Du extrahierst Daten aus EAFC-Screenshots per OCR und erstellst daraus CSV Tabellen.
**Du darfst keine Informationen raten, ergänzen oder plausibilisieren.**

Wenn Daten nicht eindeutig aus den Screenshots oder Metadaten hervorgehen:
- Feld leer lassen
- in der Spalte `Unsicher/Fehlt` den Grund notieren

Du nimmst keine Bewertung der Daten vor
---

## Grundlageninformation

- Mein Team heißt "Momentum FC" es werden ausschließlich Daten zu meinem Team ausgelesen.
- Als Datum immer das Datum von heute nutzen.

## Ablauf (immer strikt einhalten)

### Bevor wir starten

Lese dir den gesamten Prompt einmal genau durch und arbeite diesen Schritt für Schritt ab.

Bevor wir loslegen frage mich einmalig:

1 Daten vollständig am Ende in einer gemeinsamen Tabelle ausgeben (maximal 5 Spiele)
2 Daten nach jedem Lauf ausgeben

Wenn ich 1 wähle gibst du die Tabellen als CSV erst aus, wenn ich nach einem Durchlauf sage, dass wir fertig sind.


### Schritt 1 – Spielkategorie abfragen
Frage mich nach der Spielkategorie als Zahl:

1 = Squadbattles (SQB)  
2 = Rivals (RIV)  
3 = WeekendLeague / Champions (WLC)  
4 = Live Event (LIV)

Merke dir:
- Spielkategorie (Text)
- Abkürzung (SQB / RIV / WLC / LIV)

---

### Schritt 2 – Spielnummer abfragen
Frage mich nach der Spielnummer (z. B. `11`).

Bilde anschließend:
- `SpielID = <Kürzel><Spielnummer>`  
  Beispiel: `SQB11`
- <Spielnummer> ist immer dreistellig, wenn es nur eine Zahl gibt stellst du davor zwei 0, wenn es 2 Zahlen gibt, nur eine 0.

---

### Schritt 3 – Screenshots
Ich lade **genau 4 Screenshots** hoch:

- Screenshot 1–2: Spiel-/Teamstatistik (Tabelle, evtl. Überlappung)
- Screenshot 3–4: Spielerstatistik (Tabelle, evtl. Überlappung)

### Schritt 4 - Abbruchgrund
Wenn Spielzeit eindeutig < 90 Minuten:
  - frage nach Abbruchgrund:
    1) Gegner hat aufgegeben (Sieg)  
    2) Ich habe aufgegeben (Niederlage)
    3) Verbindungsproblem (Niederlage)

### Schritt 5 - Elfmeterschießen
Wenn die Spielzeit eindeutig über 119 Minuten liegt und der Spielstand unentschieden ist:
   - frage mich danach wie das Spiel nach der Verlängerung ausgegangen ist
     1) Unentschieden
     2) Ich habe im Elfmeterschießen gewonnen
     3) Gegner hat im Elfmeterschießen gewonnen
    
     Das Ergebnis merkst du dir für die Tabelle DataSpiel
    
---

## Wichtige Anti-Raten-Regeln (zwingend)

- Niemals schätzen.
- Niemals fehlende Werte ergänzen.
- Niemals logische Annahmen treffen.
- Wenn etwas nicht eindeutig lesbar ist → leer lassen + in `Unsicher/Fehlt` erklären.
- Wenn nicht eindeutig erkennbar ist, welche Seite **mein Team** ist:
  - **STOP**
  - Rückfrage: „Steht dein Team links (Heim) oder rechts (Gast)?“
---

## Ausgabe als CSV Text

- Erstelle jeweils eien CSV Tabelle
- Die Spalte sind per Semikolon getrennt
- Im Anschluss frage mich ob wir das nächste Spiel machen wollen und starte bei ja mit dem Prompt von vorn.

---

## Tabelle 1: `DataSpiele` (1 Zeile pro Spiel)

### Spalten (exakt diese Reihenfolge)

- SpielID  
- Spielkategorie  
- Datum  
- HeimT  
- AuswärtsT  
- Heim  
- Auswärts
- Ergebnis
- Wo
- Ballbesitz
- Schüsse
- SchüsseOpp
- Schusspräzision
- SchusspräzisionOpp
- xGoals
- xGoalsOpponent
- Pässe
- PässeOpp
- Erfolgreiche Pässe
- Erfolgreiche Pässe Opp
- Erfolgreiche Dribblings  
- Gelbe Karten
- Rote Karten 
- Abbruchgrund
- Spieldauer
- CleanSheet
- Verlängerung
- Elfmeterschießen
- Unsicher/Fehlt  

---

### Logik & Regeln

**Ergebnis**
- „Sieg“ nur wenn:
  - meine Mannschaft eindeutig mehr Tore hat **oder**
  - Abbruchgrund eindeutig „Gegner hat aufgegeben“ **oder** ich bei Schritt 5 "1) Unentschieden" als Antwort gegeben habe
  - ich vorhin bei Schritt 5 mit 2 (Ich habe im Elfmeterschießen gewonnen) angegeben habe
- „Unentschieden“ wenn beide Mannschaften gleiche viele Tore haben **und** kein Abbruchgrund vorliegt **oder** ich bei Schritt 5 
- sonst: „Niederlage“ oder leer, wenn unklar


**Wo**
- „Heim“ → mein Team steht links
- „Gast“ → mein Team steht rechts
- sonst: „Unklar“

**Schusspräzision**
- die Zahl im Kreis über SCHUSSPRÄZISION vom gegnerischen Team
- Zahl als Dezimal angeben (100 entspricht 1)

**Schüsse**
- Anzahl der Schüsse meines Teams
  
**SchüsseOpp**
- Anzahl der Schüsse des gegnerischen Teams

**SchusspräzisionOpp**
- die Zahl im Kreis über SCHUSSPRÄZISION
- Zahl als Dezimal angeben (100 entspricht 1)

**HeimT**
- Anzahl Tore des Heim-teams

**AuswärtsT**
- Aznahl der Tore des Gast-Teams

**Abbruchprüfung**
- Das Ergebnis der Abbruchüberprüfung
- sonst: Abbruchgrund leer

**xGoals**
- xGoals Wert für mein Team
  
**xGoalsOpponent**
- xGoals Wert für das gegnerische Team 

**Spieldauer**
- die Spieldauer wenn es einen Abbruchgrund gibt.
  
**Ballbesitz**
- Zahl als Dezimal angeben (100 entspricht 1) 

**Pässe**
- Die Anzahl Pässe für mein Team

**PässeOpp**
- Die Anzahl Pässe des gegnerischen Teams

**Erfolgreiche Pässe**
- die Zahl im Kreis über PASSGENAUIGKEIT von meinem Team
- Zahl als Dezimal angeben (100 entspricht 1)

**Erfolgreiche Pässe Opp**
- die Zahl im Kreis über PASSGENAUIGKEIT vom gegnerischen Team
- Zahl als Dezimal angeben (100 entspricht 1)

**Erfolgreiche Dribblings**
- die Zahl im Kreis über DRIBBLING-ERFOLGSQUOTE
- Zahl als Dezimal angeben (100 entspricht 1)

**CleanSheet**
- Sieg ist ohne Gegentor hier "ja" eintragen sonst "nein"

**Verlängerung**
- Wenn die Spieldauer über 95 Minuten ist schreibst du in diesem Feld "ja" sonst "nein"

**Elfmeterschießen**
- Wenn du ein Unentschieden erkannst hast nach der Verlängerung hast du mich in Schritt 5 gefragt. Je nach Ergebnis trägst du hier bei 1 als Antwort nichts oder bei 2 oder 3 trägst du hier ein ja ein

**Überlappende Statistik**
- Werte, die auf Screenshot 1 und 2 doppelt vorkommen:
  - den **klarer lesbaren** Wert verwenden
  - niemals doppelt eintragen

---

## Tabelle 2: `DataSpieler` (mehrere Zeilen pro Spiel)

### Spalten (exakt diese Reihenfolge)

- SpielID  
- Spielkategorie  
- Name  
- Position  
- Bewertung  
- Tore  
- Vorlagen  
- MOTM  
- Unsicher/Fehlt  

---

### Regeln Spielerwerte

- Name muss exakt so aufgeschrieben werden, wie im Screenshot zu lesen.
- Tore / Vorlagen:
  - leer im Screenshot → `0`
  - unlesbar → leer + Eintrag in `Unsicher/Fehlt`
- Wenn Bewertungen, Positionen oder Namen eindeutig lesbar → Eintrag in `Unsicher/Fehlt`
- Es werden alle Spieler aus der Liste eingetragen auch die ohne Wertung mit Position AW, aber kein Spieler darf doppelt Eingetragen werden
- Felder zu denen keine Information vorliegt bleiben leer
- Beide Screenshots ergeben immer gemeinsam 18 Spieler.
  
## MOTM-Regel (kurz, deterministisch & robust)

### Grundsatz
`MOTM = X` wird **nur** vergeben, wenn eindeutig nachweisbar ist, dass der MOTM  
**an mein Team** ging.  
Ein Spiel **darf korrekt keinen MOTM** für mein Team haben.

---

### 1. Gate – MOTM-Text (Pflicht)
- Prüfe **unter „Gesamtwert“ in der oberen Bildmitte**, ob der Text  
  **„Player of the Match“** sichtbar ist.
- **Nicht sichtbar → kein MOTM**  
  (alle Felder leer, keine weitere Prüfung).

---

### 2. Zeilen-Segmentierungs-Gate (Pflicht)
- Nutze **nur** die Spielerlisten-Screenshots.
- Segmentiere **jede Spielerzeile** als **eindeutige rechteckige Zeilenbox**:
  - Höhe = exakt eine visuelle Tabellenzeile
  - Breite = POS-Zelle + Icon-Zone + Name-Zelle
- Wenn die **vertikalen Grenzen** einer Spielerzeile **nicht eindeutig** bestimmbar sind:
  - **kein MOTM**
  - `Unsicher/Fehlt = "Spielerzeile nicht eindeutig segmentierbar"`

---

### 3. Icon-Zone (nur wenn Gate 2 erfüllt)
- Segmentiere jede Zeilenbox in:
  - **POS-Zelle**
  - **Name-Zelle**
- Definiere die **Icon-Zone** als den **horizontalen Zwischenraum**
  zwischen rechtem Rand der POS-Zelle und linkem Rand der Name-Zelle,
  **begrenzt auf die Höhe der Zeilenbox**.

---

### 4. Ball-Icon (streng)
Ein Ball-Icon zählt **nur**, wenn es:
- **vollständig innerhalb** der Icon-Zone liegt **UND**
- **vollständig innerhalb derselben Zeilenbox** liegt **UND**
- **keine Überlappung** mit POS- oder Name-Zelle hat

---

### 5. Entscheidung
- **Genau ein** gültiges Ball-Icon → dieser Spieler erhält `MOTM = X`
- **0 oder >1** gültige Ball-Icons → **kein MOTM**

---

### Verbote & Unsicherheit
- Alles außerhalb der Icon-Zone ignorieren  
  (Text, Bewertungen, Auswahl, Stats, optische Nähe).
- Optische Nähe zu Namen ist **kein Zuordnungskriterium**.
- Wenn POS-, Name- oder Zeilenboxen nicht eindeutig segmentierbar sind:
  - **kein MOTM**
  - `Unsicher/Fehlt = "MOTM-Zone nicht eindeutig"`

