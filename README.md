Power BI Sales Orders — Data Cleaning, Transformation & Modeling
Project overview
This project demonstrates an end-to-end Power BI workflow using three CSV files: List of Orders.csv, Order Details.csv, and Sales target.csv. The work includes importing and cleaning data in Power Query, creating calculated columns, merging tables, detecting missing and duplicate records, sorting/filtering, aggregating results, and building data-model relationships.
Scope: This README documents how to reproduce the assignment. It does not claim particular output totals or counts that have not been verified against the actual CSV files. Screens and menu labels may vary slightly between Power BI Desktop versions.

Data files and expected fields
File	Main fields used	Purpose
List of Orders.csv	Order ID, Order Date, CustomerName, City, State	Order-level information
Order Details.csv	Order ID, Amount, Profit, Quantity, Category, Sub-Category	Sales line details
Sales target.csv	Category, Target, and a month/date field (often Month of Order Date)	Sales targets


Preparation: Install Power BI Desktop and keep the three CSV files in an accessible folder. Match column names to those in your files; for example, CustomerName may be written with or without a space. Save your final report as a .pbix file.
1. Import all three CSV files
1. Open Power BI Desktop.
2. Select Home → Get data → Text/CSV.
3. Browse to List of Orders.csv, select it, and click Open.
4. In the preview window select Transform Data (rather than loading directly). This opens Power Query Editor.
5. In Power Query Editor choose Home → New Source → Text/CSV.
6. Select Order Details.csv and accept the preview (the confirmation button might be OK or Transform Data).
7. Repeat Home → New Source → Text/CSV for Sales target.csv.
8. Verify that all three queries appear in the Queries pane on the left.
Check: Three separate queries should exist: List of Orders, Order Details, and Sales target.
2. Restrict List of Orders to the first 500 rows
1. Select List of Orders in Power Query Editor.
2. Choose Home → Keep Rows → Keep Top Rows.
3. Enter 500, then select OK.
4. Inspect the Applied Steps pane: a Kept First Rows step should be present.
Power Query M example:
= Table.FirstN(#"Previous Step", 500)
Why: This limits this query to the first 500 rows in its current order. If you need a particular selection of 500 rows, sort explicitly before keeping the top rows. This step does not automatically limit other queries.
3. Set Order Date to the Date type
1. In List of Orders, select the Order Date column.
2. Right-click the column header and select Change Type → Using Locale….
3. Set Data Type = Date and Locale = English (United Kingdom) if the CSV has dates such as 13-04-2018 (DD-MM-YYYY).
4. Select OK and confirm that dates convert without errors.
Why use locale? A date like 13-04-2018 cannot be parsed as US-style MM-DD-YYYY, because month 13 is invalid. Even ambiguous dates (for example 05-12-2018) could be silently read as the wrong month/day. If an earlier automatic Changed Type step already caused errors, edit or remove that step before applying the correct locale.

Check: The Order Date header has a date-type icon and Column Quality shows no unexpected errors.
4. Change Amount and Target to Fixed Decimal Number
1. Select Order Details, then select Amount.
2. Use Transform → Data Type → Fixed Decimal Number (or the type icon beside the column header).
3. Select Sales target and repeat for Target.
Why: The fixed decimal/currency data type is appropriate for amounts requiring consistent four-decimal-place underlying precision. Check the locale if text values contain currency symbols or separators that prevent conversion.

Check: Both columns show a fixed decimal/currency-type icon and contain no unexpected conversion errors.
5. Format CustomerName in proper case
1. Open List of Orders and select CustomerName.
2. Choose Transform → Format → Capitalize Each Word.
3. Review names to ensure special capitalization has not been incorrectly changed.
Power Query M example:
= Table.TransformColumns(
    #"Previous Step",
    {{"CustomerName", Text.Proper, type text}}
)
Example: aNJaLi shaRMa becomes Anjali Sharma. Special names, acronyms, and brands may need manual corrections after applying proper case.
6. Merge City and State into Location
1. In List of Orders, select the City column, then Ctrl-click State in that order.
2. Choose Transform → Merge Columns. (Choose Add Column → Merge Columns instead if you need to retain both original columns for later filtering/modeling.)
3. Choose Custom separator and type ,  (comma followed by a space).
4. Enter new column name Location and click OK.
5. Verify values such as Chennai, Tamil Nadu.
Power Query M example, retaining original columns:
= Table.AddColumn(
    #"Previous Step",
    "Location",
    each Text.Combine({[City], [State]}, ", "),
    type text
)
Important: The assignment later requires filtering by State. If you replace City and State with Location, the original State column will no longer be available. Prefer Add Column → Merge Columns to preserve State (or split Location again later). Check missing City/State values before concatenating.
7. Create Profit Margin in Order Details
1. Select Order Details.
2. Choose Add Column → Custom Column.
3. Enter New column name: Profit Margin.
4. Enter:
if [Amount] = null or [Amount] = 0 then null else [Profit] / [Amount]
5. Click OK.
6. Select the new column and change its data type to Percentage.
Formula:

\(\text{Profit Margin (\%)} = \frac{\text{Profit}}{\text{Amount}} \times 100\)

Example: Profit = 200, Amount = 1,000 → margin = 0.20, displayed as 20%. Negative margins reflect losses. Do not multiply by 100 in the custom column if you format the resulting fraction as Percentage.

Analysis note: For an overall profit margin, divide total profit by total amount. Do not average individual row margins unless that is explicitly the metric requested.
8. Create Profit Status as a conditional column
1. Select Order Details.
2. Choose Add Column → Conditional Column.
3. Enter new column name Profit Status.
4. Define the conditions in this order:
   Condition on Profit	Output
   is less than 0	Loss
   equals 0	Break-Even
   Otherwise	Profit
5. Select OK and confirm the labels.
Equivalent custom column M expression:
if [Profit] = null then "Unknown"
else if [Profit] < 0 then "Loss"
else if [Profit] = 0 then "Break-Even"
else "Profit"
This handles missing Profit as Unknown, rather than incorrectly classifying it as Profit.
9. Merge List of Orders and Order Details as Orders Data
1. In Power Query Editor choose Home → Merge Queries → Merge Queries as New.
2. Select List of Orders as the first table and Order Details as the second.
3. Click Order ID in each preview to specify the matching key.
4. Choose Left Outer (all from first, matching from second) and select OK.
5. In the resulting query, locate the nested Order Details column where cells show Table.
6. Click the expand icon on its header.
7. Choose the relevant detail columns, such as Amount, Profit, Quantity, Category, Sub-Category, Profit Margin, and Profit Status. Exclude the second Order ID because the first table already has it.
8. Optionally uncheck Use original column name as prefix and click OK.
9. In Query Settings → Properties → Name, rename the new query to Orders Data.
Why Left Outer? It preserves all retained orders from the first table and brings in matching order-detail rows. If an order has several line items, expanding will result in several rows for that Order ID. Thus, Orders Data can have more than 500 rows, even though List of Orders was limited to 500.

Check: Verify expected columns, nulls in expanded detail fields, and the number of matching/unmatched Order IDs. A nested table cell showing Table does not alone prove that a match exists.
10. Identify and handle missing data
Find missing values
1. In Power Query Editor open View.
2. Enable Column Quality, Column Distribution, and Column Profile.
3. At the bottom-left, change profiling from Based on top 1000 rows to Based on entire data set when possible.
4. Inspect Valid, Error, and Empty indicators for each column.
5. Check whether apparent blanks are actual null, empty strings, or whitespace.
Choose a strategy, rather than replacing everything
Field	Suggested approach
Order ID	Investigate first; remove irrecoverable records lacking a usable join key
Order Date	Recover where possible; exclude missing dates from date-trend analysis if not recoverable
CustomerName	Use Unknown only where a placeholder is acceptable
City / State	Recover or label missing geographic information as Unknown, avoiding misleading locations
Amount, Profit, Quantity, Target	Investigate missing or invalid values; avoid replacing with 0 unless zero is genuinely correct


Filter null Order IDs: Open the Order ID filter and uncheck (null) / (blank). To filter both null and blank-text IDs with M:
= Table.SelectRows(
    #"Previous Step",
    each [Order ID] <> null and Text.Trim([Order ID]) <> ""
)
Quality check: Repeat profiling after cleaning and document any records excluded. Review earlier Applied Steps for accidental nulls introduced by transformations or joins.
11. Check and handle duplicate records
1. Select the table in Power Query Editor.
2. To inspect duplicates without destroying your original query, right-click a query → Reference (or Duplicate) and perform checks in the new query.
3. For full-row duplicates, select all columns and choose Home → Keep Rows → Keep Duplicates to inspect repeating records.
4. To remove verified, fully identical duplicate records from the appropriate query, select all columns and choose Home → Remove Rows → Remove Duplicates.
5. For List of Orders, inspect duplicate Order ID values separately because each ID should generally represent one order. Resolve conflicting rows manually or by an agreed data-quality rule before removing anything.
6. For Order Details, do not remove duplicates based only on Order ID: one order can legitimately contain multiple products/line items.
7. For Sales target, check likely business keys such as Category + Month and investigate repetitions before deduplication.
Power Query M examples:
// Deduplicate on all columns (example; use actual current step name)
= Table.Distinct(#"Previous Step")

// Deduplicate List of Orders on Order ID ONLY after verifying the IDs should be unique
= Table.Distinct(#"Previous Step", {"Order ID"})
Caution: Keep Duplicates followed by Remove Duplicates on Order ID is not a safe general cleaning process for Order Details. It can discard valid line items, affecting the average Profit and total Amount.
12. Sort and filter Orders Data
Sort by newest Order Date
1. Select Orders Data.
2. Open the Order Date header dropdown.
3. Choose Sort Descending (newest dates first).
4. Verify that Order Date is a true Date type, not text.
Filter to Tamil Nadu
1. If State was retained during the location merge, open the State dropdown.
2. Deselect Select All, tick Tamil Nadu, and select OK.
3. If you only kept the combined Location, use Text Filters → Ends With → Tamil Nadu as a temporary alternative, or split/recreate the State column for more reliable analysis.
4. Review the filtered rows and reset the filter when finished, if needed.
Note: Filtering the Power Query table changes which records load into the model; use report-page slicers instead if you want to keep all states loaded and interactively compare them later.
13. Group and aggregate Order Details
Goal: Create a separate aggregation query without changing the original transaction table.
1. Right-click Order Details → Duplicate (as requested in the assignment) and rename it, for example Order Details - Summary.
2. In the duplicate query, choose Home → Group By.
3. For one aggregation at a time, use Basic or Advanced grouping:
Question	Group By	Operation	Column	Output name
How many rows per Order ID?	Order ID	Count Rows	Not applicable	Order Line Count
What is average Profit by Category?	Category	Average	Profit	Average Profit
What is total Amount by Sub-Category?	Sub-Category	Sum	Amount	Total Amount


4. Click OK and inspect the resulting summary rows.
5. Create separate duplicates of the original Order Details query if you want all three summaries independently, because these require different grouping keys.
Example M for average Profit by Category:
= Table.Group(
    #"Previous Step",
    {"Category"},
    {{"Average Profit", each List.Average([Profit]), type nullable number}}
)
Important: Work from the original complete Order Details query, not from a version where repeated Order IDs were incorrectly removed. Count Rows per Order ID is the number of detail rows, not necessarily the number of distinct orders (which is one for each grouped Order ID).
14. Aggregate Sales target by month
1. Right-click Sales target → Duplicate and name the result Sales Target by Month.
2. Ensure Target is Fixed Decimal Number.
3. Examine the date/month field (for example Month of Order Date). If it is a Date, optionally create Year and Month or a Year-Month field to avoid combining the same month across different years.
4. Select Home → Group By.
5. Group By: Month of Order Date (or the Year-Month field); New column name: Total Target; Operation: Sum; Column: Target.
6. Click OK and review the summary.
Example M (when the exact field is named Month of Order Date):
= Table.Group(
    #"Previous Step",
    {"Month of Order Date"},
    {{"Total Target", each List.Sum([Target]), type nullable number}}
)
Check: If the source Month field contains only names such as April, grouping mixes data from all years. Group by a Year-Month value if your data spans multiple years.
15. Build relationships in the data model
1. Select Home → Close & Apply to load the queries into Power BI Desktop.
2. Open Model view (diagram icon), or choose Home → Manage relationships → New.
3. Set up the two relationships requested in the assignment:
Relationship A: List of Orders → Order Details
- Table 1: List of Orders; column: Order ID.
- Table 2: Order Details; column: Order ID.
- Expected cardinality: One to many (1:*) if List of Orders has a unique Order ID and Order Details has multiple line items per order.
- Prefer Single cross-filter direction unless analysis calls for another configuration.
- Mark the relationship Active, then confirm.
Relationship B: Order Details → Sales target (Category)
- Table 1: Order Details; column: Category.
- Table 2: Sales target; column: Category.
- In Manage relationships, create the relationship and check Make this relationship active if Power BI permits and this matches the intended model.
Modeling caution: Category usually repeats in both tables (many transactions per category and multiple monthly targets per category). A direct Category relationship may therefore be many-to-many, which can give ambiguous filtering and misleading totals. The assignment asks to create it, but for a production-quality model the preferred approach is generally to create a unique Category dimension and connect both tables to it (one-to-many). For reliable month-wise sales-vs-target comparisons, also create a shared calendar/month dimension and align the date grains. Do not force a relationship to active if Power BI reports ambiguity; resolve the model structure first.

Check: Open Manage relationships and confirm that the Order ID relationship and, if feasible, the requested Category relationship are shown with the intended active status and cardinality.
