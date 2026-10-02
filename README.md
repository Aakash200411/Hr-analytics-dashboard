# HR Attrition Analytics Dashboard

A three-page Power BI report on 1,464 employees that answers three questions: **who leaves, which factors raise the risk, and which current employees to talk to first.**

![Page 1: Attrition at a glance](images/Hr_overview.png)
![Page 2: Why people leave](images/Hr_Drivers.png)
![Page 3: People at risk](images/Hr_People_at_Risk.png)

**Tools:** Power BI · DAX · Power Query

---

## Key findings

| Finding | Number |
|---|---|
| Overall attrition | **16.1%** (235 of 1,464 employees) |
| Overtime vs no overtime | **30.6%** vs **10.3%** attrition (3× the rate) |
| Share of all leavers who worked overtime | **54%** |
| First-year hires who left | **34%** (73 of 213) |
| 18–25-year-olds, with vs without overtime | **64%** vs **23%** attrition |
| Sales Representatives | **39.5%** attrition, about 2.5× the company rate |
| Highest-attrition departments | Sales 25%, Human Resources 23%, Administration 21% (Operations lowest at 8%) |
| Attrition across salary bands | **15–19%** in every band, so pay band does not separate leavers from stayers |
| Stock option level 0 vs level 1 | **24%** vs **9%** attrition |
| Low job satisfaction (level 1) | **28%** of leavers vs **18%** of stayers |
| Single vs married employees | **21%** vs **11%** attrition |
| Frequent travellers vs no travel | **25%** vs **8%** attrition |

### The highest-risk profile

Risk builds as factors stack up:

| Group | People | Attrition |
|---|---|---|
| All employees | 1,464 | 16% |
| Works overtime | 415 | 31% |
| + Job level 1 | 155 | 53% |
| + No stock options | 74 | **68%** |

The 74 people in the last row are **21% of all leavers**. The wider overtime + job level 1 group (155 people) accounts for **35%** of leavers. The two figures describe different groups, so the dashboard labels them separately.

## Recommendations

1. **Review overtime workloads**, starting with junior staff and the Sales Representative role.
2. **Support first-year hires** with structured onboarding and a check-in before the six-month mark.
3. **Look at stock-option eligibility for new joiners.** Level 0 staff leave at nearly 3× the rate of level 1.
4. **Don't rely on pay rises as the main lever.** Attrition is flat across salary bands.
5. **Start with the 73 active employees** in the overtime + job level 1 group. 24 of them have no stock options (the "Highest" tier) and 49 do (the "High" tier).

## What's in the report

**Page 1: Attrition at a glance.** A hero KPI with donut, four KPI tiles, a treemap of departments (size = headcount, colour = attrition rate), a bubble chart of job roles, an area chart of attrition by years at company, and small multiples of age group split by overtime.

**Page 2: Why people leave.** A decomposition tree that finds the highest-rate path, a stacked bar comparing the satisfaction of leavers and stayers, a stock-options matrix with data bars, a line chart of attrition by salary slab, and a ranked bar of other segments.

**Page 3: People at risk.** Summary cards, a table of active at-risk employees with risk-tier tags, active at-risk employees by department, and a donut of the two risk tiers.

All three pages share a left navigation rail with Department, Age group and Gender slicers and a reset button.

## How it was built

**Power Query**
- Removed duplicate employee IDs (1,480 → 1,464 rows)
- Standardised inconsistent `BusinessTravel` labels
- Dropped `EmployeeCount`, `Over18` and `StandardWorkingHours` (identical in every row)
- Kept the 61 rows with a blank `YearsWithCurrManager` instead of deleting them
- Added helper columns: `Status` (Left / Stayed), `Tenure Band`, order columns so bands don't sort alphabetically, and `Overtime Label`

**Key DAX measures**

```dax
Headcount = COUNTROWS(HR)

Left = CALCULATE([Headcount], HR[Attrition] = "Yes")

Attrition Rate = DIVIDE([Left], [Headcount])

Overall Attrition Rate = CALCULATE([Attrition Rate], ALL(HR))

Overtime Attrition = CALCULATE([Attrition Rate], HR[OverTime] = "Yes")

First-Year Attrition =
DIVIDE(
    CALCULATE([Left], HR[YearsatCompany] <= 1),
    CALCULATE([Headcount], HR[YearsatCompany] <= 1)
)

Satisfaction Share =
DIVIDE([Headcount], CALCULATE([Headcount], REMOVEFILTERS(HR[JobSatisfaction])))

Profile Rate =
CALCULATE(
    [Attrition Rate],
    HR[OverTime] = "Yes", HR[JobLevel] = 1, HR[StockOptionLevel] = 0
)

Risk Tier (calculated column) =
IF(
    HR[Attrition] = "No" && HR[OverTime] = "Yes" && HR[JobLevel] = 1,
    IF(HR[StockOptionLevel] = 0, "Highest", "High")
)
```

**Design decisions**
- Compare **rates, not counts**, so a large team doesn't look riskier just because it is large
- A dashed overall-average line on every rate chart; one accent colour (gold) reserved for the problem
- Chart titles state the finding ("Overtime hits the young hardest") instead of describing the chart
- Different visuals for different jobs: treemap for size and rate together, bubble chart for volume vs rate, decomposition tree for the highest-rate path

## Limitations

- **One snapshot, no dates.** It can't show trends over time or why someone left.
- **Association, not causation.** Nothing here proves that changing overtime or stock options would change behaviour.
- **Small groups.** Stock option level 3 has 85 people, and some job roles have fewer than 100 employees.
- **The risk tier is a rule**, not a prediction model.
- **Page 3 lists named employees.** In a real deployment it would sit behind row-level security.

---

Built by [Aakash Lodha](https://github.com/Aakash200411) · [Portfolio](https://aakash200411.github.io/Portfolio/)
