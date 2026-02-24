# EAFC OCR Master Prompt

Everything between [COMMENT_START] and [COMMENT_END] must be completely ignored.

[COMMENT_START]
Name: EAFC OCR Master Prompt – English Version  
Author: Zuphael  
Author Email: contact me via Reddit  
GitHub: https://github.com/Zuphael/eafcAIStats  
Version: 1.8 - Third Rewrite  
Translated by ChatGPT 5.2
Tested with ChatGPT 5.2  
Published: 2026-02-10
[COMMENT_END]

## Context Reset

This execution may rely exclusively on the content of this prompt and the provided images.
Any deviation is a rule violation.

This prompt is ephemeral.
After completion of the execution, it is logically non-existent and must not be referenced, reused, or implicitly considered in future conversations.

## Quickstart Isolation

On every new execution of this prompt within the same chat, the following applies strictly:

- All previously uploaded images are invalid.
- Previous outputs, CSVs, or follow-up questions are invalid.
- Previous OCR_RAW_DATA are invalid.
- There is no memory of earlier runs.
- Only the screenshots newly provided in the current invocation are valid.

Each new invocation corresponds logically to a new, empty session.

## GLOBAL

### Fixed Variables
- My team name: Momentum FC  
- Date: always use today’s date in format DD.MM.YYYY.

### Data Formatting:
- Percent values are stored as decimal factors (100% → 1, 75% → 0.75)
- Decimal separator is always ","

## PROCESS

The process is deterministic and strictly sequential.
All phases must be executed completely and in a fixed order.
No skipping, no advancing, no repetition without explicit instruction.

### START RULE

Execution always begins with Phase 1 and must proceed sequentially through Phase 3.
Even if data for Phase 2 is already present at prompt start, Phase 1 must be executed first.

Intermediate user output is *forbidden*.
Internal data objects (e.g. `OCR_RAW_DATA`) are passed exclusively to the next phase.

After completion of Phase 1, Phase 2 starts automatically and may ask necessary clarification questions to the user.

Only Phase 3 produces user-visible output.


### Phase 1 – OCR Detection

Pure OCR extraction.
No interpretation. No logic. No questions. No CSV.
Only reading is allowed, no reasoning.

Prompt start parameters (<Cat>,<No>) are logically present,
but may only be read or interpreted in Phase 2.
In Phase 1 they are inert.

#### Input:
- Exactly 4 screenshots
  - 2 screenshots → match statistics
  - 2 screenshots → player statistics

#### Task:
Read ALL visible data in raw form.

Phase 1 = pure text recognition.
Anything beyond reading is forbidden.


##### From the match statistics screenshots:
- Score
- Match time
- Team names (left / right)
- Circular values (%):
  - Possession
  - Dribbling success rate
  - Shot accuracy
  - Pass accuracy
- All team numbers:
  - Shots
  - xGoals
  - Passes
  - Tackles
  - Cards
  - etc.


##### From the player statistics screenshots

OCR processing of the two player statistics screenshots is performed in two steps.

1. Table OCR
2. Detection of Player of the Match (POTM)

###### Step 1 – Table OCR
From the left table, read per row:
- POS  
- Name  
- Rating (RB or “N.A.”)  
- Goals  
- Assists  

Rules:
- Exactly 12 rows per screenshot  
- Names exactly as displayed in the table  
- The right detail panel is NOT a source

###### Step 2 – Marker

####### POTM Gate (Text) – existence only, no assignment

Set:
meta.overall_text = "Player of the Match"

ONLY IF visible in the same screenshot:
- the word “Overall”
- directly below it “Player of the Match”

Otherwise:
meta.overall_text = "no"

IMPORTANT HARDENING:

- This gate only states whether a POTM exists.
- It must NEVER be used to determine the POTM player.
- The currently selected player (white highlight) is NOT a POTM source.
- Any information from the right detail panel is strictly forbidden for POTM determination.

###### Extended Ball Icon Detection (POTM Icon)

####### Table-Only & Row Scan (Mandatory)

- Scan ALL 12 table rows of the left player list.
- Set ballIcon per row strictly based on pixel characteristics.
- The right detail panel including "Player of the Match", heatmap, portrait, value list is ALWAYS forbidden.
- The currently highlighted/selected table entry is NOT POTM information.

Detection (only if gate is active):

ballIcon = "yes" ONLY if clearly visible (table anchor):

- a small round yellow/orange symbol
- WITHIN the table area of the left player list
- the symbol lies horizontally at the same height as exactly one table row
  (row center ± approx. 1/3 row height)
- and it is located close to this row, either
  - directly left of the rating (RB)
  - or in the POS column directly left of the position label
- unambiguous assignment to exactly ONE row is mandatory

Blockers → must be "no":

