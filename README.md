# Dare Careers – Student Progress & Performance Dashboard (Power BI)

Power BI dashboard that tracks learner engagement, progress and outcomes for **Dare Careers**, a training provider running **Power BI** and **AWS Cloud** tracks.

The dataset is **one cohort (Cohort 1)** with **two learning tracks**. The dashboard uses a Cohort slicer, so more cohorts can be added in the future.

## Files

| File / folder | Description |
|---|---|
| `dare_careers_v2.pbix` | **Final dashboard** (submit this one) |
| `dare_careers.pbix` | Earlier version |
| `data/` | Source data: Zoom attendance, participation, labs & quizzes, learner status |

## Report pages

### 1. Overall Performance
- **Cards:** Total Learners, Total Graduates, Total Certified, Total Dropouts, Graduation Rate
- **Bar charts by Track:** Graduation Rate, Certification Rate, Dropout Rate, Attendance Rate, Participation Rate, Average Score
- **Line chart:** Attendance Rate over time
- **Slicers:** Cohort, Track, Certification Type, Certification Status (Learner Status)

### 2. Learner Insights
- **Learner table:** email, track, attendance rate, participation rate, average score
- **Cards:** Count of Labs, Avg Labs Done, Total Hours, Attendance Rate, Participation Rate, Average Score
- **Charts:** Average Score by Type (Quiz vs Lab), by Week, by Track
- **Slicers:** Cohort, Track, Month, Week, Certification Status, Program Status

### 3. Track Outcomes (bonus)
- Summary table and Outcomes by Track chart (graduation, certification and dropout rates side by side)

## Business rules

- **Attendance:** a learner counts as *attended* only if they spent **more than 30 minutes** in a Zoom session (30 min → 0, 31 min → 1).
- **Dropout:** a learner is a dropout only if **all three** are true:
  1. Non-Graduate
  2. Not Certified
  3. Incomplete attendance (missed at least 1 session)
- **Certification Type:** AWS or Power BI, based on the certified learner's track.

## Data model

Star schema with **Learners** as the main dimension (joined on email).

| From (many) | To (one) | Column |
|---|---|---|
| Assessments | Learners | email |
| Participation | Learners | email |
| Zoom_Attendance | Learners | email |
| Participation | Calendar | Date |
| Zoom_Attendance | Calendar | Date |
| Calendar | Weeks | Week (cross filter: Both) |
| Assessments | Weeks | Week Number |

**Weeks bridge table:** Assessments are weekly and have no date, so a small `Weeks` table (weeks 1–10) connects Calendar and Assessments. This way the **Month** and **Week** slicers filter attendance, participation **and** scores.

## Key DAX

```DAX
Attendance Rate = DIVIDE(SUM(Zoom_Attendance[attended]), COUNTROWS(Zoom_Attendance)) + 0

Graduation Rate = DIVIDE([Total Graduates], [Total Learners]) + 0

Certification Rate = DIVIDE([Total Certified], [Total Learners]) + 0

Dropout Rate = DIVIDE([Total Dropouts], [Total Learners]) + 0

Participation Rate =
DIVIDE(
    COUNTROWS(Participation),
    [Total Learners] *
        CALCULATE(
            DISTINCTCOUNT(Participation[Date]),
            ALLEXCEPT(Learners, Learners[Track])
        )
) + 0

Total Dropouts =
VAR MinMissedSessions = 1
RETURN
CALCULATE(
    [Total Learners],
    Learners[Graduation Status] = "Non Graduate",
    Learners[Certification Status] = "Not Certified",
    FILTER(
        Learners,
        CALCULATE(COUNTROWS(Zoom_Attendance), Zoom_Attendance[attended] = 0) >= MinMissedSessions
    )
)

Certification Type =
IF(
    Learners[Certification Status] = "Certified",
    IF(Learners[Track] = "AWS Cloud", "AWS", "Power BI"),
    "Not Certified"
)

Weeks = DISTINCT(Calendar[Week])
```

`+ 0` shows **0** instead of a blank when there is no data. `ALLEXCEPT` in Participation Rate compares each learner with **all session dates of their track**, not only the days they participated.

## Key insights

| Track | Learners | Graduation | Certification | Dropout | Attendance | Participation |
|---|---|---|---|---|---|---|
| AWS Cloud | 40 | 82.50% | 60.00% | 17.50% | 79.25% | 89.20% |
| Power BI | 100 | 86.00% | 68.00% | 0.00% | 100.00% | 85.06% |
| **Total** | **140** | **85.00%** | **65.71%** | **5.00%** | **94.07%** | **86.24%** |

- **All 7 dropouts come from AWS Cloud**, which is also the track with lower attendance.
- Power BI has **100% attendance and 0 dropouts**.
- Both tracks graduate learners at a similar rate (~83–86%).

## How to open

1. Install **Power BI Desktop**.
2. Open `dare_careers_v2.pbix`.

**Author:** Christine Umutoni – AmaliTech Training, Data Engineering