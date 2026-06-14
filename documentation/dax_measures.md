# 📐 DAX Measures Documentation — AI & Data Science Job Market Dashboard

This document lists and explains all DAX measures created in the `_Mesures` table of the Power BI model.

---

## Basic Measures

### Average Salary (Average Salary)
```dax
Salaire Moyen = AVERAGE(Fact_Salaires[salary_in_usd])
```
Returns the average salary in USD across the filtered context.

---

### Job Offers (Number of Job Offers)
```dax
Nb Offres = COUNTROWS(Fact_Salaires)
```
Counts the total number of job postings in the current filter context.

---

### Country Count (Number of Countries)
```dax
Nb Pays = DISTINCTCOUNT(Dim_Pays[company_location])
```
Returns the number of distinct countries represented in the dataset.

---

### Max Salary (Maximum Salary)
```dax
Salaire Max = MAX(Fact_Salaires[salary_in_usd])
```
Returns the highest salary value in the filtered context.

---

### Min Salary (Minimum Salary)
```dax
Salaire Min = MIN(Fact_Salaires[salary_in_usd])
```
Returns the lowest salary value in the filtered context.

---

## Advanced Measures

### Salary Growth
```dax
Croissance Salaire = 
VAR SalaireAnneeActuelle = AVERAGE(Fact_Salaires[salary_in_usd])
VAR SalaireAnneePrec = CALCULATE(
    AVERAGE(Fact_Salaires[salary_in_usd]),
    FILTER(
        Dim_Temps,
        Dim_Temps[work_year] = MAX(Dim_Temps[work_year]) - 1
    )
)
RETURN
DIVIDE(SalaireAnneeActuelle - SalaireAnneePrec, SalaireAnneePrec, 0)
```

**Purpose:** Calculates the year-over-year salary growth rate.

**Techniques used:**
- `VAR` / `RETURN` for readable, reusable calculations
- `CALCULATE` to modify filter context
- `FILTER` to isolate the previous year
- `DIVIDE` for safe division (avoids divide-by-zero errors)

---

### Rank JobTitle (Job Title Ranking)
```dax
Rank Metier = 
RANKX(
    ALL(Dim_Metier[job_title]),
    AVERAGE(Fact_Salaires[salary_in_usd]),
    ,
    DESC
)
```

**Purpose:** Ranks job titles by their average salary, from highest (rank 1) to lowest.

**Techniques used:**
- `RANKX` for dynamic ranking
- `ALL()` to ignore existing filters and rank across all job titles
- `DESC` order (highest salary = rank 1)

---

### % Full Remote (Full Remote Job Share)
```dax
% Full Remote = 
DIVIDE(
    CALCULATE(COUNTROWS(Fact_Salaires), Fact_Salaires[remote_ratio] = "Full Remote"),
    COUNTROWS(Fact_Salaires),
    0
)
```

**Purpose:** Calculates the proportion of job postings that are fully remote.

**Techniques used:**
- `CALCULATE` with a boolean filter condition
- `DIVIDE` for safe percentage calculation

---

## Data Model Notes

All measures are organized in a dedicated `_Mesures` table (best practice: measures separated from data tables) created using:

```dax
_Mesures = {1}
```

This keeps the data model clean and makes measures easy to locate and maintain.

---

## Star Schema Context

These measures operate on the **Fact_Salaires** table, which is connected via 1:* relationships to:
- `Dim_Time` (work_year)
- `Dim_JobTitle` (job_title)
- `Dim_Country` (company_location)
- `Dim_Experience` (experience_level)
- `Dim_Contract` (employment_type)
