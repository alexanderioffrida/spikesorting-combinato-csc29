# Spike sorting `CSC29.ncs` with SpikeInterface + Combinato

This repo reproduces the [Combinato **Real Data** tutorial](https://github.com/jniediek/combinato/wiki/Tutorial-Real-Data)
— spike sorting a single-channel human hippocampal Neuralynx recording (`CSC29.ncs`) —
but driven entirely from **SpikeInterface** instead of Combinato's `css-*` command-line tools.

The full pipeline lives in **[`combinato_spikeinterface_CSC29.ipynb`](combinato_spikeinterface_CSC29.ipynb)**
(outputs are committed, so it renders with plots on GitHub).

## What the notebook does

Each Combinato step is mapped to its SpikeInterface equivalent:

| # | Combinato step | SpikeInterface |
|---|---|---|
| 1 | load `.ncs` | `read_neuralynx` + a 1-contact dummy probe |
| 2 | `css-extract` (filter) | `bandpass_filter(300, 3000)` *(visualization)* |
| 3 | `css-extract` (detect) | `detect_peaks(detect_threshold=5, peak_sign="neg")` *(visualization)* |
| 4 | `css-mask-artifacts` | amplitude-based reporting *(see note below)* |
| 5 | `css-simple-clustering` | `run_sorter("combinato", ...)` |
| 6 | `css-plot-sorted` | `SortingAnalyzer` + plotting widgets |
| 7 | `css-gui` | export to **phy** for manual curation |

The actual sort (step 5) calls the real Combinato through SpikeInterface's external-sorter
wrapper, which runs `css-extract` + `css-simple-clustering`. Combinato does its own
filtering (elliptic bandpass: 300–1000 Hz detection, 300–3000 Hz extraction, ~2 kHz notch)
and detection, so it receives the **raw** recording. The filtering/detection in steps 2–3
are SpikeInterface re-implementations shown for understanding only.

> **Note:** the wrapper does **not** run `css-mask-artifacts` — it only invokes `css-extract`
> and `css-simple-clustering` (confirmed in `spikeinterface/sorters/external/combinato.py` and
> by `No artifacts defined` in the run log). Step 4 therefore only *reports* candidate artifacts.

Result on `CSC29.ncs`: **4 units** (negative threshold, `detect_threshold=5`).

## Setup

### 1. Python environment

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt   # phy is a pre-release; if pip refuses it, run: pip install --pre phy
```

Tested on macOS (Apple Silicon), Python 3.13.

### 2. Combinato (not on PyPI)

```bash
git clone https://github.com/jniediek/combinato
cd combinato
python3 setup_options.py          # run once; creates options.py
```

Then tell the notebook where it lives (the notebook reads `COMBINATO_PATH`,
falling back to `~/combinato`):

```bash
export COMBINATO_PATH=/absolute/path/to/combinato
```

Combinato's `css-*` scripts run as subprocesses and import `tables` (PyTables) and
`pywt` (PyWavelets) — both are in `requirements.txt`, so install them into the **same**
environment you launch Jupyter from.

### 3. Data

`CSC29.ncs` (~118 MB) is **not** committed. Download it from the
[Combinato tutorial wiki](https://github.com/jniediek/combinato/wiki/Tutorial-Real-Data)
and place it at:

```
data_real/CSC29.ncs
```

## Run

```bash
source .venv/bin/activate
jupyter lab combinato_spikeinterface_CSC29.ipynb
```

Run the cells top to bottom. The Combinato sort takes ~70 s.

### Manual curation in phy

After step 6, export a phy dataset and launch the GUI **from a terminal** (not inside Jupyter —
it opens its own window and blocks):

```python
si.export_to_phy(analyzer, output_folder="results/phy_CSC29", remove_if_exists=True)
```
```bash
.venv/bin/phy template-gui results/phy_CSC29/params.py
```

## Repo layout

```
combinato_spikeinterface_CSC29.ipynb   # the pipeline (with outputs)
requirements.txt                       # pip environment
README.md
.gitignore                             # excludes data_real/, results/, .venv/
data_real/    (gitignored)  CSC29.ncs  # download separately
results/      (gitignored)             # sorter output, phy export, analyzer
```
