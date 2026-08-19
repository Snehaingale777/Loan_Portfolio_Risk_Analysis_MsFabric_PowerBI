# Loan_Portfolio_Risk_Analysis_MsFabric_PowerBI

Tools: Microsoft Fabric (Dataflow Gen2, Notebooks/PySpark, Lakehouse, Star Schema, Semantic Model), Power BI
Dataset: Real-world loan portfolio data (38,577 loans originally, 36,976 after cleaning, 24 columns, 50 U.S. states, 2021)
Live report: https://www.kaggle.com/datasets/datawitharyan/financial-loan-dataset

Why I built this

After building an HR attrition dashboard (which implemented column-level security) and a sales performance dashboard (which implemented row-level security), I wanted a third project that moved into a new domain — Finance/BFSI — worked at real scale instead of a small sample dataset, and combined both security approaches in one place rather than just one or the other. This is the first of my three projects with both row-level and column-level security implemented together.

How I built it

I loaded the raw loan data into a Lakehouse using Dataflow Gen2, adding DTI Band and Income Band as conditional columns during ingestion, and later went back and added a filter step to remove rows with a zero DTI or a missing employment title, since those looked like bad data rather than genuine zero values.

From there I built a star schema in a PySpark notebook: one fact table for the loans themselves, and separate dimension tables for state, purpose, grade, and date. Each dimension gets a surrogate key through monotonically_increasing_id(), and the date dimension is a full generated calendar rather than just the distinct dates that happen to appear in the data, so there are no gaps. After building the joins, I added a validation cell that checks the fact table's row count against the source and confirms there are zero missing foreign keys across every join — I wanted proof the joins actually worked, not just an assumption that they did.

The semantic model connects to these Lakehouse tables directly through Direct Lake, with relationships from each dimension into the fact table and five core DAX measures: Total Loans, Total Loan Amount, Default Rate, Avg DTI, and Avg Interest Rate.

For security, I set up two separate things. First, row-level security restricting a role to a single state (address_state = "CA") — I built and saved this on the live semantic model, but couldn't test it with a second logged-in account since my Fabric trial doesn't let me provision additional Entra ID users. Second, column-level security through Fabric's native OneLake Security, restricting a role from seeing the annual_income column entirely — this one I could confirm was applied, since Fabric flags the table with a visible "CLS constraints applied" indicator once the rule is saved.

What's in the dashboard

The Overview page has headline KPIs (total loans, total loan amount, average DTI, default rate, average interest rate), the top 5 riskiest states and loan purposes by default rate, a ranked table of the top 10 riskiest sub-grades, and a breakdown of average DTI by grade.

The Deep Dive page covers default rate by grade, by income band, and by DTI band, a monthly trend of loan volume against default rate, and a grade-by-income-band matrix with a heatmap. That matrix is the one visual that surfaces something the single-dimension charts can't: Grade G borrowers earning under $40k default at 48.15%, well above either Grade G alone (31.00%) or under-$40k income alone (16.90%) — the risk compounds when both factors are present.

Screenshots
https://github.com/Snehaingale777/Loan_Portfolio_Risk_Analysis_MsFabric_PowerBI/blob/main/df_loans.jpg
https://github.com/Snehaingale777/Loan_Portfolio_Risk_Analysis_MsFabric_PowerBI/blob/main/notebook_star_schema_cell6.jpg

[add your screenshots here: Dataflow Gen2 filter step, notebook star schema build + validation output, semantic model relationships, RLS role setup, CLS applied indicator, dashboard Overview and Deep Dive pages]

What I'd improve

I'd like to properly test the row-level security setup against a real second user account once I have access to one, the same limitation I ran into on the RLS side of this project. The role and filter logic are already built and saved on the live model — it just needs an actual second login to confirm the restriction behaves as expected in practice, the same way the column-level security already has.
