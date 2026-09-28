# Agentic AI for Bacterial Whole-Genome Sequencing

To build a scientific workflow in which an **AI agent** retrieves, quality-controls, preprocesses, and evaluates bacterial **whole-genome sequencing (WGS)** data, running on **Jetstream2**.

The workflow will use established bioinformatics tools for sequencing QC and preprocessing, followed by a simple **machine-learning anomaly detection** step. The AI agent will orchestrate the workflow, inspect intermediate results, decide which permitted actions to take, and stop for human review when necessary.

**Status:** planning/development stage. Raw sequencing data will be retrieved from a public sequencing archive rather than stored directly in the repository.


## The data

The initial workflow will use small, publicly available **paired-end Illumina bacterial WGS datasets** from the NCBI's Sequence Read Archive (SRA).

The workshop dataset will be specified by its accession number rather than distributed through GitHub. During the workflow, the AI agent will use the **SRA Toolkit** to retrieve the sequence data and convert them into paired FASTQ files.

Example:

```text
Accession: ERR4567643
Organism: Escherichia coli
Sequencing type: Illumina paired-end WGS
```

The raw sequencing files will be stored locally on the Jetstream2 instance under:

```text
data/raw/
```

The original sequencing data will remain unchanged throughout the workflow.


## Agent policy

The AI agent will operate under a predefined policy that separates routine computational actions from decisions requiring human scientific judgment.

Examples of actions the agent may perform automatically include:

- retrieving the specified sequencing accession
- inspecting FASTQ files
- identifying paired reads
- running sequencing QC
- performing approved adapter or quality trimming
- generating summary statistics
- running the predefined machine-learning model
- generating reports

The agent will not be permitted to:

- delete raw sequencing data
- overwrite original FASTQ files
- exclude biological samples
- change the QC policy
- make consequential scientific decisions without human review

Conditions such as missing read pairs, corrupted FASTQ files, extreme read loss, or unexpected sample counts will trigger a **human checkpoint**.

## Planned workflow

1. **Receive sequencing accessions**
   - The user specifies the bacterial WGS dataset to analyze (ERR4567643, etc).

2. **Retrieve raw sequencing data**
   - The agent recognizes the accession and uses the SRA Toolkit to obtain paired-end FASTQ files.

3. **Validate the input**
   - Confirm expected files are present.
   - Check R1/R2 pairing.
   - Identify missing, empty, corrupted, or inconsistently named files.

4. **Run initial sequencing QC**
   - Calculate basic read statistics with SeqKit.
   - Run FastQC.
   - Aggregate QC information with MultiQC.

5. **Evaluate QC results**
   - The agent inspects QC outputs and compares them with the rules defined in `qc_policy.yaml`.

6. **Perform preprocessing when appropriate**
   - If permitted by the policy, run `fastp` for adapter and/or quality trimming.
   - Original FASTQ files remain unchanged.

7. **Run post-processing QC**
   - Repeat FastQC and read statistics.
   - Compare read counts, quality metrics, adapter content, and read retention before and after preprocessing.

8. **Construct a QC feature table**
   - Extract measurements such as:
     - total reads
     - GC content
     - mean read quality
     - adapter content
     - duplication
     - read retention

9. **Run machine-learning anomaly detection**
   - Use `scikit-learn` **Isolation Forest** to identify samples with unusual combinations of QC characteristics.

10. **Agent investigates flagged samples**
    - The machine-learning model identifies unusual samples.
    - The AI agent examines the associated QC measurements and reports which characteristics contributed to the concern.

11. **Human review**
    - A researcher decides whether anomalous or problematic samples should be retained, repeated, excluded, or investigated further.

12. **Generate a reproducible report**
    - Record:
      - commands executed
      - software versions
      - QC results
      - preprocessing actions
      - ML results
      - agent decisions
      - human checkpoints

## Role of the AI agent

Conceptually:

```text
Human defines goal + policy
          |
          v
      AI agent
          |
          +----> SRA Toolkit
          |
          +----> SeqKit
          |
          +----> FastQC / MultiQC
          |
          +----> fastp
          |
          +----> Python / scikit-learn
          |
          v
 Inspect intermediate results
          |
          v
 Decide next permitted action
          |
     +----+----+
     |         |
  continue   human review
```

The agentic workflow can modify its path according to intermediate results while remaining constrained by researcher-defined rules.

For example:

```text
Initial FastQC
      |
      +---- acceptable QC ----> skip unnecessary trimming
      |
      +---- adapter issue ----> run approved trimming
      |
      +---- corrupted FASTQ --> stop for human review
```

## Machine-learning component

The initial machine-learning demonstration will use an **Isolation Forest** for anomaly detection.


For example:

```text
sample       reads     GC     quality    adapter    retention
sample01     410000    50.8     35.1       0.8        99.1
sample02     398000    50.5     34.7       1.1        98.7
sample03     220000    62.4     27.1      18.5        61.2
```

The ML model might flag `sample03` as anomalous.

The AI agent would then investigate the QC results and report that the sample has unusual GC content, poorer quality, high adapter content, and substantial read loss.

## Jetstream2 environment

A prepared Jetstream2 image containing tools such as:

- SRA Toolkit
- SeqKit
- FastQC
- MultiQC
- fastp
- Python
- pandas
- scikit-learn
- PyYAML


## Possible downstream analyses

- Compare multiple preprocessing strategies
- Introduce intentionally problematic FASTQ datasets for agent troubleshooting
- Add bacterial genome assembly with SPAdes (only with few samples since assembly usually takes awhile)
- Evaluate assemblies using QUAST
- Annotate genomes with Bakta
- Detect antimicrobial resistance genes with AMRFinderPlus
- Compare multiple bacterial isolates
- Replace/compare Isolation Forest with other machine-learning approaches

## Data source

The workshop repository will contain the accession and retrieval instructions, but **will not store the raw FASTQ files themselves**.

