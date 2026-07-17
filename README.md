# D3 Bar Race

A self-contained **bar chart race** viewer: animated horizontal bars that grow,
shrink, and reorder over time, with a running caption in the corner. Originally
built from a [D3 / Observable](https://d3js.org/) notebook and flattened into a
plain static page so it can run locally with no build step — handy for screen
recording a race from your own data.

## Requirements

- [Node.js](https://nodejs.org/) (for the dev server; any static file server works)

## Running locally

From the repository root, serve the folder over HTTP and open it in a browser:

```sh
npx http-server -p 8080 -c-1
```

Then go to <http://localhost:8080>. Click **Replay** to restart the animation
(useful when recording).

> The page must be served over **HTTP** — opening `index.html` directly via
> `file://` won't work, because the browser blocks the CSV `fetch` and ES module
> loading. The `-c-1` flag disables caching so edits to the CSVs show up on a
> plain refresh.

## Using your own data

The chart reads two CSV files from the [`files/`](files/) folder:

| File | Purpose |
| --- | --- |
| `files/data.csv` | The race data — what the bars represent. |
| `files/labels.csv` | The caption shown in the corner for each step. |

`data.csv` and `labels.csv` are **git-ignored** so your real datasets stay out of
the repo. The repo instead ships example files you copy from:

```sh
cp files/data_placeholder.csv   files/data.csv
cp files/labels_placeholder.csv files/labels.csv
```

Then edit `files/data.csv` and `files/labels.csv` (or overwrite them with your own
files) — keep the **same column names and structure** shown below.

### `data.csv` format

One row per entity per time step (long/tidy format):

```csv
timestamp,name,value
1,Alice,10
1,Bob,8
2,Alice,15
2,Bob,12
```

- `timestamp` — integer time step (`1, 2, 3, …`). Each distinct value is one frame
  of the race; the chart interpolates smoothly between them.
- `name` — the label of the bar (the entity that races).
- `value` — the numeric bar length at that step.
- *(optional)* add a `category` column to color bars by group instead of per-name.

### `labels.csv` format

The caption ("ticker") shown in the bottom-right corner:

```csv
timestamp,label
1,January 2026
2,February 2026
```

- `timestamp` — matches the `timestamp` values in `data.csv`.
- `label` — the text to display (a date, a heading, anything).

> **Heads-up on alignment:** the caption shown while the race is at data timestamp
> `N` is currently taken from the `labels.csv` row with timestamp `N + 1` (a
> leftover from the source notebook, where the first row was a pre-race caption).
> Account for this offset in your label rows, or ask to have it mapped 1:1.

## Tuning the animation

A few knobs live as one-line functions near the top of [`index.js`](index.js):

| Setting | Line | Default | Meaning |
| --- | --- | --- | --- |
| `duration` | [index.js:54](index.js#L54) | `250` | milliseconds per frame (lower = faster) |
| `n` | [index.js:58](index.js#L58) | `24` | number of bars shown at once |
| `k` | [index.js:81](index.js#L81) | `10` | interpolation frames between steps (higher = smoother) |
| `barSize` | [index.js:260](index.js#L260) | `48` | bar height in pixels |

## Project layout

```
index.html      entry point — loads the runtime and renders the chart
index.js        the chart logic (the part you edit)
runtime.js      vendored Observable runtime (don't edit)
inspector.css   styling
files/          CSV data + *_placeholder.csv examples
```
