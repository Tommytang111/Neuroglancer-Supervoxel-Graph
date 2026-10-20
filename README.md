# CREMI Neuroglancer segmentation editor

This standalone Python application displays a small CREMI electron-microscopy
crop, represents its neuron segmentation with a supervoxel graph, and supports
persistent merge and split edits from the Neuroglancer client.

## Requirements

- Python 3.11 or newer
- A CREMI HDF5 file containing:
  - `volumes/raw`
  - `volumes/labels/neuron_ids`

The default data location is `data/sampleA.h5` beside `app.py`.

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
python app.py --data data/sampleA.h5
```

The application prints a Neuroglancer URL and normally opens it in the default
browser. Keep the terminal process running while using the viewer.

For a CREMI file elsewhere:

```bash
python app.py --data /path/to/sample_A_20160501.hdf
```

Useful options:

```bash
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
- `edits.json`: current graph plus edit history

The application saves `edits.json` after every successful merge or split and
loads it at the next launch. It refuses to apply state to an incompatible crop.
Use a different `--state-dir` for a different crop, or pass `--reset-state` when
you intentionally want to discard the existing state.

## Deployment notes

The default server binds to `127.0.0.1`, so only the same computer can connect.
This is the safest mode for a take-home demonstration.

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

## First-run behavior

On the first run, the application starts Neuroglancer before constructing the
supervoxels. The browser therefore shows the supervoxels appearing as blocks
are processed. It then builds the adjacency graph and switches to the finished
two-panel editor. Later runs restore the cached volume and saved graph directly.
