# CREMI Neuroglancer segmentation editor

This standalone Python application displays a small CREMI electron-microscopy
crop, represents its neuron segmentation with a supervoxel graph, and supports
persistent merge and split edits from the Neuroglancer client.

## Requirements

- Python 3.11 or newer
- Internet access on the first run so `torch_em` can download CREMI sample A

The default data location is `data/sampleA.h5` beside `app.py`. If that file
does not exist, the application automatically downloads CREMI sample A before
loading the crop. Existing files are reused and are not downloaded again.

## Install

Create a virtual environment and install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

On Windows PowerShell, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

## Run

From this directory:

```bash
python app.py
```

On the first run, `torch_em` downloads approximately 167 MB to
`data/sampleA.h5`. A byte-level progress bar is forced onto standard output so
it remains visible in consoles that hide standard error.

For a CREMI file elsewhere:

```bash
python app.py --data /path/to/file_of_interest.h5
```

An explicitly supplied `--data` path must already exist. Automatic downloading
is limited to the default `data/sampleA.h5` location so the application never
silently ignores a custom path.

Useful options:

```bash
#Run on default settings
python app.py

# Use an explicit crop in Z,Y,X order.
python app.py --data data/sampleA.h5 --crop 0:32,433:817,433:817

# Print the URL without opening a browser.
python app.py --data data/sampleA.h5 --no-open-browser

# Build or validate cached state without launching Neuroglancer.
python app.py --data data/sampleA.h5 --prepare-only

# Explicitly discard this app's cache and edit history, then rebuild it.
python app.py --data data/sampleA.h5 --reset-state
```

See every option with:

```bash
python app.py --help
```

## Default data configuration

- **Crop:** `Z=0:32, Y=433:817, X=433:817` in the CREMI HDF5 file's
  `Z,Y,X` order. After conversion to Neuroglancer's `X,Y,Z` order, the array
  shape is `384 x 384 x 32` voxels.
- **Voxel resolution:** CREMI Sample A reports `40 x 4 x 4 nm` in `Z,Y,X`
  order, which the application converts to `4 x 4 x 40 nm` in `X,Y,Z` order.
  The value is read from the HDF5 metadata rather than hard-coded.
- **Supervoxel block size:** `64 x 64 x 8` voxels in `X,Y,Z` order. At the
  CREMI resolution, each block spans approximately `256 x 256 x 320 nm`.

The default crop size and block size can be changed with `--crop-size` and
`--block-shape`, or an exact crop can be selected with `--crop`.

## Controls

| Input | Action |
|---|---|
| Shift + left-click | Select seed A (green) |
| Control/Option + left-click | Select seed B (magenta) |
| `M` | Merge the objects containing A and B |
| `S` | Split one object between A and B using a minimum graph cut |
| `C` | Clear both seeds |

Selecting seeds also selects their current graph components, causing their
meshes to appear in the 3-D panel.

## Generated state

By default, generated files are stored in `state/` beside `app.py`:

- `supervoxels_xyz.npy`: deterministic supervoxel volume
- `preprocessing.json`: crop, resolution, and content fingerprint
- `state/edits.json`: current graph plus edit history

The application saves `state/edits.json` after every successful merge or split
and loads it at the next launch. A custom `--state-dir` changes this location to
`<state-dir>/edits.json`. The application refuses to apply state to an
incompatible crop. Use a different `--state-dir` for a different crop, or pass
`--reset-state` when you intentionally want to discard the existing state.

## Deployment notes

The default server binds to `127.0.0.1`, so only the same computer can connect.

For a controlled remote machine you may use Neuroglancer's server options:

```bash
python app.py \
  --data data/sampleA.h5 \
  --bind-address 0.0.0.0 \
  --bind-port 8080 \
  --no-open-browser
```

The viewer URL contains a random secret that grants access to the Python-backed
data source. Do not publish or reuse that URL. Production internet deployment
should additionally place the application behind TLS and access controls.