- if meta.overall_text = "no"
- icon in the right detail panel
- icon in the heatmap / graphic area
- icon outside the table area
- icon only next to portrait or overall rating display
- symbol not clearly assignable to a table row
- multiple icons in one row
- white/gray UI markers instead of yellow/orange
- blurred or partially obscured

Core Rule (Maximum Strictness):

The POTM player may be determined exclusively via the table icon.
If the icon cannot be unambiguously assigned to a table row:
→ ballIcon = "no".


### Format for OCR_RAW_DATA
```
OCR_RAW_DATA:{
"match":{
"score":"",
"match_time":"",
"teamL":"",
"teamR":"",
"circles":{
"possession":{"L":"","R":""},
"dribbling":{"L":"","R":""},
"shot":{"L":"","R":""},
"pass":{"L":"","R":""}
},
"values":{
"ball_recovery_s":{"L":"","R":""},
"shots":{"L":"","R":""},
"xGoals":{"L":"","R":""},
"passes":{"L":"","R":""},
"tackles":{"L":"","R":""},
"won_tackles":{"L":"","R":""},
"interceptions":{"L":"","R":""},
"saves":{"L":"","R":""},
"fouls":{"L":"","R":""},
"offsides":{"L":"","R":""},
"corners":{"L":"","R":""},
"free_kicks":{"L":"","R":""},
"penalties":{"L":"","R":""},
"yellow":{"L":"","R":""},
"red":{"L":"","R":""},
"defense_direct":{"L":"","R":""},
"defense_wide":{"L":"","R":""},
"defense_high":{"L":"","R":""},
"defense_attempted":{"L":"","R":""}
}
},
"players":[
{"ss":"","name":"","pos":"","rating":"","goals":"","assists":"","ballIcon":"yes/no"}
],
"meta":{"overall_text":""}
}```


Phase 1 Result:
Phase 1 produces exclusively the object `OCR_RAW_DATA`, which is not output but passed to Phase 2.

This object is the mandatory input for Phase 2.
It is not a final output, but internal raw data.


In Phase 1:
- no further processing  
- no normalization  
- no validation  
- no final output  
- only creation of `OCR_RAW_DATA`


PHASE_1_COMPLETE = true

### Phase 2 – Rule-Based Data Processing and Clarification

Phase 2 may only start if PHASE_1_COMPLETE == true

Phase 2 operates exclusively on the OCR_RAW_DATA generated in Phase 1 and optional prompt start parameters.
No new data may be created or assumed.

#### Clarification of Match Category and Match ID

Mandatory Gate:
Match category and match number must be clearly defined before any further Phase-2 processing.
Without valid values, Phase 2 must not continue.

Start Rule:
- If a valid `<Cat>,<No>` is present at prompt start → adopt it, gate fulfilled.
- Otherwise → ask for category/number and wait for a response.

Response Rule:
- A response in the format `<Cat>,<No>` is always considered category/number,
  provided both values are valid.
- After a valid response, the gate is fulfilled and Phase 2 continues.

Valid Categories:
1 = Squad Battles (SQB)
2 = Rivals (RIV)
3 = Champions (WLC)
4 = Live Event (LIV)

MatchID:
MatchID = <Abbreviation><three digits> (leading zeros).
Example: RIV054

Category (output value in CSV):
- In the column “Spielkategorie” the fully written category name is stored.
- The number (1–4) is used exclusively for internal mapping and for forming the MatchID.

Output Mapping:
1 → Squadbattles  
2 → Rivals  
3 → Champions  
4 → Live Event

As long as match category or match number are unresolved:
- no further Phase-2 rules may be applied
- no fields may be calculated
- no assumptions may be made
- exclusively wait for valid `<Cat>,<No>`


#### Questions Regarding Match Statistics and DataMatches

##### A) Match Location (Where) – automatic decision, no user question

- **Home** → my team is listed **on the left**
- **Away** → my team is listed **on the right**

Record in column “Wo” whether my team is Home or Away

- **Home** = name of left team
- **Away** = name of right team

- If my team is on the left, right-side values belong to the opponent
- If my team is on the right, left-side values belong to the opponent


##### B) Abandonment (Mandatory if match time < 90:00)

Trigger:  
- If `MatchDuration < 90:00` → abandonment must be clarified.

Question:
1) Opponent quit (win)  
2) I quit (loss)  
3) Connection issue (loss)

Rule:
- The answer is mandatory.
- Without an answer, no further processing.
- The field `AbandonmentReason` is filled exactly with the selected option text, including the text in parentheses.

##### C) Penalty Shootout (Mandatory if > 119:00 & Draw)

Trigger:  
- If `MatchDuration > 119:00` **and** `HomeGoals = AwayGoals` → outcome after extra time must be clarified.

Question:
1) Draw (no penalty shootout)  
2) I won the penalty shootout  
3) Opponent won the penalty shootout

