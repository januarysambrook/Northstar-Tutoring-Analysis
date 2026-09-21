# Northstar Tutoring Analytics

A Power BI model and measure library built to explain how 520 booked tutoring sessions
turn into revenue, profit and satisfied students.

> £22.1k of revenue against £15.6k of tutor cost, a **29.8% gross margin**, with
> **87.7%** of booked sessions completed. The pressure on this business is not delivery
> quality. It is concentration: Maths is 83% of revenue and online is 70% of it.

## The question

Northstar books sessions. It does not know which of them make money. Session records
carry a price and a duration but no cost, so profitability has to be derived rather than
read.

- Where does revenue actually come from, by subject, tutor and delivery mode?
- What does a session cost to deliver?
- How many booked sessions are lost, and what is that worth?
- Does the highest-earning tutor also produce the most satisfied students?

## Findings

**Maths carries the business.** GCSE Maths £7.0k, A Level Maths £4.9k, KS3 Maths £3.7k
and other Maths £2.7k, against £3.8k for Science and English combined. Concentration is
an advantage while Maths demand holds and a single point of failure if it does not.

**Online is the growth route.** £15.4k of revenue online against £6.7k face to face.
Online capacity is not limited by travel time or venue, so growth does not carry a
fixed-cost step.

**64 of 520 booked sessions never happened.** 34 were no-shows or late cancellations and
30 were cancelled early and earned nothing. At £46.64 per completed session, those 30
represent roughly £1.4k of capacity the diary had already reserved. That is recoverable
without a single new student.

**The top earner is not the top-rated tutor.** Tutor D brings in £6.2k, around 28% of all
revenue, but Tutors F and G have the strongest 5-star rates on completed sessions.
Allocating on revenue alone concentrates risk and ignores what students actually rate.

## Modelling decisions that change the answer

- **Blank satisfaction scores stay blank.** A zero would read as a real one-star rating
  and drag every average down.
- **The 30 early cancellations stay in the completion denominator.** They earned nothing
  but they still occupied booked capacity. Removing them lifts completion from 87.7% to
  93.1% and hides the very loss the business needs to see.
- **80 registered students are kept distinct from the 76 who actually booked,** so
  per-student figures mean what they say.
- **Key columns are hidden** so report users build from measures rather than raw fields.

## The model

A star schema: `FactSessions` with 520 rows, filtered by `DimStudents` (80),
`DimTutors` (8), `DimSubjects` (10) and a date table built rather than imported.

## Measures

`Total Revenue`, `Sessions Booked`, `Completed Sessions`, `Completion Rate`,
`Online Revenue`, `F2F Revenue`, `Tutor Cost`, `Gross Profit`, `Profit Margin %`,
`Revenue per Completed Session`, `Tutor Revenue Rank`, `High Satisfaction %`.

The cost column did not exist, so it is calculated row by row:

```dax
Tutor Cost =
SUMX (
    FactSessions,
    FactSessions[DurationHours]
      * RELATED ( DimTutors[PayRate] )
      * [Status Multiplier]
)
```

`SUM` cannot do this. Pay rate lives on the tutor, duration lives on the session, and a
cancelled session costs a different fraction of a full one, so there is no single column
to add up. `SUMX` evaluates the expression per row and totals it, with `RELATED` pulling
the rate across the relationship.

Patterns used across the library: `SUMX`, `RELATED`, `SWITCH`, `CALCULATE`, `DIVIDE`,
`REMOVEFILTERS`, `TOTALYTD`, `SAMEPERIODLASTYEAR`, `RANKX` and `SELECTEDVALUE`.

## Recommendations

1. **Refill cancelled slots.** Roughly £1.4k of reserved capacity earned nothing, and a
   follow-up process costs nothing to trial.
2. **Protect the Maths engine.** Track Maths completion and waiting-list demand as a
   leading indicator rather than a year-end number.
3. **Allocate tutors on two axes,** balancing revenue rank against satisfaction rank.
4. **Grow one non-Maths subject deliberately,** to the point where it could absorb a dip.
5. **Keep online as the default,** watching satisfaction by mode as volume rises.

## Limitations

- **Synthetic and small.** 520 sessions across 8 tutors, so slices by tutor or subject
  sit on small counts and will move.
- **No student outcomes.** No grades, retention or repeat bookings, so value is measured
  in revenue rather than in learning.
- **Ratings only where sessions completed.** A no-show is the worst experience a student
  can have and carries no score at all.
- **Best next addition:** repeat-booking data, to test whether satisfaction converts into
  future revenue.

## Repository contents

- `Northstar_Tutoring.pbix` the Power BI model, measures and report

The source dataset is a synthetic file provided as course material and is not
redistributed here. The schema is described above and the model is embedded in the pbix.

---

Built by January Sambrook
