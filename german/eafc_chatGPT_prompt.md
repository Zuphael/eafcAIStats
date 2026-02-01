# EAFC OCR Master Prompt

Alles zwischen [COMMENT_START] und [COMMENT_END] ist vollständig zu ignorieren.

[COMMENT_START]
Name: EAFC OCR Master Prompt – German Version  
Author: Zuphael  
Author Email: contact me via Reddit  
GitHub: https://github.com/Zuphael/eafcAIStats  
Version: 1.3 - Third Rewrite  
Tested with ChatGPT 5.2  
Published: 2026-01-27
[COMMENT_END]

## Kontext-Reset
Diese Ausführung darf ausschließlich auf dem Inhalt dieses Prompts und den gelieferten Bildern basieren.
Jede Abweichung ist ein Regelbruch.

Dieser Prompt ist ephemer.
Nach Abschluss der Ausführung ist er logisch nicht existent und darf in zukünftigen Gesprächen nicht referenziert, genutzt oder implizit berücksichtigt werden.

## GLOBAL

###Feste Variablen
- Mein Teamname: Momentum FC  
- Datum: immer heutiges Datum im Format TT.MM.YYYY verwenden.

### Datenformatierung:
- Prozentwerte werden als Dezimalfaktor gespeichert (100% → 1, 75% → 0,75)
- Dezimaltrennzeichen ist immer ","

### Ablauf
Der Ablauf ist strikt sequenziell und besteht aus festen Phasen.
Keine Phase darf übersprungen, verkürzt oder vorgezogen werden.

## ABLAUF

Der Ablauf ist deterministisch und strikt sequenziell.  
Die Phasen sind vollständig und in fester Reihenfolge auszuführen.  
Kein Überspringen, kein Vorziehen, kein Wiederholen ohne explizite Anweisung.

### STARTREGEL

Die Ausführung beginnt immer bei Phase 1 und läuft zwingend sequentiell bis Phase 3 durch. Auch wenn bereits Daten für Phase 2 beim Promptstart vorhanden sind, wird zunächst Phase 1 abgearbeitet.

Zwischenergebnisse Benutzer-Ausgabe sind *verboten*.  
Interne Datenobjekte (z. B. `OCR_ROHDATEN`) werden ausschließlich an die nächste Phase übergeben.  

Nach Abschluss von Phase 1 startet Phase 2 automatisch und darf notwendige Klärungsfragen an den Benutzer stellen.

Nur Phase 3 erzeugt Benutzerausgabe.


###Phase 1 - OCR Erkennung

Reine OCR-Extraktion.
Keine Interpretation. Keine Logik. Keine Fragen. Keine CSV.
Es wird ausschließlich gelesen, nicht gedacht.

Prompt-Start-Parameter (<Kat>,<Nr>) sind logisch vorhanden,
dürfen aber erst in Phase 2 gelesen oder interpretiert werden.
In Phase 1 sind sie inert.

#### Input:
- Genau 4 Screenshots
  - 2 Screenshots → Spielstatistik
  - 2 Screenshots → Spielerstatistik

#### Aufgabe:
Lies ALLE sichtbaren Daten roh aus. 

Phase 1 = reine Texterkennung.
Alles was nicht Lesen ist, ist verboten.


##### Aus Spielstatistik:
- Spielstand
- Spielzeit
- Teamnamen (links / rechts)
- Kreiswerte (%):
  - Ballbesitz
  - Dribbling-Erfolgsquote
  - Schusspräzision
  - Passgenauigkeit
- Alle Teamzahlen:
  - Schüsse
  - xGoals
  - Pässe
  - Zweikämpfe
  - Karten
  - usw.


##### Aus Spielerstatistik (ERSATZ – vollständig gehärtete Version)

###### MOTM-Existenz-Gate (Text, nur Existenz – keine Zuordnung)

Suchraum ausschließlich:
- x: 36–55 % Bildbreite
- y: 18–28 % Bildhöhe

Regel:
Wenn im Suchraum der Text „Gesamtwert“ vorkommt
und direkt darunter (≤ 1 Textzeilenhöhe)
der Text „Player of the Match“ steht:

OCR_ROHDATEN.meta.gesamtwert_text = "Player of the Match"

sonst:

OCR_ROHDATEN.meta.gesamtwert_text = "nein"

Wichtig:
- Dieses Gate zeigt ausschließlich, dass irgendein MOTM existiert.
- Es trifft keine Aussage, welcher Spieler MOTM ist.

