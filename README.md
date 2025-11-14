# ColabDNMP v1.0

> Google Colab pipeline to **design and evaluate de novo binders** against any protein target, integrating **P2Rank**, **RFdiffusion**, **ProteinMPNN** and **AlphaFold2**.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USER/REPO/blob/main/ColabMiniprot.ipynb)

---

## 🧠 What is ColabMiniprot?

**ColabMiniprot** is an end-to-end workflow that:

1. Takes a **target protein structure** (PDB/mmCIF).
2. Predicts **surface pockets** with **P2Rank**.
3. Identifies **hotspot residues** in a chosen pocket using per–residue SASA.
4. Trims the target around those hotspots to define a **binding site**.
5. Uses **RFdiffusion** (backbones) + **ProteinMPNN** (sequence design) to generate **candidate binders**.
6. Evaluates target–binder complexes with **AlphaFold2** and interface metrics.

All the heavy lifting (installations, paths, file conversions, etc.) is automated inside the Colab notebook and linked to your Google Drive.

---

## 🧬 Pipeline overview

The notebook is organized into logical blocks:

1. **Project setup (Google Drive)**  
   - Asks for a `project_name`.  
   - Creates a working directory in your Drive:  
     `MyDrive/<project_name>/`  
   - Subfolders:
     - `01_target`
     - `02_p2rank`
     - `03_hotspots`
     - `04_trimming`
     - `05_rfdiffusion`
     - `06_sequences`
     - `07_alphafold`
     - `08_metrics`

2. **Upload target structure 🎯**  
   - You upload a **PDB** or **(mm)CIF** file.
   - The notebook:
     - Copies it into `01_target/`.
     - Converts CIF → PDB if needed.
     - Standardizes the name to:  
       `target_input.pdb` (both locally and in Drive).

3. **Pocket prediction with P2Rank 🔍**  
   - Downloads and installs **P2Rank** (if not present).
   - Runs P2Rank on `target_input.pdb`.
   - Stores results in `p2rank_output/target_input.pdb_predictions.csv`.
   - Loads and cleans the CSV into a `pandas` DataFrame.

4. **Pocket selection & hotspot detection 💥**  
   - You choose:
     - `pocket_rank` (e.g. 1 = top-ranked pocket),
     - `sasa_cutoff`,
     - residue filters:
       - `none`
       - `aromatic_only`
       - `hydrophobic_only`
       - `aromatic_or_hydrophobic`
   - The notebook:
     - Extracts `residue_ids` for the chosen pocket.
     - Computes **per–residue SASA** (Biopython + Shrake–Rupley).
     - Ranks pocket residues by exposure / filters them by chemistry.
     - Prints the **top hotspot residues**.

5. **Binding-site trimming ✂️**  
   - Input:
     - `chain_id` (e.g. `"A"`)
     - `hotspots_str` (e.g. `"A79, A145, A173"`)
     - `flank` (±N residues around min/max hotspot index)
   - The notebook:
     - Parses the hotspot list.
     - Extracts a **contiguous sequence window** from `target_input.pdb`.
     - Writes `target_trimmed_seqwin.pdb` for downstream modeling.

6. **RFdiffusion setup (binder backbone design)**  
   - Installs:
     - `RFdiffusion` repo.
     - Required Python packages.
     - Schedules and checkpoints.
   - Downloads **AlphaFold2 parameters** into `params/`.
   - Prepares everything to:
     - Generate **de novo binder backbones**.
     - Or use **existing backbone batches** from Drive (e.g. `MyDrive/RFdiffusion_CDA/`).

7. **Backbone discovery & configuration**  
   - Automatically scans common locations in Drive and local paths for PDB backbones:
     - `MyDrive/RFdiffusion_CDA/*.pdb`
     - `MyDrive/RFdiffusion_CDA/CDA_run_b*/*.pdb`
     - `outputs/*.pdb`, etc.
   - Chooses a default `BACKBONE_GLOB` pattern (first pattern with hits).
   - Prints examples of detected backbone files.

8. **Sequence design with ProteinMPNN + AF2 prediction 🧬 → 🧩**  
   - For each backbone:
     - Runs **ProteinMPNN** to generate multiple binder sequences (`NUM_SEQS`).
     - Optionally uses an **initial guess** sequence.
     - Removes specific amino acids from the design alphabet (e.g. `RM_AA="C"`).
   - For each designed sequence:
     - Builds a **target–binder complex** model using **AlphaFold2**:
       - Monomer or Multimer mode (`USE_MULTIMER`).
       - Configurable `NUM_RECYCLES`.
   - Saves outputs (PDBs, JSONs…) under:
     - `06_sequences/`
     - `07_alphafold/`

9. **Visualization & interactive selection 🧿**  
   - Uses **py3Dmol** to:
     - Visualize target + binder.
     - Switch between best designs via dropdown.
   - Loads best-scoring models (e.g., `best_designX.pdb`).

10. **Scoring, metrics & downloads 📊📦**  
    - Additional packages:
      - `biopython`
      - `freesasa`
      - `mdanalysis`
      - `pandas`, `numpy`
      - (optionally) `pyrosetta` via `pyrosetta-help`
    - Can compute:
      - Interface / buried SASA.
      - Contact counts.
      - Simple quality metrics.
      - Optional PyRosetta-based energy terms.
    - Packs results:
      - Zips all outputs into `<run_name>.result.zip`.
      - Provides a direct download via Colab.

---

## 🚀 How to use

1. Open the notebook in Google Colab (GPU runtime recommended).  
2. Run the cells **from top to bottom**, in order:
   - Define `project_name` and create folders.
   - Upload your target structure.
   - Run pocket prediction with P2Rank.
   - Select a pocket and compute hotspots.
   - Trim the binding site.
   - Set up RFdiffusion (and/or provide backbones).
   - Run ProteinMPNN + AlphaFold2.
   - Inspect, score and download designs.

3. Check your Google Drive:
   - All intermediate files and results will be neatly organized in:
     `MyDrive/<project_name>/01_target ... 08_metrics`.

---

## 📦 Requirements

Because everything runs inside Colab, you **don’t need** to pre-install tools locally. You only need:

- A **Google account** and **Google Drive**.
- Colab runtime with:
  - Python 3
  - GPU (recommended for RFdiffusion & AlphaFold2).

The notebook itself takes care of:

- Installing **P2Rank**.
- Cloning and configuring **RFdiffusion**.
- Installing **ProteinMPNN**, `biopython`, `freesasa`, `mdanalysis`, etc.
- Downloading **AlphaFold2 parameters**.

---

## 📁 Folder structure (in Drive)

```text
MyDrive/
└── <project_name>/
    ├── 01_target/
    ├── 02_p2rank/
    ├── 03_hotspots/
    ├── 04_trimming/
    ├── 05_rfdiffusion/
    ├── 06_sequences/
    ├── 07_alphafold/
    └── 08_metrics/
