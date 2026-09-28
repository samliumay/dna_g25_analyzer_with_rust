# 🧬 g25 — a terminal analyzer for Global25 coordinates

**A fast, keyboard-driven terminal app for working with G25 (Global25) coordinates.**
Find your closest populations, model your ancestry as a mix of sources, blend populations together, and look at the results as plots, heatmaps and maps without leaving the terminal.

The goal is to be as capable and easy to use as the popular web G25 tools (Vahaduo, G25 Studio, MyGeneticMaps, …), while running offline on your own machine, answering instantly, and keeping your coordinates on your own computer.

> **Status:** early development. The feature list below is the design target. See the [Roadmap](#roadmap) for what's done.

---

## Table of contents

- [What is G25?](#what-is-g25)
- [Features](#features)
- [Installation](#installation)
- [Quick start](#quick-start)
- [The interactive TUI](#the-interactive-tui)
- [Command-line mode](#command-line-mode)
- [Data files](#data-files)
- [File formats](#file-formats)
- [How the math works](#how-the-math-works)
- [Project layout](#project-layout)
- [Roadmap](#roadmap)
- [Credits & data attribution](#credits--data-attribution)
- [Disclaimer](#disclaimer)

---

## What is G25?

**Global25 (G25)** is a 25-dimensional PCA (principal component analysis) of autosomal DNA created by Davidski of the [Eurogenes](https://eurogenes.blogspot.com/) project. Each individual or population is described by **25 numbers**. Two samples with close coordinates have similar genetic makeup.

With those 25 numbers you can:

| Question | Technique |
|---|---|
| *Who am I closest to?* | Euclidean distance to every reference population |
| *What mix of populations best explains me?* | Admixture modelling (nMonte / Vahaduo-style least-squares fitting) |
| *What would a 50/50 Greek + Norwegian look like?* | Weighted averaging ("mixing") of coordinates |
| *Where do I sit on the genetic map?* | Plotting PC1 vs PC2 (or any pair of PCs) |
| *Which populations cluster together?* | Hierarchical clustering / nearest-neighbour graphs |

Coordinates come in two variants: **scaled** (the standard one for distances and modelling) and **unscaled**. This tool uses **scaled** coordinates by default.

---

## Features

### 🔎 Distance & closest populations
- Rank the N closest populations/individuals to any target (yours or a reference).
- Filter by dataset (modern / ancient / custom), by name (fuzzy search), by region or country, or by time period for ancient samples.
- Show distance as raw Euclidean or as a percentage "fit".
- **Multi-target** mode shows the closest populations for several targets side by side.

### 🧪 Admixture modelling (nMonte / Vahaduo-style)
- Model a target as a weighted mix of 2…N source populations.
- Solves a constrained least squares problem (weights ≥ 0, sum = 1) to minimise the distance to the target. You get results in milliseconds, even with hundreds of sources.
- **Auto-source selection**: give it a large pool and it keeps only the sources that matter.
- Adjustable **penalty** / **cycle multiplier**, matching the familiar Vahaduo options.
- Per-source breakdown with bars, plus a **per-PC residual view** showing which dimensions fit badly.
- Can **aggregate** results by prefix or group (e.g. collapse `Italian_Tuscany`, `Italian_Lombardy` → `Italian`).

### 🧫 Population mixing ("what if")
- Build synthetic populations from weighted components, e.g. `0.5 × Greek + 0.25 × Turkish + 0.25 × Levantine`.
- Save them as named custom samples and use them as targets or sources anywhere in the app.
- Average several individuals into a single group coordinate.

### 📈 Visualisations (in the terminal)
- **PCA scatter plot** of any two PCs using braille/half-block rendering, with zoom and pan, and labels that avoid overlapping each other.
- **Distance heatmap** between a selection of populations.
- **Choropleth map** (world, Europe, MENA, Caucasus, South Asia, East Asia, …) coloured by distance to your target, rendered from the bundled SVG maps.
- **Cluster tree** (dendrogram) and nearest-neighbour graph.
- Bar charts for admixture results.

### ⚡ Workflow
- Everything is keyboard-driven, with vim-style keys and a command palette (`:`).
- Paste coordinates straight from the clipboard, in the same comma-separated format the web tools use.
- Session state is saved automatically (your samples, last models, open tabs).
- Export any result as **CSV, JSON, Markdown, or a PNG/SVG plot**.
- Every screen also has a **non-interactive CLI subcommand**, so you can script it and pipe results to other tools.
- 100% offline, and your DNA coordinates never leave your machine.

---

## Installation

Requires Rust 1.85+ (edition 2024).

```bash
git clone <this-repo> g25
cd g25/dna_g25_analyzer_with_rust
cargo install --path .
```

Or run straight from source:

```bash
cargo run --release
```

---

## Quick start

```bash
# Launch the interactive app (loads the bundled datasets in ./data/g25)
g25

# Add your own coordinates (copy the line your G25 provider gave you)
g25 add "Me,0.123,0.140,0.045,..."

# Closest 20 modern populations
g25 dist Me --dataset modern --top 20

# Model yourself with a pool of ancient sources
g25 model Me --sources ancient --top-sources 6

# Create a synthetic population
g25 mix "Greek:0.5,Turkish:0.25,Levantine:0.25" --name MyMix
```

---

## The interactive TUI

```
┌ g25 ─────────────────────────────────────────────────────────────────────────┐
│ [1] Samples  [2] Distance  [3] Model  [4] Mix  [5] PCA  [6] Map  [7] Heatmap │
├──────────────────────┬───────────────────────────────────────────────────────┤
│ Target: Me           │  Closest populations (modern, scaled)                 │
│ ──────────────────── │  #  Population                 Distance               │
│ ▸ Me                 │  1  Greek_Thessaly             0.02841  ██████████▏   │
│   MyMix              │  2  Greek_Peloponnese          0.02977  █████████▊    │
│   Father             │  3  Italian_Calabria           0.03310  █████████     │
│                      │  4  Albanian                   0.03455  ████████▋     │
│ Filters              │  5  Macedonian_Greek           0.03520  ████████▌     │
│  dataset: modern     │  …                                                    │
│  region:  *          │                                                       │
├──────────────────────┴───────────────────────────────────────────────────────┤
│ / search  f filter  m model-from-here  e export  : command  ? help  q quit   │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Key bindings (default)

| Key | Action |
|---|---|
| `1`–`7` / `Tab` | Switch tabs |
| `j` `k` / `↑` `↓` | Move selection |
| `/` | Fuzzy search populations |
| `f` | Open filter panel |
| `Space` | Toggle selection (for sources, heatmap, mixes) |
| `Enter` | Set as target / open details |
| `m` | Send selection to the modeller |
| `x` | Send selection to the mixer |
| `p` | Paste coordinates from clipboard |
| `e` | Export current view |
| `+` `-` / `h` `l` | Zoom / pan in plots and maps |
| `:` | Command palette (`:dist`, `:model`, `:mix`, `:load <file>`, …) |
| `?` | Help |
| `q` | Quit |

Key bindings and colour themes can be changed in `~/.config/g25/config.toml`.

---

## Command-line mode

Every feature is also available as a subcommand for scripting:

```text
g25                          Launch the TUI
g25 add <coords|file>        Add samples to your personal sample store
g25 list [--dataset D]       List samples/populations
g25 dist <target> [opts]     Closest populations
g25 model <target> [opts]    Admixture model
g25 mix <spec> --name N      Create a weighted mix
g25 avg <a,b,c> --name N     Average samples into a group
g25 pca <targets> --pc 1,2   Render a PCA plot (terminal / --out plot.png)
g25 heatmap <a,b,c,...>      Distance matrix / heatmap
g25 map <target> --map world Choropleth distance map
g25 export <what> --format csv|json|md
```

Common options: `--dataset modern|ancient|all|<file>`, `--top N`, `--filter <regex>`, `--json`.

---

## Data files

The repository ships with a ready-to-use reference set in [`data/`](data/). It comes from the datasets used by [mygeneticmaps.com](https://mygeneticmaps.com/), which is based on the **Moriopoulos Collection** of G25 scaled coordinates.

```
data/
├── g25/
│   ├── modern_moriopoulos_2026.csv     # 3,227 modern populations/samples
│   ├── ancient_moriopoulos_2026.csv    # 5,071 ancient populations/samples
│   ├── map_refs_<region>.csv           # per-country reference averages used for each map
│   │                                   #   (world, mena, caucasus, southasia, eastasia,
│   │                                   #    siberia, macaronesia, southerncone, rome, caribbean)
│   ├── extra_47tolkien.csv             # 163 extra regional averages (47Tolkien)
│   └── extra_famous_sim.csv            # 24 simulated "famous people" coordinates (for fun)
├── maps/
│   ├── <region>.svg                    # map outlines (MapChart-based)
│   ├── <region>.mapping.json           # reference population → map region ids
│   └── map-data-free.json              # original bundle (svg + sources + mapping)
└── meta/
    ├── population_to_country.json      # population label → country
    └── country_display_names.json      # country id → short display name
```

You can drop any other G25 file (e.g. Davidski's official modern/ancient scaled files, or your own
spreadsheet export) into `data/g25/`. It will be picked up automatically, or you can load it with `:load <file>`.

---

## File formats

### G25 coordinate file (CSV)

The standard format used by Vahaduo and most G25 tools. There is **no header**. Each line holds a label and 25 comma-separated values:

```
Population_Name,PC1,PC2,PC3,...,PC25
Greek_Thessaly,0.1245,0.1398,0.0213,...,0.0012
```

- Labels may carry a sample count suffix such as `Xegwi_(n=3)`.
- Individual samples are often written `Population:SampleID`.
- Lines starting with `#` are ignored.

### Mix spec

```
Greek:0.5,Turkish:0.25,Levantine:0.25
```

Weights are normalised automatically, so `Greek:2,Turkish:1,Levantine:1` gives the same result.

---

## How the math works

**Distance** between two samples *a* and *b* is plain Euclidean distance over the 25 PCs:

```
d(a, b) = sqrt( Σᵢ (aᵢ − bᵢ)² ),  i = 1..25
```

**Mixing** is a weighted average: `mix = Σⱼ wⱼ · sourceⱼ`, with `Σ wⱼ = 1`.

**Admixture modelling** looks for the weights *w* that minimise
`‖ target − Σⱼ wⱼ · sourceⱼ ‖²` subject to `wⱼ ≥ 0` and `Σ wⱼ = 1`.
This is the same objective that nMonte and Vahaduo optimise. We solve it with an exact active-set NNLS solver instead of Monte-Carlo sampling, so results are deterministic and fast. An optional Vahaduo-compatible stochastic mode is planned for reproducing web results exactly.

**Fit** is reported as the distance between the target and the fitted model. Lower is better. As a rough rule of thumb, `< 0.02` is excellent, `< 0.04` is good, and `> 0.05` means the sources probably don't explain the target well.

---

## Project layout

```
.
├── README.md
├── data/                           # bundled reference datasets (see above)
└── dna_g25_analyzer_with_rust/     # the Cargo crate
    ├── Cargo.toml
    └── src/
        ├── main.rs                 # CLI entry point (clap)
        ├── tui/                    # ratatui app: tabs, widgets, key handling
        ├── data/                   # parsing, dataset registry, sample store
        ├── analysis/               # distance, NNLS modelling, mixing, clustering
        └── render/                 # plots, heatmaps, SVG map rasteriser, export
```

Planned core crates: `ratatui` + `crossterm` (TUI), `clap` (CLI), `nucleo` (fuzzy search), `rayon` (parallel distance scans), `serde` / `csv` (I/O), `resvg` / `usvg` (map rendering), `arboard` (clipboard), `plotters` (PNG/SVG export).

---

## Roadmap

- [x] Collect reference datasets (modern + ancient + map references)
- [ ] G25 CSV parser & dataset registry
- [ ] Distance / closest populations (CLI)
- [ ] NNLS admixture modeller (CLI)
- [ ] Mixing & averaging
- [ ] TUI shell: tabs, sample list, fuzzy search, filters
- [ ] Distance & model views in TUI
- [ ] PCA scatter plot widget
- [ ] Heatmap & cluster tree
- [ ] Terminal choropleth maps from bundled SVGs
- [ ] Export (CSV / JSON / Markdown / PNG)
- [ ] Vahaduo-compatible stochastic mode
- [ ] Config file, themes, custom key bindings

---

## Credits & data attribution

- **Global25**: created by Davidski / [Eurogenes](https://eurogenes.blogspot.com/).
- **Reference coordinates**: the *Moriopoulos Collection 2025/2026*, as used and published by
  [MyGeneticMaps](https://mygeneticmaps.com/). Extra regional averages were contributed by 47Tolkien.
- **Map outlines**: based on [MapChart.net](https://www.mapchart.net/) SVGs, via MyGeneticMaps.
- The UX takes inspiration from [Vahaduo](https://vahaduo.github.io/) and MyGeneticMaps.

The bundled data belongs to its respective authors and is included for personal and educational use. If you redistribute this project, please check with the data owners first.

---

## Disclaimer

G25 coordinates are a statistical summary, not a genealogy. Distances and admixture percentages depend on which reference populations are available and which ones you pick. Different models can fit almost equally well. Treat the results as exploratory, not definitive.