###### Text-Gate (harte Vorbedingung)

Wenn:
OCR_ROHDATEN.meta.gesamtwert_text != "Player of the Match"

dann gilt zwingend:
ballIcon = "nein" für alle Spieler

- Keine Icon-Erkennung
- Alle nachfolgenden Ball-Icon-Regeln sind inaktiv

###### Verbotene Quellen (absolut)

- Rechter Detailbereich (Portrait, Heatmap, Name groß, Einzelspieler-Stats)
- Bewertungsring / Gesamtwert
- Ausgewählte Tabellenzeile
- UI-Selektion (Highlight)
- „Player of the Match“ außerhalb des definierten Text-Gates

###### Erlaubte Quelle (exklusiv)

- MOTM-Zuordnung ausschließlich über Icon-Spalte der Tabellenliste

###### Spieler-OCR (pro Screenshot)

Suchraum:
- x: 0–36 %
- y: 33–88 %

Es existieren exakt 12 Tabellenzeilen (Z1–Z12).

Pro Tabellenzeile roh lesen:
- POS
- Name
- RB (oder „N.V.“)
- T
- VOR

Spielername – harte Regel:
- Name exakt aus Tabellenzeile
- Keine Auflösung von Initialen
- Rechter Detailbereich ist keine Quelle

###### ballIcon – Icon-Erkennung

Icon-Spalte:
- x: 6,8–9,1 %
- nur im Suchraum

####### UI-Selektion – ABSOLUTER BLOCKER

Wenn eine Pixelgruppe:
- ≥ 90 % Zeilenhöhe
- oder ≥ 60 % ROI-Breite
- oder Kontakt mit Zeilenrand
- oder weiß/hellgrau und flächig

Dann:
ballIcon = "nein"

####### Ball-Icon-Erkennung (nur ohne Blocker)

Eine Pixelgruppe ist Ball-Icon nur wenn:
1) Höhe ≥ 10 px
2) Breite ≥ 10 px
3) In 25–75 % der Zeilenhöhe
4) Seitenverhältnis 0,5–2,0
5) Überlappung ≥ 70 %

Wenn erfüllt:
ballIcon = "ja"
sonst:
ballIcon = "nein"

Uneindeutig → ballIcon = "nein"
	
  
###Format für OCR_ROHDATEN

```
OCR_ROHDATEN:{
  "spiel":{
    "spielstand":"",
    "spielzeit":"",
    "teamL":"",
    "teamR":"",
    "kreis":{
      "ballbesitz":{"L":"","R":""},
      "dribbling":{"L":"","R":""},
      "schuss":{"L":"","R":""},
      "pass":{"L":"","R":""}
    },
    "werte":{
      "ballrueckeroberung_s":{"L":"","R":""},
      "schuesse":{"L":"","R":""},
      "xGoals":{"L":"","R":""},
      "paesse":{"L":"","R":""},
      "zweikaempfe":{"L":"","R":""},
      "gew_zweikaempfe":{"L":"","R":""},
      "abgefangen":{"L":"","R":""},
      "paraden":{"L":"","R":""},
      "fouls":{"L":"","R":""},
      "abseits":{"L":"","R":""},
      "ecken":{"L":"","R":""},
      "freistoesse":{"L":"","R":""},
      "elfmeter":{"L":"","R":""},
      "gelb":{"L":"","R":""},
      "rot":{"L":"","R":""},
      "abwehr_direkt":{"L":"","R":""},
      "abwehr_aussen":{"L":"","R":""},
      "abwehr_hoch":{"L":"","R":""},
      "abwehr_versucht":{"L":"","R":""}
    }
  },
  "spieler":[
    {"ss":"","name":"","pos":"","bew":"","tore":"","vorl":"","ballIcon":"ja/nein"}
  ],
  "meta":{"gesamtwert_text":""}
}

```

Phase-1-Ergebnis:
Phase 1 erzeugt als Ergebnis ausschließlich das Objekt `OCR_ROHDATEN`, dieses wird nicht ausgegeben sondern an Phase 2 übergeben.

Dieses Objekt ist die verpflichtende Eingabe für Phase 2.  
Es handelt sich nicht um eine finale Ausgabe, sondern um interne Rohdaten.



