# The EAFC AI-Assisted Stats

The idea is inspired by the manual Excel tracking of EAFC Champs / Weekend League results created by Reddit user **u/xeque_279**. He kindly shared his Excel dashboard with me. Being the tracking/data nerd that I am, I used this as a starting point to create an AI-assisted version that works across multiple game modes.

If you like my work, you are free to use it. Feedback is appreciated via **[my Reddit Profile](https://www.reddit.com/user/Zuphael/)**.

If you want to tip via [Ko-fi](https://ko-fi.com/zuphael) that would be fantastic, but absolutely not required. I will buy Coffee (to keep me runnig) oder EAFC Points ;)

The prompt and the Excel file are released under the **Creative Commons Attribution–NonCommercial 4.0 International License (CC BY-NC 4.0)**.

---

## How to use it

### 1. Prepare the Excel template

You will need the Excel template provided here so you can paste the output generated as a CSV table.

Add the Names of the Player like you see it in the screen where you grab the data in EAFC. For Example Salah will Show up as "M. Salah". It is critical, that you use the exact same naming for the excel file to work. The easiest way to do it is after your first usage of the AI Prompt. There is a enough space for your Main Squad and Bench but I also added some lots for some lagacy players you may want to keep tracked to compare.

### 2. Create the screenshots after the match

You need **four screenshots**:

- Two screenshots of the **player statistics**
- Two screenshots of the **match statistics**

I recommend selecting the **first item/player** in the list for the first screenshot and the **last item/player** for the second one.

The prompt is designed to *merge* the data and avoid using duplicate entries.

If you are using a PlayStation 5 (or PS5 Pro), the easiest workflow is to export the screenshots either to a folder via the iOS app or directly into the ChatGPT app and then start the prompt.

### 3. Prepare the master prompt in your language

When copying the prompt from GitHub for the first time, use the **“Code” view**. In line 39, insert your club name into `"Your Clubname"`. This is important so the AI can clearly distinguish your team from the opponent.

### 4. Wait for the output

The AI will ask you which kind of match you played and for the match number.

If the match duration is unusually short, it will also ask for the reason.

### 5. Copy the data into Excel

The output will indicate which Excel tab to use.  
The format is CSV with `;` as the column separator.

Sometimes Excel detects this automatically. If not, use **Data → Text to Columns** to split the data correctly.

You can delete the topic/header row, but it will be ignored by the dashboard anyway. I intentionally do not suppress it so you can verify that the AI placed everything in the correct columns.

### 6. Process the next match

You can continue by adding the next four screenshots in the same chat.  
The AI will ask the initial questions again, but you can also answer them directly when uploading the screenshots.

---

## Known issues

### 1. MOTM sometimes falsely detected

In some cases, it is difficult to reliably recognize the football icon that indicates the **Man of the Match**. Since I play the German version, CAM is labeled **“ZOM”**. The letter **O** is sometimes misrecognized as a football icon.

### 2. AI performance degrades after several matches in the same chat

I don’t know exactly why, but after about 5–10 games, errors may start to appear. This is likely due to the AI’s tendency to “guess” expected outcomes over time.

If you notice this behavior, simply start a new chat.
