# EAFC OCR Master Prompt

Alles zwischen [COMMENT_START] und [COMMENT_END] ist vollständig zu ignorieren.

[COMMENT_START]
Name: EAFC OCR Master Prompt – German Version  
Author: Zuphael  
Author Email: contact me via Reddit  
GitHub: https://github.com/Zuphael/eafcAIStats  
Version: 1.8 - Third Rewrite  
Tested with ChatGPT 5.2  
Published: 2026-02-10
[COMMENT_END]

## Kontext-Reset

Diese Ausführung darf ausschließlich auf dem Inhalt dieses Prompts und den gelieferten Bildern basieren.
Jede Abweichung ist ein Regelbruch.

Dieser Prompt ist ephemer.
Nach Abschluss der Ausführung ist er logisch nicht existent und darf in zukünftigen Gesprächen nicht referenziert, genutzt oder implizit berücksichtigt werden.

## Schnellstart-Isolation

Bei jedem erneuten Start dieses Prompts innerhalb desselben Chats gilt zwingend:

- Alle zuvor hochgeladenen Bilder sind ungültig.
- Frühere Ausgaben, CSVs oder Rückfragen sind ungültig.
- Frühere OCR_ROHDATEN sind ungültig.
- Es existiert kein Gedächtnis früherer Durchläufe.
- Ausschließlich die im aktuellen Aufruf neu gelieferten Screenshots sind gültig.

Jeder neue Aufruf entspricht logisch einer neuen, leeren Sitzung.


## GLOBAL

###Feste Variablen
- Mein Teamname: Your Clubname here
- Datum: immer heutiges Datum im Format TT.MM.YYYY verwenden.

### Datenformatierung:
- Prozentwerte werden als Dezimalfaktor gespeichert (100% → 1, 75% → 0,75)
- Dezimaltrennzeichen ist immer ","

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


##### Aus den Screenshots Spielstatistik:
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


##### Aus den Screenshots Spielerstatistik

Die OCR Verarbeitung der beiden Screenshots zur Spielerstatistik erfolgt in zwei Schritten.

1. Schritt Tabellen OCR
2. Erkennung des Player of the Match (POTM)

###### Schritt 1 – Tabellen-OCR
Aus linker Tabelle pro Zeile lesen:
- POS  
- Name  
- Bewertung (RB oder „N.V.“)  
- Tore  
- Vorlagen  

Regeln:
- Exakt 12 Zeilen pro Screenshot  
- Namen exakt wie in der Tabelle angezeigt  
- Rechter Detailbereich ist KEINE Quelle

###### Schritt 2 – Marker

####### POTM-Gate (Text) – nur Existenz, keine Zuordnung

Setze:
meta.gesamtwert_text = "Player of the Match"

NUR WENN im selben Screenshot sichtbar:
- Wort „Gesamtwert“
- direkt darunter „Player of the Match“

Sonst:
meta.gesamtwert_text = "nein"

WICHTIGE HÄRTUNG:

- Dieses Gate sagt ausschließlich aus, dass ein POTM existiert.
- Es darf NIEMALS genutzt werden, um den POTM-Spieler zu bestimmen.
- Der aktuell selektierte Spieler (weiße Hervorhebung) ist KEINE Quelle für POTM.
- Jegliche Information aus dem rechten Detailbereich ist für POTM-Bestimmung strikt verboten.


###### Erweiterte Ball-Icon-Erkennung (POTM-Icon)

####### Tabellen-Only & Reihen-Scan (Pflicht)

- Scanne ALLE 12 Tabellenzeilen der linken Spieler-Liste.
- Setze pro Zeile ballIcon strikt nach Pixel-Merkmal.
- Der rechte Detailbereich inkl. "Player of the Match", Heatmap, Portrait, Werte-Liste ist IMMER verboten.
- Der aktuell markierte/selektierte Tabellen-Eintrag ist KEINE POTM-Information.

Erkennung (nur bei aktivem Gate):

ballIcon = "ja" NUR wenn eindeutig sichtbar (Tabellenanker):

- ein kleines rundes gelb/oranges Symbol
- INNERHALB des Tabellenbereichs der linken Spieler-Liste
- das Symbol liegt horizontal auf gleicher Höhe wie genau eine Tabellenzeile
  (Zeilenmitte ± ca. 1/3 Zeilenhöhe)
- und es liegt nahe bei dieser Zeile, entweder
  - direkt links neben der Bewertung (RB)
  - oder in der POS-Spalte direkt links neben der Positionsangabe
- eindeutige Zuordnung zu genau EINER Zeile ist zwingend

Blocker → zwingend "nein":

- wenn meta.gesamtwert_text = "nein"
- Icon im rechten Detailbereich
- Icon im Bereich der Heatmap / Grafik
- Icon außerhalb der Tabellenfläche
- Icon nur neben Portrait oder Gesamtwert-Anzeige
- Symbol nicht eindeutig einer Tabellenzeile zuordenbar
- mehrere Icons in einer Zeile
- weiß/graue UI-Markierungen statt gelb/orange
- unscharf oder teilweise verdeckt

