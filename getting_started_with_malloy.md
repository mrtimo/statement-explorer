# Getting Started with Malloy in VS Code

A step-by-step guide for exploring data using Malloy, DuckDB, and Claude.

---

## Step 1: Download and Install VS Code

- Go to [https://code.visualstudio.com](https://code.visualstudio.com) and download VS Code for your operating system (Windows, Mac, or Linux).
- Run the installer and accept the defaults.

---

## Step 2: Download Your Data File

- Download the `.csv` file provided for this assignment (your instructor will share a link).
- Take note of where it saved — usually your **Downloads** folder.

---

## Step 3: Open the File in Excel and Re-Save as UTF-8 CSV

CSV files can use different text encodings. Malloy and DuckDB work best with **UTF-8**, which supports all characters cleanly.

1. Open the `.csv` file in **Microsoft Excel**.
2. Go to **File → Save As**.
3. In the "Save as type" dropdown, choose **CSV UTF-8 (Comma delimited) (*.csv)**.
4. Save the file. You can overwrite the original or give it a new name.

> **Why this matters:** If your CSV isn't saved as UTF-8, you may see garbled characters or errors when Malloy tries to read it. This step prevents that.

---

## Step 4: Create a Project Folder

Create a new folder somewhere convenient (e.g., your Desktop or Documents folder). Give it a clear name like:

```
malloy-project
```

This will be your **workspace** — everything for this project lives here.

---

## Step 5: Create a "data" Subfolder and Move Your CSV Into It

1. Inside your project folder, create a new folder called **`data`**.
2. Move (or copy) your UTF-8 CSV file into the `data` folder.

Your folder structure should now look like this:

```
malloy-project/
  └── data/
        └── your-file.csv
```

---

## Step 6: Open the Project Folder in VS Code

1. Open VS Code.
2. Go to **File → Open Folder...** (on Mac: **File → Open...**).
3. Select your `malloy-project` folder and click **Open**.

You should see your folder and the `data` subfolder in the **Explorer** panel on the left.

> **Important:** Always open the *project folder*, not just a single file. Malloy needs to resolve file paths relative to the project root.

---

## Step 7: Install the Malloy Extension

1. Click the **Extensions** icon on the left sidebar (it looks like four small squares).
2. Search for **Malloy**.
3. Click **Install** on the Malloy extension (published by Malloy).
4. Wait for it to finish installing. You may see a prompt to reload VS Code — go ahead and reload if asked.

---

## Step 8: Create Your First Malloy File

1. In the Explorer panel, right-click in the empty space of your project folder and choose **New File**.
2. Name it something like **`explore.malloy`** (the `.malloy` extension is required).
3. Type in your first source definition. For example:

```malloy
source: arrests is duckdb.table("data/arrests-latest.csv") extend {
}
```

Replace `arrests-latest.csv` with the actual name of your CSV file.

4. **Save the file** (Ctrl+S / Cmd+S).

---

## Step 9: Explore the Malloy Interface

After saving, you should see some new features appear in VS Code:

- **Preview** — hover over or click on a source name to see a preview of the data.
- **Schema** — a panel showing all the columns (fields) in your data, along with their types.
- **Explore** — opens an interactive query builder where you can drag and drop fields.

Click around and get familiar with your data. Look at the column names, the data types, and a few rows of actual values. This is how you'll figure out what questions to ask.

---

## Step 10: Use Claude to Help You Write Malloy Code

Now that you know what your data looks like, head to [https://claude.ai](https://claude.ai) and create a free account if you don't already have one.

Give Claude a prompt like this:

> I'm working with data about ICE arrests loaded into Malloy. Here is a preview of the first few lines of data:
>
> *(paste a few rows from the Preview here)*
>
> And here is my current Malloy code:
>
> ```malloy
> source: arrests is duckdb.table("data/arrests-latest.csv") extend {
> }
> ```
>
> Can you give me back my Malloy code with some basic measures added (like a count and any useful aggregations), plus a couple of interesting views I can run?

### Tips for a good prompt:

- **Include actual data.** Paste in the column names and a few sample rows so Claude knows what it's working with.
- **Include your current code.** Even if it's just the bare source, this gives Claude the right starting point.
- **Say what you're curious about.** If you have a question about the data ("Which states have the most arrests?"), mention it — Claude can write a view for that.

---

## Step 11: Replace Your Code with Claude's Response

1. Copy the Malloy code that Claude gives you.
2. Go back to VS Code and select all the text in your `explore.malloy` file (Ctrl+A / Cmd+A).
3. Paste the new code (Ctrl+V / Cmd+V).
4. Save the file.

You should see **Run** buttons appear above each `query:` block. Click one to execute it and see results.

---

## Step 12: Read the Results and Iterate

- Look at what the query returned. Does it make sense? Is anything surprising?
- If something doesn't work or you want to change it, go back to Claude and say something like:

> This query ran but the results don't look right — I think the date field needs to be cast. Here's the error message: *(paste the error)*

  or:

> This is great. Can you add a view that breaks this down by month and shows a trend over time?

The loop is: **run → read → ask Claude → paste → run again**. Each cycle, you learn a little more about both the data and the language.

---

## Step 13: Experiment on Your Own

Once you've seen a few working examples, try modifying the code yourself:

- Change a `group_by` field to see a different breakdown.
- Add a `where:` clause to filter the data.
- Change `# bar_chart` to `# line_chart` to see a different visualization.
- Create a new `view:` by copying an existing one and tweaking it.

You don't have to write everything from scratch — reading and modifying working code is one of the best ways to learn.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Malloy shows red underlines or errors | Check that your CSV filename in the `source:` line exactly matches the actual filename in your `data/` folder (capitalization matters). |
| Strange characters in your data | Re-do Step 3 — make sure you saved as **CSV UTF-8**, not just "CSV". |
| No Run/Preview buttons appear | Make sure you saved the file with the `.malloy` extension and that the Malloy extension is installed. |
| "File not found" error | Make sure you opened the *project folder* in VS Code (Step 6), not just the `.malloy` file by itself. |
| Query runs but returns nothing | Try a simpler query first (just a count). Your `where:` clause might be filtering out all rows. |

---

## Summary

Here's what you just did:

1. Set up a clean project structure with a `data/` folder.
2. Connected Malloy to a CSV file via DuckDB (no database setup required).
3. Used Claude to generate working Malloy code with measures and views.
4. Ran real queries against real data — all inside VS Code.

This is the same basic workflow used in professional data analysis: get the data, explore it, ask questions, refine. The tools are just friendlier.
