# GUbioinformatics
## Goal for 3/12:
### FIRST: dowload reads and trim using trimmomatic
- we have both downloaded ascension number SAMN08784163 into directories labeled fastqc_6 on our respective local directories
- this created a directory called SRR6996008
- we used `fasterq-dump *.sra` & `gzip *.fastq` to convert the files to fastqc
- we are making a directory within fastqc_6 labeled raw and moving the raw fastqc into this directory
- `cp SRR6996008.sra_*.fastq.gz ../raw/` #moved fastqc reads into raw
- `gsutil cp -r raw gs://gu-biology-dept-class/mmw162/mmw162` #files in directory raw were copied to the bucket
### SECOND: Organize
- `gsutil mv gs://gu-biology-dept-class/mmw162/mmw162/SRR6996008.sra_1.fastq.gz gs://gu-biology-dept-class/mmw162/Bioinformatics_Project/raw/`
- `gsutil mv gs://gu-biology-dept-class/mmw162/mmw162/SRR6996008.sra_2.fastq.gz gs://gu-biology-dept-class/mmw162/Bioinformatics_Project/raw/` #moved fastqc files to a directory called raw under gs://gu-biology-dept-class/mmw162/Bioinformatics_Project/raw/
- There is a folder under gu-biology-dept-class/mmw162/ labeled Bioinformatics_Project with a subfolder raw
### THIRD: FastQC of raw
- ran FastQC on raw data, got output files in mam840/fastqc_6/fastqc_out
- `fastqc -o fastqc_out raw/SRR6996008.sr
a_1.fastq.gz raw/SRR6996008.sra_2.fastq.gz`
- downloaded HTML files from HPC onto local system
### FOURTH: Trimmomatic 
- ran trimmomatic with paramaters as stated:
  - minimum length of 50 base pairs, and sliding window of four with average score of 20
    `#!/bin/bash
#SBATCH --job-name="sample6.A"
#SBATCH --output="%x.o%j"
#SBATCH --mail-type=END,FAIL --mail-user=mam840@georgetown.edu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=03:00:00
#SBATCH --mem=10G

# load trimmomatic module ("aliases" needed for GU HPC setup here)
shopt -s expand_aliases
module load trimmomatic

# define paths and variables
R1=/home/mam840/fastqc_6/raw/SRR6996008.sra_1.fastq.gz
R2=/home/mam840/fastqc_6/raw/SRR6996008.sra_2.fastq.gz

adapters=/home/mam840/HW4_input_files/TruSeq3-PE.fa
OUTDIR=/home/mam840/fastqc_6/cleanedreads
mkdir -p $OUTDIR

trimmomatic PE -threads $SLURM_CPUS_PER_TASK \
        $R1 $R2 \
        $OUTDIR/SRR6996008.sra_1.paired.fq.gz   $OUTDIR/SRR6996008.sra_1.unpaired.fq.gz \
        $OUTDIR/SRR6996008.sra_2.paired.fq.gz   $OUTDIR/SRR6996008.sra_2.unpaired.fq.gz \
        ILLUMINACLIP:/home/mam840/HW4_input_files/TruSeq3-PE.fa:2:30:10 \
        SLIDINGWINDOW:4:20 MINLEN:50`
### FIFTH: FastQC of trimmed