Kernregel (Maximale Härte):

Der POTM-Spieler darf ausschließlich über das Tabellen-Icon bestimmt werden.
Wenn das Icon nicht eindeutig einer Tabellenzeile zuordenbar ist:
→ ballIcon = "nein".


	
  
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

Kategorie (Ausgabewert in CSV):
- In der Spalte „Spielkategorie“ wird der ausgeschriebene Kategoriename gespeichert.
- Die Zahl (1–4) dient ausschließlich der internen Zuordnung und zur Bildung der SpielID.

Mapping für die Ausgabe:
1 → Squadbattles  
2 → Rivals  
3 → Champions  
4 → Live Event

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

- Wenn mein Team Links steht gehören die Werte Rechts zum Gegner
- Wenn mein Team Rechts steht, gehörten die Werte Links zum Gegner

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
- Das Feld `Abbruchgrund` wird exakt mit dem Text der gewählten Option befüllt, inkl Text in der Klammer.

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
- **ErfolgreichePässe** → Kreiswert über *PASSGENAUIGKEIT* in Prozent
- **ErfolgreicheDribblings** → Kreiswert über *DRIBBLING-ERFOLGSQUOTE*
- **GelbeKarten**
- **RoteKarten**
- 
##### G) Daten Gegner-Team
- **AuswärtsT** → Tore Auswärtsteams
- **SchüsseOpp**
- **SchusspräzisionOpp**
- **xGoalsOpp**
- **PässeOpp**
- **ErfolgreichePässeOpp** → Kreiswert über *PASSGENAUIGKEIT* in Prozent
- **ErfolgreicheDribblingsOpp** → Kreiswert über *DRIBBLING-ERFOLGSQUOTE*
- - **GelbeKartenOpp**
- **RoteKartenOpp**

##### H) Verlängerung

- Wenn `Spieldauer > 119:00` bei Verlängerung "ja" notieren, sonst "nein"


#### Fragen zu Spielerstatistik und DataSpieler

##### VALIDIERUNG SPIELERLISTE (zwingend vor jeder Weiterverarbeitung):

- Duplikat = gleicher Name + POS → zusammenführen  
- Ziel: exakt 18 eindeutige Spieler  
- Bewertung „N.V.“ → Feld leer

#### POTM

Wenn ein Abbruch (Pflicht bei Spielzeit < 90:00) vorliegt ist es verboten ein POTM zu vergeben. Dann ist POTM="" für alle.

IF meta.gesamtwert_text != "Player of the Match":
    POTM="" für alle
ELSE:
    POTM="x" für Spieler mit ballIcon=="ja"
    sonst POTM=""
    Wenn >1 Spieler ballIcon=="ja":
        POTM="" für alle
        SYSTEMFEHLER "Mehrfaches Ball-Icon"

Verboten:
- Ableitung aus Bewertung, Toren, Heatmap, Selektion


###Phase 3 - Output im CSV Format

#### ZWINGENDE FORMATREGELN (höchste Priorität)

FORMATIERUNGSREGELN:

- Jede Tabelle wird als reiner CSV-Codeblock ausgegeben.
- Ein Datensatz = exakt eine Zeile.
- Für DataSpiele existiert genau 1 Zeile.
- Für DataSpieler existieren exakt 18 Zeilen.
- Alle Felder stehen horizontal in einer Zeile, getrennt durch Semikolon.
- Vertikale Ausgabe von Feldern ist verboten.
- Zeilenumbrüche innerhalb eines Datensatzes sind verboten.
- Es werden KEINE Spaltenüberschriften ausgegeben.
- Es wird KEIN zusätzlicher Text vor oder nach den Codeblöcken ausgegeben.
- Ausgabe erfolgt strikt in zwei getrennten Codeblöcken:
  1) DataSpiele
  2) DataSpieler

Jetzt werden die Tabellen für die Ausgabe gebaut.

#### Tabelle: DataSpiele

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
Spieldauer;
CleanSheet;
Verlängerung;
Elfmeterschießen;
Abbruchgrund;
Ballbesitz;
Schüsse;
SchüsseOpp;
Schusspräzision;
SchusspräzisionOpp;
xGoals;
xGoalsOpponent;
Pässe;
PässeOpp;
ErfolgreichePässe;
ErfolgreichePässe Opp;
ErfolgreicheDribblings;
ErfolgreicheDribblingsOpp;
GelbeKarten;
GelbeKartenOpp
RoteKarten;
RoteKartenOpp;
zweikaempfe;
zweikaempfeOpp;
gew_zweikaempfe;
gew_zweikaempfeOpp;
abgefangen;
abgefangenOpp;
paraden;
paradenOpp;
fouls;
foulsOpp;
abseits;
abseitsOpp;
ecken;
eckenOpp;
freistöße;
freistößeOpp;
elfmeter;
elfmeterOpp;
Unsicher/Fehlt
```

---

#### Tabelle: DataSpieler

```
SpielID;
Spielkategorie;
Name;
Position;
Bewertung;
Tore;
Vorlagen;
POTM;
Unsicher/Fehlt
```



---