Rule:
- The answer is mandatory. Without an answer, no further processing.

Set:
- Answer 1 → `PenaltyShootout = no`  
- Answer 2/3 → `PenaltyShootout = yes`


##### D) Result (automatic, no user question)

Priority:

1) If `AbandonmentReason` is set:
   - contains “Opponent quit” → `Result = Win`
   - otherwise → `Result = Loss`

2) Else, if `HomeGoals ≠ AwayGoals`:
   - If `Wo = Home` and `HomeGoals > AwayGoals` → `Result = Win`
   - If `Wo = Away` and `AwayGoals > HomeGoals` → `Result = Win`
   - otherwise → `Result = Loss`

3) Else (goals equal):
   - If penalty answer = 2 → `Result = Win`
   - If penalty answer = 3 → `Result = Loss`
   - Otherwise → `Result = Draw`


##### E) Clean Sheet – automatic decision, no user question

CleanSheet is calculated, not decided.

Rule:
- If `Result = Win` and `ConcededGoals = 0` → `CleanSheet = yes`
- Otherwise → `CleanSheet = no`

ConcededGoals:
- `Wo = Home` → ConcededGoals = `AwayGoals`
- `Wo = Away` → ConcededGoals = `HomeGoals`


##### F) My Team Data
- **HomeGoals** → goals of the home team
- **Shots** → number of shots
- **ShotAccuracy** → circular value for *SHOT ACCURACY*
- **xGoals**
- **Possession**
- **Passes**
- **SuccessfulPasses** → circular value for *PASS ACCURACY* in percent
- **SuccessfulDribbles** → circular value for *DRIBBLING SUCCESS RATE*
- **YellowCards**
- **RedCards**


##### G) Opponent Team Data
- **AwayGoals** → goals of the away team
- **ShotsOpp**
- **ShotAccuracyOpp**
- **xGoalsOpp**
- **PassesOpp**
- **SuccessfulPassesOpp** → circular value for *PASS ACCURACY* in percent
- **SuccessfulDribblesOpp** → circular value for *DRIBBLING SUCCESS RATE*
- **YellowCardsOpp**
- **RedCardsOpp**


##### H) Extra Time

- If `MatchDuration > 119:00`, record "yes" for extra time, otherwise "no"

#### Questions Regarding Player Statistics and DataPlayers

##### PLAYER LIST VALIDATION (mandatory before any further processing):

- Duplicate = same Name + POS → merge  
- Target: exactly 18 unique players  
- Rating “N.A.” → field left empty


#### POTM

If an abandonment (mandatory if match time < 90:00) is present, it is forbidden to assign a POTM.
Then POTM = "" for all.

IF meta.overall_text != "Player of the Match":
    POTM = "" for all
ELSE:
    POTM = "x" for player with ballIcon == "yes"
    otherwise POTM = ""
    If more than one player has ballIcon == "yes":
        POTM = "" for all
        SYSTEM ERROR "Multiple Ball Icons"

Forbidden:
- Derivation from rating, goals, heatmap, selection


### Phase 3 – Output in CSV Format

#### MANDATORY FORMAT RULES (highest priority)

FORMATTING RULES:

- Each table is output as pure CSV code block.
- One record = exactly one line.
- For DataMatches exactly 1 line exists.
- For DataPlayers exactly 18 lines exist.
- All fields are horizontal in one line, separated by semicolons.
- Vertical field output is forbidden.
- Line breaks within a record are forbidden.
- NO column headers are output.
- NO additional text before or after the code blocks.
- Output is strictly in two separate code blocks:
  1) DataMatches
  2) DataPlayers

Now the tables for output are built.


#### Table: DataMatches

´´´
MatchID;
MatchCategory;
Date;
HomeGoals;
AwayGoals;
Home;
Away;
Result;
Wo;
MatchDuration;
CleanSheet;
ExtraTime;
PenaltyShootout;
AbandonmentReason;
Possession;
Shots;
ShotsOpp;
ShotAccuracy;
ShotAccuracyOpp;
xGoals;
xGoalsOpponent;
Passes;
PassesOpp;
SuccessfulPasses;
SuccessfulPassesOpp;
SuccessfulDribbles;
SuccessfulDribblesOpp;
YellowCards;
YellowCardsOpp;
RedCards;
RedCardsOpp;
tackles;
tacklesOpp;
won_tackles;
won_tacklesOpp;
interceptions;
interceptionsOpp;
saves;
savesOpp;
fouls;
foulsOpp;
offsides;
offsidesOpp;
corners;
cornersOpp;
free_kicks;
free_kicksOpp;
penalties;
penaltiesOpp;
Uncertain/Missing
´´´

---

#### Table: DataPlayers
´´´
MatchID;
MatchCategory;
Name;
Position;
Rating;
Goals;
Assists;
POTM;
Uncertain/Missing
´´´
