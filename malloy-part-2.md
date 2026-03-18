# Malloy Lab Guide: Exploring ICE Arrests Data

## Overview

In this lab you will learn how to extend a Malloy data source by adding **dimensions**, **measures**, and **views**. You will also learn how to **nest** views inside each other, apply **visualizations** like bar charts, and use **tooltips** to create interactive data explorations.

We are working with a dataset of ICE/ERO arrest records stored as a parquet file.

---

## Part 1: Your Starting Source

Open your `.malloy` file. You should have this as your base source code:

```malloy
source: arrests is duckdb.table('data/arrests-latest.csv') extend {


}
```

All of the code we write in this lab will go **between the opening `{` and closing `}`** of this source block. Each time you add new code, paste it below the previous code you added, but always **before the final `}`**.

Click **Preview** on your source after each step to see the data and confirm everything works.

---

## Part 2: Adding Simple Dimensions

Dimensions are new columns that you derive from existing columns in the data. They don't change the underlying data — they just give you new ways to slice and view it.

**Copy and paste the following code inside your source block (before the closing `}`):**

```malloy
  // --- DIMENSIONS ---

  // Extract just the year from the apprehension date
  dimension: apprehension_year is year(apprehension_date)

  // A true/false flag: was this person a convicted criminal?
  dimension: is_convicted_criminal is apprehension_criminality = '1 Convicted Criminal'

  // Was the person deported back to their home country?
  dimension: deported_to_home_country is citizenship_country = departure_country

  // Uppercase cleanup of gender field
  dimension: gender_clean is upper(gender)
```

Your full source should now look like this:

```malloy
source: arrests is duckdb.table('data/arrests-latest.parquet') extend {

  // --- DIMENSIONS ---

  dimension: apprehension_year is year(apprehension_date)
  dimension: is_convicted_criminal is apprehension_criminality = '1 Convicted Criminal'
  dimension: deported_to_home_country is citizenship_country = departure_country
  dimension: gender_clean is upper(gender)

}
```

**Click Preview.** Scroll right in the results table. You should see your four new columns at the end: `apprehension_year`, `is_convicted_criminal`, `deported_to_home_country`, and `gender_clean`. These columns didn't exist in the original data — you just created them.

---

## Part 3: Adding Measures

Measures are aggregations — they summarize your data. Unlike dimensions (which appear once per row), measures collapse many rows into a single value, like a count or an average.

**Paste the following code inside your source block, below your dimensions but before the closing `}`:**

```malloy
  // --- MEASURES ---

  // Count of all arrest records
  measure: arrest_count is count()

  // Count of country values (non-null)
  measure: country_count is count(citizenship_country)

  // Percentage of records that are convicted criminals
  measure: pct_criminal is count() { where: is_convicted_criminal = true } / count() * 100
```

---

## Part 4: Writing Simple Views

You can't see measures just by clicking Preview on the source — measures need to be used inside a **view**. A view tells Malloy *how* to group and aggregate the data.

A view has two main parts:

- **`group_by`** — which dimension(s) to break the data into groups by. Think of it like the row labels in a pivot table.
- **`aggregate`** — which measure(s) to calculate for each group.

**Paste the following views inside your source block, below your measures but before the closing `}`:**

```malloy
  // --- VIEWS ---

  // Arrests grouped by state
  view: by_state is {
    group_by: apprehension_state
    aggregate:
      arrest_count
      pct_criminal
  }

  // Arrests grouped by citizenship country
  view: by_country is {
    group_by: citizenship_country
    aggregate:
      arrest_count
      country_count
  }

  // Arrests grouped by year
  view: by_year is {
    group_by: apprehension_year
    aggregate:
      arrest_count
      pct_criminal
  }
```

**To run a view:** In the Malloy Explorer, click on the name of a view (like `by_state`) to run it. You should see a table with one row per state, showing the arrest count and criminal percentage for each.

Try running `by_country` and `by_year` as well.

**What just happened?**

- `group_by: apprehension_state` told Malloy to create one row for each unique state.
- `aggregate: arrest_count` told Malloy to count the records in each group.
- `aggregate: pct_criminal` told Malloy to calculate the criminal percentage for each group.

---

## Part 5: Nesting Views

Nesting is one of Malloy's most powerful features. When you **nest** a view inside another view, you get a sub-table for each row of the outer view. For example, nesting `by_country` inside `by_state` gives you a list of countries *within each state*.

### Option A: Nesting with Code

**Paste this new view inside your source block, below your other views but before the closing `}`:**