In Phase 1:
- keine Weiterverarbeitung  
- keine Normalisierung  
- keine Validierung  
- keine finale Ausgabe  
- nur Erzeugung von `OCR_ROHDATEN`


PHASE_1_COMPLETE = true

###Phase 2 - Regelbasierte Datenaufbereitung und Klärung

Phase 2 darf nur starten, wenn PHASE_1_COMPLETE == true

Phase 2 arbeitet ausschließlich auf Basis der in Phase 1 erzeugten OCR_ROHDATEN sowie optionaler Prompt-Start-Parameter.
Es dürfen keine neuen Daten erzeugt oder angenommen werden.

#### Klärung Spielkategorie und SpielID

Pflicht-Gate:
Spielkategorie und Spielnummer müssen vor jeder weiteren Phase-2-Verarbeitung eindeutig geklärt sein.
Ohne gültige Werte darf Phase 2 nicht fortgesetzt werden.

Start-Regel:
- Wenn am Prompt-Start ein gültiges `<Kat>,<Nr>` vorliegt → übernehmen, Gate erfüllt.
- Andernfalls → Kategorie/Nummer per Frage abfragen und auf Antwort warten.

Antwort-Regel:
- Eine Antwort im Format `<Kat>,<Nr>` gilt jederzeit als Kategorie/Nummer,
  sofern beide Werte gültig sind.
- Nach gültiger Antwort ist das Gate erfüllt und Phase 2 wird fortgesetzt.

Gültige Kategorien:
1 = Squadbattles (SQB)
2 = Rivals (RIV)
3 = Champions (WLC)
4 = Live Event (LIV)

SpielID:
SpielID = <Kürzel><dreistellig> (führende Nullen).
Beispiel: RIV054

Kategorie: ist die Kategorie ohne ABkürzung.

Solange Spielkategorie oder Spielnummer ungeklärt sind:
- keine weitere Phase-2-Regel anwenden
- keine Felder berechnen
- keine Annahmen treffen
- ausschließlich auf gültige `<Kat>,<Nr>` warten

#### Fragen zu Spielstatistik und DataSpiele

##### A) Spielort (Wo) - autmatisch entscheiden, keine Rückfrage

- **Heim** → mein Team steht **links**
- **Auswärts** → mein Team steht **rechts**

Notiere in der Spalte "Wo" ob mein Team Heim oder Gast ist

- **Heim** = Name des Team Links
- **Auswärts** = Name des Team Rechts


##### B) Abbruch (Pflicht bei Spielzeit < 90:00)

Trigger:  
- Wenn `Spieldauer < 90:00` → Abbruch ist zwingend zu klären.

Frage:
1) Gegner hat aufgegeben (Sieg)  
2) Ich habe aufgegeben (Niederlage)  
3) Verbindungsproblem (Niederlage)

Regel:
- Die Antwort ist verpflichtend.
- Ohne Antwort keine Weiterverarbeitung.
- Das Feld `Abbruchgrund` wird exakt mit dem Text der gewählten Option befüllt.

##### C) Elfmeterschießen (Pflicht bei > 119:00 & Unentschieden)

Trigger:  
- Wenn `Spieldauer > 119:00` **und** `HeimT = AuswärtsT` → Ausgang nach Verlängerung ist zwingend zu klären.

Frage:
1) Unentschieden (kein Elfmeterschießen)  
2) Ich habe im Elfmeterschießen gewonnen  
3) Gegner hat im Elfmeterschießen gewonnen

Regel:
- Die Antwort ist verpflichtend. Ohne Antwort keine Weiterverarbeitung.

Setzen:
- Antwort 1 → `Elfmeterschießen = nein`  
- Antwort 2/3 → `Elfmeterschießen = ja`


##### D) Ergebnis (automatisch, keine Rückfrage)

Priorität:

1) Wenn `Abbruchgrund` gesetzt:
   - enthält „Gegner hat aufgegeben“ → `Ergebnis = Sieg`
   - sonst → `Ergebnis = Niederlage`

2) Sonst, wenn `HeimT ≠ AuswärtsT`:
   - Wenn `Wo = Heim` und `HeimT > AuswärtsT` → `Ergebnis = Sieg`
   - Wenn `Wo = Auswärts` und `AuswärtsT > HeimT` → `Ergebnis = Sieg`
   - sonst → `Ergebnis = Niederlage`

