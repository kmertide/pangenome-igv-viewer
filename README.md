# JupyterLab IGV Viewer — Quick Start Guide

**Author**: Andrew Blair

Interactive browser-based genome visualization of HPRC v2.0 pangenome variant calls
across the GEUVADIS cohort (397 samples), built on IGV.js running inside JupyterLab.

---

## Prerequisites

- **Cluster access** — a Phoenix HPC account with access to `/private/groups/patenlab/shared/plotly_dashbio_igv/`
- **conda / miniconda** — must be installed in your home directory on the cluster.
  If you don't have it, install [Miniconda](https://docs.conda.io/en/latest/miniconda.html):
  ```bash
  wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
  bash Miniconda3-latest-Linux-x86_64.sh -b -p ~/miniconda3
  ~/miniconda3/bin/conda init bash
  # Log out and back in, then verify:
  conda --version
  ```

---

## 1. Environment Setup

Build the conda environment from the provided YAML file (**one-time setup, ~10–15 min**):

```bash
conda env create -f /private/groups/patenlab/shared/plotly_dashbio_igv/jupyterlab-nbi-plotly_dashbio-igv.yml
```

This creates an environment named `plotly_dashbio_igv`. Verify it installed:

```bash
conda env list
```

> You only need to do this once. The YAML file is at:
> `/private/groups/patenlab/shared/plotly_dashbio_igv/jupyterlab-nbi-plotly_dashbio-igv.yml`

---

## 2. Launch JupyterLab on the Cluster

A convenience script `run.sh` is provided. First find an available node:

```bash
sinfo -p medium -o "%N %t" | grep idle
```

Then launch JupyterLab by running `run.sh` with `srun`, replacing `phoenix-22`
with an available node from the output above:

```bash
srun --partition=medium \
     --nodelist=phoenix-22 \
     --cpus-per-task=4 \
     --mem=16G \
     --time=4:00:00 \
     --pty bash -c '
  source ~/miniconda3/etc/profile.d/conda.sh
  conda activate plotly_dashbio_igv
  cd /private/groups/patenlab/shared/plotly_dashbio_igv
  jupyter lab --no-browser --ip=0.0.0.0 --port=9191 \
    --ServerApp.token="mytoken123" \
    --ServerApp.password=""
'
```

> **If using miniforge3 instead of miniconda3**, replace the `source` line with:
> ```bash
> source ~/miniforge3/etc/profile.d/conda.sh
> ```

Note the node name printed in your terminal (e.g. `phoenix-22`) — you will need
it in the next step.

---

## 3. Connect from Your Local Machine

Open a **new terminal on your local machine** (not the cluster) and run:

```bash
ssh -L 9191:<node>.prism:9191 -J emerald.prism <your_username>@phoenix.prism
```

Replace `<node>` with the node from step 2 (e.g. `phoenix-22`) and
`<your_username>` with your cluster username. Keep this terminal open.

Then open your browser and go to:
```
http://localhost:9191
```

When prompted for a token, enter: `mytoken123`

---

## 4. Open the Notebook

In the JupyterLab file browser on the left, navigate to:
```
plotly_dashbio_igv_hprc_r2_pangenie_geuvadis_eqtl.ipynb
```

---

## 5. Run the Notebook

Run all cells from top to bottom using **Shift + Enter** on each cell, or:

**Kernel → Restart Kernel and Run All Cells**

The IGV viewer will appear as an inline output in the last cell. It may take
a few seconds to load the tracks.

---

## 6. Using the IGV Viewer

### Navigating to a Locus
Use the search bar at the top of the IGV panel:
- Type a gene name: `BRCA2`
- Type a region: `chr15:50,000,000-51,000,000`
- Type a chromosome: `chr15`

### Tracks Displayed
| Track | Description |
|---|---|
| **Gencode v47** | Human gene annotation (GRCh38) |
| **HPRC v2.0 GRCh38 Cohort (397)** | Variant calls from PanGenie genotyping across 397 GEUVADIS samples |

### Changing the Default Locus
To change the region that loads on startup, edit the `locus` line
in the last cell before running:

```python
locus="chr15"                         # whole chromosome view
locus="chr15:50,000,000-51,000,000"  # specific region
locus="BRCA2"                         # gene name
```

---

## 7. Troubleshooting

**Port already in use**
The first cell automatically kills any process on port 8002 before starting.
If you see an error, simply rerun that cell.

**IGV viewer does not appear**
- Confirm all cells ran without errors
- Try Kernel → Restart Kernel and Run All Cells
- Check that the SSH tunnel terminal on your local machine is still open

**Tracks not loading / spinning**
The cohort VCF is ~30GB and is served on-demand — only data at the current
locus is fetched. Allow a few seconds when navigating to a new region.

**"Port is free" but nothing loads**
Make sure the SSH tunnel command is running in a terminal on your **local machine**
and that the node name in the tunnel matches the node from `srun`.

**conda activate fails**
Make sure you ran `source ~/miniconda3/etc/profile.d/conda.sh` (or `miniforge3`)
before trying to activate the environment.

---

## Reference

| Item | Details |
|---|---|
| Notebook | `plotly_dashbio_igv_hprc_r2_pangenie_geuvadis_eqtl.ipynb` |
| Conda env YAML | `jupyterlab-nbi-plotly_dashbio-igv.yml` |
| Conda env name | `plotly_dashbio_igv` |
| Launch script | `run.sh` |
| Assets | `assets/` |
| Reference genome | GCA_000001405.15 GRCh38 no alt analysis set |
| Variant data | HPRC v2.0 MC graph, PanGenie genotypes, 397 samples |
| Gene annotation | Gencode v47 |
| Port | 8000 (Flask file server) / 9191 (JupyterLab) |
| Contact | apblair@ucsc.edu |
