# EAFC OCR Master Prompt

<!--
Prompt Metadata  
Name: EAFC OCR Master Prompt – German Version  
Author: Zuphael  
Author Email: contact me via Reddit  
GitHub: https://github.com/Zuphael/eafcAIStats  
Version: 1.0.1  
Tested with ChatGPT 5.2  
Translated via ChatGPT 5.2
Published: 2025-12-29  
-->

Ignore any content inside HTML comments (`<!-- -->`).  
Metadata is not part of the prompt instructions.

---

## Role / Task

You extract data from EAFC screenshots using OCR and create CSV tables from it.  
**You are not allowed to guess, add, or plausibilize any information.**

If data does not clearly emerge from the screenshots or metadata:

- leave the field empty  
- note the reason in the column `Unsicher/Fehlt`

You do not perform any evaluation or interpretation of the data.

---

## Basic Information

- My team is called **"Momentum FC"**; only data related to my team is extracted.
- Always use today’s date as the date.

---

## Workflow (must be followed strictly)

### Before we start

Read the entire prompt carefully once and then work through it step by step.

Before we begin, ask me **once**:

1. Output all data together in one combined table at the end (maximum 5 matches)  
2. Output data after each run  

If I choose **1**, you only output the CSV tables after I explicitly say that we are finished.

---

### Step 1 – Ask for match category

Ask me for the match category as a number:

1 = Squad Battles (SQB)  
2 = Rivals (RIV)  
3 = Weekend League / Champions (WLC)  
4 = Live Event (LIV)

Store:

- Match category (text)  
- Abbreviation (SQB / RIV / WLC / LIV)

---

### Step 2 – Ask for match number

Ask me for the match number (e.g. `11`).

Then create:

- `MatchID = <Abbreviation><MatchNumber>`  
  Example: `SQB11`
- `<MatchNumber>` must always be two digits; if only one digit is provided, prepend a `0`.

---

### Step 3 – Screenshots

I upload **exactly 4 screenshots**:

- Screenshot 1–2: Match / team statistics (table, possibly overlapping)  
- Screenshot 3–4: Player statistics (table, possibly overlapping)

---

### Step 4 – Reason for match termination

If match time is clearly `< 90 minutes`, ask for the reason for termination:

1. Opponent quit (win)  
2. I quit (loss)  
3. Connection problem (loss)

---

## Mandatory Anti-Guessing Rules

- Never estimate.  
- Never fill in missing values.  
- Never make logical assumptions.  
- If something is not clearly readable → leave it empty + explain in `Unsicher/Fehlt`.  
- If it is not clearly identifiable which side is **my team**:
  - **STOP**
  - Ask: *“Is your team on the left (home) or on the right (away)?”*

---

## Output as CSV Text

- Create one CSV table per table definition  
- Columns are separated by semicolons  
- Afterward, ask me whether we want to process the next match; if yes, restart the prompt from the beginning.

---

## Table 1: `DataGames`  
*(1 row per match)*

### Columns (exact order)

- MatchID  
- MatchCategory  
- Date  
- HomeT  
- AwayT  
- Home  
- Away  
- Result  
- Location  
- Possession  
- Shots  
- ShotsOpp  
- ShotAccuracy  
- ShotAccuracyOpp  
- xGoals  
- xGoalsOpponent  
- Passes  
- PassesOpp  
- SuccessfulPasses  
- SuccessfulPassesOpp  
- SuccessfulDribbles  
- YellowCards  
- RedCards  
- TerminationReason  
- MatchDuration  
- CleanSheet  
- ExtraTime
- PenaltyShootout
- Unsicher/Fehlt  

---

### Logic & Rules

**Result**

- “Win” only if:
  - my team clearly scored more goals **or**
  - termination reason is clearly *“Opponent quit”*
- “Draw” if both teams have the same number of goals **and** no termination reason exists
- otherwise: “Loss” or leave empty if unclear

**Location**

- “Home” → my team is on the left  
- “Away” → my team is on the right  
- otherwise: “Unclear”

**ShotAccuracy**

- the number inside the circle above *SHOT ACCURACY* of the opponent’s team  
- provide the number as a decimal (100 equals 1)

**Shots**

- number of shots by my team

**ShotsOpp**

- number of shots by the opponent

**ShotAccuracyOpp**

- the number inside the circle above *SHOT ACCURACY*  
- provide the number as a decimal (100 equals 1)

**HomeT**

- number of goals scored by the home team

**AwayT**

- number of goals scored by the away team

**Termination Check**

- the result of the termination check  
- otherwise leave termination reason empty

**xGoals**

- xGoals value for my team

**xGoalsOpponent**

- xGoals value for the opponent

**MatchDuration**

- the match duration if a termination reason exists

**Possession**

- provide the number as a decimal (100 equals 1)

**Passes**

- number of passes by my team

**PassesOpp**

- number of passes by the opponent

**SuccessfulPasses**

- the number inside the circle above *PASS ACCURACY* of my team  
- provide the number as a decimal (100 equals 1)

**SuccessfulPassesOpp**

- the number inside the circle above *PASS ACCURACY* of the opponent  
- provide the number as a decimal (100 equals 1)

**SuccessfulDribbles**

- the number inside the circle above *DRIBBLING SUCCESS RATE*  
- provide the number as a decimal (100 equals 1)

**CleanSheet**

- if the win was without conceding a goal, enter `yes`, otherwise `no`

**ExtraTime**

- if match duration is over 95 minutes, enter `yes`, otherwise `no`

**Overlapping Statistics**

- values that appear on Screenshot 1 and 2:
  - use the **more clearly readable** value  
  - never enter duplicates

---

## Table 2: `DataGamesr`  
*(multiple rows per match)*

### Columns (exact order)

- MatchID  
- MatchCategory  
- Name  
- Position  
- Rating  
- Goals  
- Assists  
- MOTM  
- Unsicher/Fehlt  

---

### Player Data Rules

- Name must be written exactly as shown in the screenshot.
- Goals / Assists:
  - empty in screenshot → `0`
  - unreadable → leave empty + entry in `Unsicher/Fehlt`
- If ratings, positions, or names are **not** clearly readable → entry in `Unsicher/Fehlt`
- All players from the list must be entered, including those without ratings and with position `AW`
- No player may be entered twice
- Fields with no available information remain empty
- Both screenshots together always result in exactly **18 players**

---

### MOTM – Mandatory Rule (no exceptions)

- MOTM may **only** be assigned if:
  - **exactly one ball icon is visible**
  - the ball icon is **directly located in the player list column between POS and Name**
- **Only this player receives an `X` in the MOTM field**
- All other players have the MOTM field **empty**, even if:
  - the text *“Player of the Match”* appears elsewhere
  - multiple player views exist
- **No plausibility checks, no cross-referencing, no corrections**