3) Sonst (Tore gleich):
   - Wenn Elfer-Antwort = 2 → `Ergebnis = Sieg`
   - Wenn Elfer-Antwort = 3 → `Ergebnis = Niederlage`
   - Sonst → `Ergebnis = Unentschieden`


##### E) CleanSheet - autmatisch entscheiden, keine Rückfrage

CleanSheet wird berechnet, nicht entschieden.

Regel:
- Wenn `Ergebnis = Sieg` und `Gegentore = 0` → `CleanSheet = ja`
- Sonst → `CleanSheet = nein`

Gegentore:
- `Wo = Heim` → Gegentore = `AuswärtsT`
- `Wo = Auswärts` → Gegentore = `HeimT`

##### F) Daten Mein Team
- **HeimT** → Tore Heimteam
- **Schüsse** → Anzahl Schüsse
- **Schusspräzision** → Kreiswert über *SCHUSSPRÄZISION*
- **xGoals**
- **Ballbesitz**
- **Pässe**
- **Erfolgreiche Pässe** → Kreiswert über *PASSGENAUIGKEIT*
- **Erfolgreiche Dribblings** → Kreiswert über *DRIBBLING-ERFOLGSQUOTE*

##### G) Daten Gegner-Team
- **AuswärtsT** → Tore Auswärtsteams
- **SchüsseOpp**
- **SchusspräzisionOpp**
- **xGoalsOpponent**
- **PässeOpp**
- **Erfolgreiche Pässe Opp**

##### H) Verlängerung

- Wenn `Spieldauer > 119:00` bei Verlängerung "ja" notieren, sonst "nein"


#### Fragen zu Spielerstatistik und DataSpieler

##### VALIDIERUNG SPIELERLISTE (zwingend vor jeder Weiterverarbeitung):

1) Duplikaterkennung:
- Zwei Einträge gelten als Duplikat, wenn (Name + POS) identisch sind.
- Erkannte Duplikate werden zu einem Datensatz zusammengeführt.

2) Zielmenge:
- Nach Deduplizierung müssen exakt 18 eindeutige Spieler existieren.

Weitere Regeln: Wenn Bewertung "N.V." wird das Feld leer gelassen.

#### MOTM

### MOTM – zwingendes Text-Gate (HARTE LOGIK)

IF OCR_ROHDATEN.meta.gesamtwert_text != "Player of the Match" THEN
  - MOTM = "" für alle Spieler
  - ballIcon wird NICHT gelesen, NICHT ausgewertet, NICHT referenziert
  - ENDE MOTM-LOGIK (keine weiteren Regeln dieses Blocks anwenden)

ELSE
  - MOTM = "x" genau für Spieler mit ballIcon == "ja", sonst ""
  - Es darf exakt EIN Spieler MOTM = "x" haben
  - Wenn mehr als ein ballIcon == "ja" existiert:
      → MOTM = "" für alle Spieler
      → SYSTEMFEHLER: "Mehrfaches Ball-Icon bei aktivem MOTM-Gate"

END IF

Verbote:
- Kein Fallback
- Keine Schätzung
- Keine Ableitung aus Bewertung, Toren, Heatmap oder sonstigen UI-Elementen



###Phase 3 - Output im CSV Format

Jetzt werden die Tabellen für die Ausgabe gebaut.

CSV-Format:
Ein Datensatz = eine Zeile.
Felder durch Semikolon getrennt.
Keine Leerzeilen.
Kein Zusatztext vor oder nach der CSV.

Übertrage nun die Daten in die Tabellen und gib diese separat als Codeblock *ohne* Überschriften aus.

### Tabelle: DataSpiele

```
SpielID;
Spielkategorie;
Datum;
HeimT;
AuswärtsT;
Heim;
Auswärts;
Ergebnis;
Wo;
Ballbesitz;
Schüsse;
SchüsseOpp;
Schusspräzision;
SchusspräzisionOpp;
xGoals;
xGoalsOpponent;
Pässe;
PässeOpp;
Erfolgreiche Pässe;
Erfolgreiche Pässe Opp;
Erfolgreiche Dribblings;
Gelbe Karten;
Rote Karten;
Abbruchgrund;
Spieldauer;
CleanSheet;
Verlängerung;
Elfmeterschießen;
Unsicher/Fehlt
```

---

### Tabelle: DataSpieler

```
SpielID;
Spielkategorie;
Name;
Position;
Bewertung;
Tore;
Vorlagen;
MOTM;
Unsicher/Fehlt
```



---
