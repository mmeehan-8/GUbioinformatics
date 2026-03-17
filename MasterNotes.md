# GUbioinformatics Project: Molly & Maddie
## Workflow 3/12:
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

#### ran trimmomatic with paramaters as stated:
- minimum length of 50 base pairs, and sliding window of four with average score of 20
#### trimmomatic script 
` #!/bin/bash`

 `#SBATCH --job-name="sample6.A"`
 
` #SBATCH --output="%x.o%j"`

 `#SBATCH --mail-type=END,FAIL --mail-user=mam840@georgetown.edu`
 
` #SBATCH --nodes=1`

 `#SBATCH --ntasks=1`
 
 `#SBATCH --cpus-per-task=4`
 
 `#SBATCH --time=03:00:00`
 
 `#SBATCH --mem=10G`

#### load trimmomatic module ("aliases" needed for GU HPC setup here)

`shopt -s expand_aliases`

`module load trimmomatic`

#### define paths and variables 
`R1=/home/mam840/fastqc_6/raw/SRR6996008.sra_1.fastq.gz`

` R2=/home/mam840/fastqc_6/raw/SRR6996008.sra_2.fastq.gz`

`adapters=/home/mam840/HW4_input_files/TruSeq3-PE.fa`

#### define output directory (on Molly's local computer)

`OUTDIR=/home/mam840/fastqc_6/cleanedreads`

`mkdir -p $OUTDIR`

`trimmomatic PE -threads $SLURM_CPUS_PER_TASK \`
   `$R1 $R2 \`
   
  `$OUTDIR/SRR6996008.sra_1.paired.fq.gz   $OUTDIR/SRR6996008.sra_1.unpaired.fq.gz \ `
  
  `$OUTDIR/SRR6996008.sra_2.paired.fq.gz   $OUTDIR/SRR6996008.sra_2.unpaired.fq.gz \`
  
 ` ILLUMINACLIP:/home/mam840/HW4_input_files/TruSeq3-PE.fa:2:30:10 \`
 ` SLIDINGWINDOW:4:20 MINLEN:50` 
 
#### ran trimmomatic:  

`sbatch slurm/sample6.trim1.sbatch.save`


### FIFTH: FastQC of trimmed
- ran FastQC on trimmed data reads, both forward and reverse paired reads

`fastqc -o fastqc_out cleanedreads/SRR6996008.sra_1.paired.fq.gz cleanedreads/SRR6996008.sra_2.paired.fq.gz`

- the html files were downloaded onto Molly's local system

`gcloud compute scp mam840@m12-controller:/home/mam840/fastqc_6/fastqc_out/SRR6996008.sra_2.paired_fastqc
.html . `
- after trimming, the per sequence base quality falls in the green and yellow, whereas the per sequece base quality dipped into the red before trimming

## Workflow 3/17:
### Running Megahit (Steps 1-3):

##### slurm script:
`#!/bin/bash`

`#SBATCH --job-name=megahit_SRR6996008.sra`

`#SBATCH --nodes=1`

`#SBATCH --cpus-per-task=8`

`#SBATCH --mem=32G`

`#SBATCH --time=03:00:00`

`#SBATCH --output=/home/mam840/fastqc_6/logs/megahit.1.test.%j.out`

`#SBATCH --error=/home/mam840/fastqc_6/logs/megahit.1.test.%j.out`

`#SBATCH --mail-type=END,FAIL --mail-user=mam840@georgetown.edu`

`#note %j = job ID`

`# ==== Load mamba/conda module (students: no need to change) ====`

`module load mamba` 

`source $(mamba info --base)/etc/profile.d/conda.sh`


`# Activate the environment where you had MEGAHIT installed`

`conda activate megahit-env`

`# ==== Set paths and filenames (students: edit this block!) ====`

`# Directory where the cleaned reads live`

`READDIR=/home/mam840/fastqc_6/cleanedreads/`

`# Input read files (paired-end)`

`READ1=${READDIR}/SRR6996008.sra_1.paired.fq.gz`

`READ2=${READDIR}/SRR6996008.sra_2.paired.fq.gz`

`# Output directory (give it a name, it will be created by MEGAHIT)`

`OUTDIR=/home/mam840/fastqc_6/megahit/megahit_out_C`

`# ==== Run MEGAHIT ====`

`megahit \`

  `-1 ${READ1} \`
  
  `-2 ${READ2} \`
  
  `-t ${SLURM_CPUS_PER_TASK} \`
  
 ` -o ${OUTDIR}`

 `echo "Done. Contigs should be in: ${OUTDIR}/final.contigs.fa"`
 
`#### called slurm script:`

` sbatch slurm/megahit.1.test`

### Find your Results (Step 4)
#### How many contigs were assembled?
`grep -c ">" final.contigs.fa`

18160 contigs were assembled

`head final.contigs.fa`

The contigs look normal

### Checking Quality with SeqKit (Step 5)

 ` module load mamba/` 
 ###### did this in megahit_out_C directory

` $ mamba activate megahit-env`      

` $ mamba install -c bioconda seqkit` 

 `$ seqkit stats -a final.contigs.fa`

###### → Write/paste into your notes! 


→ What does it mean?