```malloy
  // A nested view: for each state, show a breakdown by country
  view: state_with_countries is {
    group_by: apprehension_state
    aggregate: arrest_count
    nest: by_country
  }
```

Run the `state_with_countries` view. You should see a table where each state row contains an embedded sub-table showing the country breakdown for that state.

### Option B: Nesting from the Explorer GUI

You can also nest views interactively:

1. In the Explorer, run `by_state`.
2. Then look for the option to **add a nested query** — click on `by_country` to nest it inside.

Both approaches produce the same result. The code approach is useful when you want to save and reuse the view.

---

## Part 6: Visualizing with Bar Charts

Malloy lets you turn any view into a visualization by adding a **renderer tag** above the view definition. The tag `# bar_chart` goes on the line *before* the `view:` keyword.

**Paste this view inside your source block, before the closing `}`:**

```malloy
  // Bar chart of arrests by state
  # bar_chart
  view: state_bar_chart is {
    group_by: apprehension_state
    aggregate: arrest_count
    limit: 15
  }
```

Note that `# bar_chart` goes **above** the `view:` line — not inside the curly braces.

Run `state_bar_chart`. Instead of a plain table, you should now see a horizontal bar chart showing the top 15 states by arrest count.

The `limit: 15` keeps the chart readable by only showing the top 15 groups. Try changing the number or removing the limit to see what happens.

---

## Part 7: Interactive Tooltips with Nested Bar Charts

Now let's combine nesting and visualization. You can use the `# tooltip` tag on a nested view so that it appears as a **popup bar chart** when you hover over a section of the outer chart.

**Paste this view inside your source block, before the closing `}`:**

```malloy
  // Bar chart with tooltip: hover over a state bar to see country breakdown
  # bar_chart
  view: state_chart_with_tooltip is {
    group_by: apprehension_state
    aggregate: arrest_count
    limit: 15
    # tooltip bar_chart size=sm
    nest: country_tooltip is {
      group_by: citizenship_country
      aggregate: arrest_count
      limit: 10
    }
  }
```

Run `state_chart_with_tooltip`. You will see the same bar chart of states as before — but now, **mouse over any bar** and a popup bar chart will appear showing the top 10 citizenship countries for that state.

**What's happening here?**

- `# bar_chart` above the view makes the outer query render as a bar chart grouped by state.
- The `nest:` creates a sub-query for each state.
- `# tooltip bar_chart size=sm` goes on the line before the `nest:` keyword. It tells Malloy to show the nested view as a small hover popup bar chart instead of an inline table.

---

## Your Final Source

After completing all sections, your full source file should look like this:

```malloy
source: arrests is duckdb.table('data/arrests-latest.parquet') extend {

  // --- DIMENSIONS ---

  dimension: apprehension_year is year(apprehension_date)
  dimension: is_convicted_criminal is apprehension_criminality = '1 Convicted Criminal'
  dimension: deported_to_home_country is citizenship_country = departure_country
  dimension: gender_clean is upper(gender)

  // --- MEASURES ---

  measure: arrest_count is count()
  measure: country_count is count(citizenship_country)
  measure: pct_criminal is count() { where: is_convicted_criminal = true } / count() * 100

  // --- VIEWS ---

  view: by_state is {
    group_by: apprehension_state
    aggregate:
      arrest_count
      pct_criminal
  }

  view: by_country is {
    group_by: citizenship_country
    aggregate:
      arrest_count
      country_count
  }

  view: by_year is {
    group_by: apprehension_year
    aggregate:
      arrest_count
      pct_criminal
  }

  view: state_with_countries is {
    group_by: apprehension_state
    aggregate: arrest_count
    nest: by_country
  }

  # bar_chart
  view: state_bar_chart is {
    group_by: apprehension_state
    aggregate: arrest_count
    limit: 15
  }

  # bar_chart
  view: state_chart_with_tooltip is {
    group_by: apprehension_state
    aggregate: arrest_count
    limit: 15
    # tooltip bar_chart size=sm
    nest: country_tooltip is {
      group_by: citizenship_country
      aggregate: arrest_count
      limit: 10
    }
  }

}
```

---

## What You Learned

- **Dimensions** create new columns derived from existing data.
- **Measures** are aggregations that summarize data (counts, percentages, etc.).
- **Views** combine `group_by` and `aggregate` to query the data.
- **Nesting** puts a view inside another view, giving you sub-tables for each group.
- **`# bar_chart`** above a view turns it into a bar chart visualization.
- **`# tooltip bar_chart size=sm`** above a nested view makes it appear as an interactive popup bar chart on hover.
