## Using YaHS (Yet-another-Hi-C-scaffolder).
YaHS (yet another Hi-C scaffolding tool, version YaHS-1.1, (Zhou et al., 2023) can be used to scaffold a genome to Hi-C data with contig and scaffolding error correction. You'll need to know how your Hi-C data was generated; for example, since, in this case, the Hi-C data was generated using the DpnII restriction enzyme, the option -e GATC was set.

First, we need to pre-process our reads using Samtools, Samblaster and bwa. FOr bwa, version 0.7.17 or higher is required for the -5 option for Hi-C data. PhaseGenomics has a useful explanation and similar pipeline here: https://phasegenomics.github.io/2019/09/19/hic-alignment-and-qc.html

### Hi-C Read Mapping and Preprocessing
```bash
# SETUP: Define paths to input read files and assembly
READ1=/path/to/r1/combined_r1.fq.gz
READ2=/path/to/r2/combined_r2.fq.gz
ASSEMBLY=/path/to/assemby.fasta

# Dependencies: Define paths to samtools and samblaster
samtools=/path/to/samtools-v.1.17
samblaster=/path/to/samblaster-v.0.1.26
bwa=/path/to/bwa-v.0.7.17

# Step 1: Index the assembly using BWA
$bwa index $ASSEMBLY

# Step 2: Align Hi-C reads to the assembly using BWA MEM. 5SP
$bwa mem -5SP -t 64 $ASSEMBLY $READ1 $READ2 -o aligned_hic.sam

# Step 3: Mark duplicates using samblaster
$samblaster -i aligned_hic.sam -o marked_duplicates.sam

# Step 4: Convert SAM to BAM, filtering out secondary alignments and unmapped reads using samtools
$samtools view -S -b -h -@ 64 -F 2316 marked_duplicates.sam > filtered.bam

# Step 5: Sort the BAM file by coordinate using samtools
$samtools sort -@ 64 filtered.bam -o sorted_coordinate.bam

# Step 6: Sort the BAM file by read names (required for downstream Hi-C analysis)
$samtools sort -@ 64 -n sorted_coordinate.bam -o sorted_name.bam
````

### Running Hi-C scaffolding with YaHS.

```bash
# Note: The input BAM file must be sorted by coordinate.
ASSEMBLY=/path/to/assemby.fasta
SORTED_BAM=/path/to/sorted_name.bam

# YAHS execution
yahs=/path/to/yahs
$yahs --no-mem-check -e GATC -q 20 --read-length 150 $reference_genome $bam
````

### Hi-C QC checks
You can check the quality of your Hi-C library using this script from PhaseGenomics: https://github.com/phasegenomics/hic_qc
The most informative Hi-C reads are the ones that are long-distance contacts, or contacts between contigs of an assembly. This tool quantifies such contacts and makes plots of contact distance distributions. The most successful Hi-C libraries have many long-distance and among-contig contacts.
