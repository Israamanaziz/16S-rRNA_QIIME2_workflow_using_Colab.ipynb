# 16S rRNA QIIME2 workflow in Google Colab

This notebook contains a full microbiome analysis with **QIIME 2** in Jupyter notebook on the web. You do not need to install anything on your computer.

So, the basic workflow goes from raw sequencing reads to a taxonomy table, diversity results, and a heatmap, all using bash to run commands one must use "!" for bash scripting in Jupyter. 

The workflow is reproduced from: [ISB course 2024](https://github.com/gibbons-lab/isb_course_2024). 
The original 2024 setup fails in Colab because of a package clash and version incompatibility.

My fix: This notebook fixes it by installing QIIME 2 in its own separate environment and uses the latest classifier for taxonomy along with complete end-to-end bash scripting for 16S rRNA.

---
# 1. Who is this for?

This notebook is for you if:

- You are new to QIIME 2 and feel a bit overwhelmed with linux.
- Your laptop is slow, old, or does not have much memory or storage.
- You use Windows, and installing QIIME 2 on your computer feels hard.
- You want to try QIIME 2 quickly before you install it for real.
- You are following the ISB course 2024 and the original setup did not work for you.

## 2. Why Google Colab?

QIIME 2 is a large program. Installing it on your own computer can be hard because:

- It needs a lot of disk space and memory.
- It works best on Linux or Mac. Windows users often need extra tools.
- Installs can fail because of package clashes.

Think of it like this: Colab is a rented computer for a few hours. You set it up, do your work, download your results, and give it back.

## 3. Quick start

1. Click the **Open in Colab** badge at the top of this page.
2. Sign in with your Google account if asked.
3. Run the cells **one by one, from top to bottom** (see the next section).
4. Wait for each cell to finish before you run the next one.
5. At the end, download your results.

Tips:

- **Run the cells in order.** Do not skip any. Each cell depends on the ones before it.
- Lines that start with `!` run **commands** (like `!qiime info`). Lines without `!` run **Python**.
- Lines that start with `#` are **comments**. They are notes for the users. The computer ignores them.
- Red text does not always mean failure. Some tools print warnings. If a cell stops with an error, see [Problems and fixes](#10-problems-and-fixes).

---
## QIIME WORKFLOW
### Part 1: Set up

| Step | What it does | Time |
|------|--------------|------|
| Install Miniforge | Installs a small tool (conda) that can install QIIME 2 | about 1 minute |
| Install QIIME 2 | Builds a separate space (an *environment*) called `qiime2` | about 5 to 15 minutes |
| Set the PATH | Makes the `qiime` command work in the notebook | seconds |
| `qiime info` | Checks that QIIME 2 is installed. You should see a version and a list of plugins | seconds |


### Part 2: Get the data

The notebook downloads the course files from GitHub into a folder called `materials`. It then shows two tables:

- **Manifest**: says which sequence file belongs to which sample.
- **Metadata**: describes each sample (for example, which group it is in).

### Part 3: Import the reads and check quality

QIIME 2 does not read plain sequence files directly. First, it packs them into its own format (`.qza`). Then it makes a **quality report** (`.qzv`) that shows how good your reads are. You use this report to decide where to trim the reads.

### Part 4: Clean the reads (DADA2)

DADA2 removes sequencing errors. It gives you a table that counts each unique sequence in each sample.

- `--p-trunc-len 150` cuts every read to 150 bases. This number can be changed according to the quality report.

### Part 5: Compare samples

- **Tree**: shows how related the sequences are.
- **Diversity**: measures how varied each sample is, and how different samples are from each other.
  - `--p-sampling-depth 5000` uses 5000 reads per sample so the comparison is fair. Samples with fewer reads are left out.
- **PERMANOVA**: a test that asks, "Are the groups different?" It uses the metadata column `parkinson_disease`.

### Part 6: Find out which bacteria are present

The notebook downloads a ready-made **classifier**. It compares your sequences to a database of known bacteria (Greengenes). You get:

- A **taxonomy table** (what each sequence is).
- A **bar plot** (which bacteria are in each sample).
- A **heatmap** (basically telling which genera are more or less common in each sample.)

---

## 4. How to open your results

Files ending in `.qzv` are QIIME 2 **visualizations**. Colab cannot show them directly.

1. Download the `.qzv` file (use the zip from Part 8, or the **folder icon** on the left of Colab, then right-click a file and choose **Download**).
2. Go to [view.qiime2.org](https://view.qiime2.org).
3. Drag the file onto the page. View interactive plots

## 5. Using your own data

When you are comfortable, you can use this notebook for your own project. Keep **Part 1** (the setup) and replace **Part 2** with your own data.

### Step 1: Get your files into Colab

- **Quick way**: click the **folder icon** on the left, then drag your files in. They are deleted when the session ends.
- **Safer way**: use Google Drive, so your files and results are saved:

```python
from google.colab import drive
drive.mount('/content/drive')
```

### Step 2: Make a manifest file

The manifest is a **tab-separated** text file (`manifest.tsv`).

**Single-end reads** (one file per sample):

```
sample-id	absolute-filepath
sample1	/content/my_data/sample1.fastq.gz
sample2	/content/my_data/sample2.fastq.gz
```

**Paired-end reads** (two files per sample):

```
sample-id	forward-absolute-filepath	reverse-absolute-filepath
sample1	/content/my_data/sample1_R1.fastq.gz	/content/my_data/sample1_R2.fastq.gz
sample2	/content/my_data/sample2_R1.fastq.gz	/content/my_data/sample2_R2.fastq.gz
```

### Step 3: Make a metadata file

A tab-separated file (`metadata.tsv`). The first column must be called `sample-id`. The other columns describe your samples (group, treatment, time, and so on).

### Step 4: Change these settings

| What | Change it to |
|------|--------------|
| Import type (paired-end) | `SampleData[PairedEndSequencesWithQuality]` with `PairedEndFastqManifestPhred33V2` |
| Cleaning command (paired-end) | `qiime dada2 denoise-paired` with `--p-trunc-len-f` and `--p-trunc-len-r` |
| Trim length | Choose from your own quality report |
| Sampling depth | Choose from your own table, so most samples are kept |
| PERMANOVA formula | A column name from your own metadata |
| Classifier | One that matches your database and the 16S region you sequenced |

If you are stuck, copy the **full error message** and ask on the [QIIME 2 Forum](https://forum.qiime2.org). Include what you ran and what you saw.

## 5. Small glossary

| Word | Meaning |
|------|---------|
| **QIIME 2** | A program for analysing microbiome data |
| **16S** | A gene used to identify bacteria |
| **Reads** | The short DNA sequences from the sequencer |
| **FASTQ** | A file format for reads and their quality |
| **Manifest** | A table that links sample names to read files |
| **Metadata** | A table that describes your samples |
| **`.qza`** | A QIIME 2 data file |
| **`.qzv`** | A QIIME 2 visualization file (open at view.qiime2.org) |
| **Environment** | A separate space with its own programs, so they do not clash |
| **conda / Miniforge** | Tools that install programs into environments |
| **PATH** | The list of places where the computer looks for commands |
| **DADA2** | A tool that removes sequencing errors |
| **Alpha diversity** | How varied one sample is |
| **Beta diversity** | How different two samples are |
| **PERMANOVA** | A test for whether groups are different |
| **Taxonomy** | The names of the bacteria (such as genus) |
| **Classifier** | A trained model that gives names to sequences |
| **Genus** | A group of closely related bacteria |

## 6. Credits

- Course data: [gibbons-lab/isb_course_2024](https://github.com/gibbons-lab/isb_course_2024)
- QIIME 2 install file: [qiime2/distributions](https://github.com/qiime2/distributions)
- QIIME 2: [qiime2.org](https://qiime2.org)
- Viewer: [view.qiime2.org](https://view.qiime2.org)
- Help: [QIIME 2 Forum](https://forum.qiime2.org)

