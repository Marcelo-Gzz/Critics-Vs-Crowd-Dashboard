# Critics-Vs-Crowd-Dashboard

**A Power BI dashboard analyzing where expert and audience opinion diverge on acclaimed films.**

When a film is universally praised, "is it good?" stops being an interesting question because the answer is always yes. So this project asks a sharper one: among films everyone agrees are great, *where do critics and audiences actually disagree, and what predicts it?*

![Dashboard preview](dashboard.png)
<!-- Replace dashboard.png with your exported screenshot -->

---

## Key findings

- **Critics and audiences mostly agree.** Across 401 acclaimed films, the average gap between critic and audience scores is roughly zero (-0.1 points). On a curated set of well-regarded movies, broad consensus is the norm.
- **But about 39% of films are genuinely contested**, differing by more than 5 points in one direction or the other. That contested third is where the story lives.
- **Audiences over-index on genre films.** Fantasy, adventure, and crime films rate higher with audiences than with critics.
- **Critics reward science fiction, thrillers, and romance** more than audiences do, with science fiction showing the largest critic-favored gap (about +5 points on average).
- **Disagreement clusters by era.** The pre-1980 canon leans critic-favored, while films from the 1980s through 2000s lean audience-favored, with the 1990s showing the strongest "audiences loved it more" tilt.
- **The single most divisive film is Uncut Gems**, which critics scored 29 points above audiences.

---

## The dashboard

A single interactive page built around the thesis. Every visual answers "where does acclaim get contested?"

| Component | What it shows |
|-----------|---------------|
| **KPI cards** | Films compared, average gap, percent contested, and the single most divisive film |
| **Scatter plot** | Critic score vs. audience score for each film, colored by the gap. Distance from the diagonal is the size of the disagreement |
| **Diverging bar chart** | Average critic/audience gap by genre, so you can see which genres split the two camps |
| **Most Divisive Films table** | The top films ranked by the absolute size of the gap, naming names |
| **Slicers** | Filter the entire page by decade or genre to ask, for example, "where do critics and audiences clash in horror?" |

A consistent diverging color scale runs through every visual: one color for critic-favored films, the other for audience-favored, with neutral tones for consensus.

---

## Data

- **Scope:** 449 acclaimed films spanning the 1920s through the 2020s, across 18 genres and 27 languages.
- **Per-film attributes:** title, year, decade, genre, studio, language, and scores from multiple rating sources (Rotten Tomatoes critic and audience, Metacritic, Letterboxd, and a blended all-sources average), plus awards data.
- **Core derived metric:** `Critic_Audience_Gap`, defined as the Rotten Tomatoes critic score minus the audience score. Positive means critics favored the film; negative means audiences did.

> **Note on source:** the rating data was assembled from [add your source / API here, e.g. the OMDb API and public rating sites]. Add the raw record count and collection date once finalized.

### Data pipeline

The dashboard consumes `movies_clean.csv`, the output of an upstream extract-transform-load process:

1. **Extract** raw film and rating records from the source(s) listed above.
2. **Transform and reconcile** multiple rating scales into one normalized row per film, keyed on `imdbID`.
3. **Engineer features**, including the blended all-sources average, the critic/audience gap, and decade buckets.
4. **Load** the cleaned, validated result to `movies_clean.csv` for analysis.

Inside Power BI, the import was validated for UTF-8 encoding (the dataset includes non-ASCII names), correct data types, and preserved nulls (missing scores are kept as null rather than coerced to zero, so they are excluded from averages rather than dragging them down).

---

## Tools and techniques

**Built with:** Power BI Desktop (Power Query, the data model, DAX, and report visuals).

Key DAX measures and a calculated column drive the analysis:

```dax
-- Real denominator: only films that actually have a gap to compare
Films Compared =
CALCULATE(
    COUNTROWS(movies_clean),
    NOT(ISBLANK(movies_clean[Critic_Audience_Gap]))
)

-- Share of films where the two camps differ by more than 5 points either way
% Contested =
DIVIDE(
    CALCULATE(
        COUNTROWS(movies_clean),
        FILTER(movies_clean, ABS(movies_clean[Critic_Audience_Gap]) > 5)
    ),
    CALCULATE(
        COUNTROWS(movies_clean),
        NOT(ISBLANK(movies_clean[Critic_Audience_Gap]))
    )
)

-- Names the most contested film in the current filter context
Most Divisive Film =
VAR MaxAbs =
    MAXX(ALLSELECTED(movies_clean), ABS(movies_clean[Critic_Audience_Gap]))
RETURN
    CALCULATE(
        MAX(movies_clean[Title]),
        FILTER(movies_clean, ABS(movies_clean[Critic_Audience_Gap]) = MaxAbs)
    )

-- Calculated column used to rank the table by disagreement, regardless of direction
Gap Magnitude = ABS(movies_clean[Critic_Audience_Gap])
```

Other techniques used: conditional formatting with a diverging gradient to encode the gap as color, cross-filtering slicers so a selection recomputes every visual on the page, and a top-N filter to surface only the most divisive films in the table.

---

## Limitations

- **The collection skews Anglophone.** About 74% of the films are English-language, so the disagreement patterns reflect a largely Western critical lens rather than global consensus.
- **It is a curated set of acclaimed films**, not a representative sample of all movies. Findings describe how opinion divides *within* a canon of well-regarded films, not movies in general.
- **Some genres have small sample sizes.** Genres with only a handful of films were filtered out of the genre breakdown to avoid drawing conclusions from noise.
- **48 films lack an audience score** and are excluded from the gap analysis, which is why the working sample is 401 rather than 449.

---

## Possible extensions

- Rebuild the single flat table as a **star schema**, breaking out genre, studio, and a split-out streaming-platform dimension with proper relationships.
- Compare disagreement across **more rating sources** (for example, Letterboxd's cinephile audience vs. the mainstream Rotten Tomatoes audience).
- Test whether **awards** track with critic or audience preference.

---

## How to open

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows).
2. Clone this repository.
3. Open `dashboard2.pbix`. The dataset.
