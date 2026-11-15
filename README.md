# Cybersecurity Events Calendar 2026–2027

A calendar of **451 public cybersecurity events** in **48 countries** across ASEAN, the US, Europe and the UK:
conferences, vendor roadshows, government awareness programmes, community chapters and adjacent tech shows. It's
built twice:

- **Excel workbook:** a calendar heatmap, a one-formula agenda and a dashboard, verified against Python with
  0 mismatches.
- **Power BI project:** a Power Query date table and three report pages (Calendar, Data Table, Insights). It
  loads the data straight from this repository, so anyone can open it and refresh.

Most source dates were free text such as "Q3 2026 (TBC)". Every date is parsed into a **precision level** before
it goes on a calendar.

![Excel calendar heatmap](images/excel-calendar.png)

**Built with:**
- **Excel (Microsoft 365):** COUNTIFS, EOMONTH, LET, FILTER, SORTBY, CHOOSECOLS, UNIQUE, data validation, colour
  scales.
- **Power BI:** Power Query (M) over a web CSV source, a date table, DAX measures, PBIP.
- **Python:** data preparation and verification.

---

## The data problem: dates you can't put on a calendar

Event dates were collected from official sites and listings and written down as found: `21-24 Apr 2026`,
`Mar 2026`, `2026 (dates TBC)`, or nothing yet. Every start date is parsed into one of four precision levels:

| Precision | Meaning | Events |
|---|---|---|
| **day** | exact start date | 319 |
| **month** | month known, day TBC | 7 |
| **year** | only the year known | 71 |
| **none** | no date published yet | 54 |

Only exact days go on the calendar grid. Month-only events are counted per month, and undated events are listed
separately rather than placed on a guessed date. An event is flagged **TBC / Undated** (128 events) when its date
is year-only or missing, says "TBC", or has a verification note marking it as unconfirmed.

Each event also keeps:
- its market, category, organiser, audience scale and source URL;
- a **verification status**: confirmed on the official source (283), date not yet confirmed (55), recurring
  programme (54), or listed only by a third-party aggregator (36).

**Scope:** two research workbooks: ASEAN (146 events) and Global (305: US 128, Europe 113, UK 64), covering
2026–2027. Two events have 2025 start dates: a 2025/26 programme and a discontinued conference kept for the
record.

## Excel version

[`data/Cyber_Events_Calendar.xlsx`](data/Cyber_Events_Calendar.xlsx) has three views over one Excel Table
(`tblEvents`):

- **Calendar:** a 12-month × 31-day heatmap of events starting each day, with **Year** and **Market** drop-downs.
  Each cell is a `COUNTIFS`, and impossible dates (30 Feb) are blanked with `EOMONTH`.
- **Agenda:** choose a year, month and market, and **one formula** lists that month's events, sorted by date:

```excel
=LET(y, $B$4,  m, MATCH($B$5, {"January", ..., "December"}, 0),  mk, $B$6,  d, tblEvents[Start Date],
     keep, (YEAR(d) = y) * (MONTH(d) = m) * (((tblEvents[Market] = mk) + (mk = "All")) > 0) * (d <> ""),
     raw,  CHOOSECOLS(tblEvents, XMATCH({"Start Date","End Date","Event Name","Country","City",
                                         "Category","Date Precision","Source URL"}, tblEvents[#Headers])),
     cols, IF(raw = "", "", raw),
     IF(SUM(keep) = 0, "No events for this selection", SORTBY(FILTER(cols, keep), FILTER(d, keep), 1)))
```

- **Dashboard:** headline numbers, events per month by market, events by category, and date precision.

![Excel agenda](images/excel-agenda.png)
![Excel dashboard](images/excel-dashboard.png)

**Verified:** the workbook was recalculated in Excel and checked against Python for the default view (2026, all
markets):
- all 372 calendar cells and the monthly totals;
- the agenda's row count and sort order;
- every dashboard number.

All of them match, with no formula errors.

## Power BI version

[`powerbi/CybersecurityEvents.pbip`](powerbi/) is a Power BI Project: the model, the Power Query steps and the
report are saved as text files.

- **Model:** `Events` linked to a **`Date` table** on start date. The date table is built in Power Query
  (2025–2027) and marked as the model's date table, with month columns sorted by number.
- **Power Query:** reads [`data/events.csv`](data/events.csv) from this repository through a `DataFolder`
  parameter. It turns blanks into nulls before typing, types every column, and adds a readable precision label.
- **Measures:** Events, Events With Exact Date, Events Date TBC, Exact Date %, Countries, Events Starting.
- **Pages:**
  - **Calendar:** a month × day matrix of events starting each day, with a colour scale and market and category
    slicers.
  - **Data Table:** every event with dates, location, category, type, verification status and source link.
  - **Insights:** headline cards, events per month by market, events by category, and how exact the dates are.

```m
Source   = Csv.Document(Web.Contents(DataFolder & "events.csv"), [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]),
Promoted = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
Blanks   = Table.ReplaceValue(Promoted, "", null, Replacer.ReplaceValue, {"Start Date", "End Date"}),
Typed    = Table.TransformColumnTypes(Blanks, {..., {"Start Date", type date}, {"End Date", type date}, ...}, "en-US")
```

**To open it:** clone or download the repository and open `powerbi/CybersecurityEvents.pbip` in Power BI Desktop.
Choose **Anonymous** when asked how to connect to `raw.githubusercontent.com`. To use a local copy of the data
instead, set the `DataFolder` parameter to that folder and change `Web.Contents` to `File.Contents`.

*Report screenshots will be added once the report is published.*

## Data

[`data/events.csv`](data/events.csv) holds 451 rows × 21 columns of public event information. There is no
personal contact data.

| Column group | Columns |
|---|---|
| Event | Event, Dataset, Market, Category, Event Type, Focus, Organiser, Scale / Audience, MSSP Relevance |
| Place | Country, City, Venue |
| Dates as researched | Start (as written), End (as written), Year |
| Parsed dates | Start Date, End Date, Date Precision, Date Status |
| Evidence | Source URL, Verification |

Dates change: check the source link before relying on any single event.

## Repository layout

```
data/events.csv                     the event list (public information)
data/Cyber_Events_Calendar.xlsx     Excel version (Microsoft 365 / Excel 2024)
powerbi/                            Power BI Project (PBIP: TMDL model + report definition)
images/                             screenshots of the Excel version
```
