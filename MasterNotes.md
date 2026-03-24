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
- after trimming, the per sequence base quality falls in the green and yellow, whereas the per sequece base quality dipped into the red before trimming **(add more notes to this)**

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

file              format  type  num_seqs    sum_len  min_len  avg_len  max_len   Q1   Q2   Q3  sum_gap  N50  N50_num  Q20(%)  Q30(%)  AvgQual  GC(%)  sum_n

final.contigs.fa  FASTA   DNA     18,160  8,380,542      224    461.5  129,695  335  396  483        0  443      852       0       0        0  47.39      0

###### → What does it mean?

This is a FASTA file with are 18160 sequences with an average length of 461.5

## Workflow 3/19
### Virsorter:

#### FIRST: getting organized:
made new folder labeled virsorter and new folder labeled votus on the Bucket

`gsutil cp gs://gu-biology-dept- class/mmw162/Bioinformatics_Project/megahit/final.contigs.fa .`

downloaded final.contigs.fa to megahit folder

#### Install virsorter and databases
installed virsorter and downloaded databases

`$ module load mamba`

`$ mamba create -y -n vs2-env -c conda-forge -c bioconda virsorter`

`$ mamba activate vs2-env`

`$ rm -rf db`

`$ virsorter setup -d db -j 4 --conda-frontend conda`

#### Run virsorter with Slurm script 

`sbatch mmw162/scripts/virsorter_6`

`#!/bin/bash`

`#SBATCH --job-name=virsorter_6`

`#SBATCH --nodes=1`

`#SBATCH --cpus-per-task=8`

`#SBATCH --mem=20G`

`#SBATCH --time=03:00:00`    

`#SBATCH --mail-type=END,FAIL`

`#SBATCH --mail-user=mmw162@georgetown.edu`

`#SBATCH --output=/home/mmw162/Bioinformatics_Project/virsorter/job_%j.out`

`#SBATCH --error=/home/mmw162/Bioinformatics_Project/virsorter/job_%j.err`

`# ==== Load mamba (students: no need to change) ====`

`module load mamba`

`source $(mamba info --base)/etc/profile.d/conda.sh`

`# Activate the environment where you had VirSorter2 installed`

`mamba activate vs2-env`

`# ==== Set paths and filenames (students: edit this block!) ====`

`#set up directories`

`INDIR=/home/mmw162/Bioinformatics_Project/megahit/final.contigs.fa #directory where input will$`

`OUTROOT=/home/mmw162/Bioinformatics_Project/virsorter    #directory output will go` 

`mkdir -p "${OUTROOT}"                                    #new directory to be created for outp$`

`SAMPLE_ID=6                                              #just the basic sample name (sample2 $`

`INPUT="${INDIR}"                        #contig file name/location`

`OUTDIR="${OUTROOT}/vs2-${SAMPLE_ID}"                     #where you’ll find the outputs`

`mkdir -p "${OUTDIR}"`


`# ==== Run virsorter2 with >5kb cutoff and DNA virus categories first`

`echo "Running VirSorter2 on ${INPUT}"`

`virsorter run \`

 ` -w "${OUTDIR}" \`
 
 ` -i "${INPUT}" \`
 
 `--keep-original-seq \`
 
 ` --include-groups dsDNAphage,NCLDV,ssDNA \`
 
  `--min-length 5000`

`echo "Done."`

#### find results

in directory under virsorter: vs2-6 

final-viral-boundary.tsv 

final-viral-score.tsv      
` 
final-viral-combined.fa    

#### count number of contigs 
`grep -c ">" final-viral-combined.fa`

10

` seqkit seq -m 5000 final-viral-combined.fa | grep -c ">"
[WARN] you may switch on flag -g/--remove-gaps to remove spaces` 

10

#### make new file 
` seqkit seq -m 5000 final-viral-combined.fa > final-viral-combined_min5kb.fa`

#### copy file to bucket

`gsutil cp /home/mmw162/Bioinformatics_Project/virsorter/vs2-6/final-viral-combined.fa gs://gu-biology-dept-class/mmw162/Bioinformatics_Project/Virsorter`

`gsutil cp /home/mmw162/Bioinformatics_Project/virsorter/vs2-6/final-viral-combined_min5kb.fa gs://gu-biology-dept-class/mmw162/Bioinformatics_Project/Virsorter`

#### SECOND: Clustering

##### Install Clustering
Install vclust 
Install by creating an environment, installing virsorter 
`module load mamba`

`mamba create -n votu-env -c bioconda -c conda-forge vclust`

` mamba activate votu-env`

`vclust prefilter -i final-viral-combined_min5kb.fa -o fltr.txt`

`vclust align -i final-viral-combined_min5kb.fa -o ani.tsv --filter fltr.txt`


`vclust cluster -i ani.tsv -o clusters.tsv --ids ani.ids.tsv --metric ani --ani 0.95 --out-repr`

put votus in new file 
` seqkit grep -f votu_seeds.txt final-viral-combined_min5kb.fa > votus_final.fna` 

#### copy file to bucket

'gsutil cp /home/mmw162/Bioinformatics_Project/virsorter/vs2-6/votus_final.fa gs://gu-biology-dept-class/mmw162/Bioinformatics_Project/Votus'

## workflow 3/24

### download checkv database

`module load checkv`
`checkv download_database ./	`

### run checkv

- running checkv with a slurm script to check the quality of contigs
- set up checkv directory
- run checkv with slurm script:

`#!/bin/bash`

`#SBATCH --job-name=checkv`

`#SBATCH --output=/home/mmw162/Bioinformatics_Project/checkv-%j.out`

`#SBATCH --error=/home/mmw162/Bioinformatics_Project/checkv-%j.err`

`#SBATCH --time=03:00:00`

`#SBATCH --nodes=1`

`#SBATCH --ntasks=1`

`#SBATCH --cpus-per-task=16`

`#SBATCH --mem=16G`

`#SBATCH --mail-type=END,FAIL`

`#SBATCH --mail-user=mmw162@georgetown.edu`


`# ==== Load checkv program module (students: no need to change) ====`

`module load checkv`


`# ==== Set variables, paths, and filenames (students: edit this block!) ====`

`CHECKVDB="/home/mmw162/Bioinformatics_Project/checkv/checkv-db-v1.5"`

`SAMPLE_ID="vOTUs"`

`INPUT="/home/mmw162/Bioinformatics_Project/virsorter/vs2-6/votus_final.fna"`

`OUTDIR="/home/mmw162/Bioinformatics_Project/checkv/${SAMPLE_ID}"`

`mkdir -p "${OUTDIR}"`


`# ==== run checkv (students: no need to change. The second line is the command) ====`

`echo "Running CheckV on ${INPUT}"`

`checkv end_to_end "${INPUT}" "${OUTDIR}" -d "${CHECKVDB}" -t ${SLURM_CPUS_PER_TASK}`

`echo "Done."`

### checkv data 
- 2 complete contigs
- 1 high quality contig
- 7 low quality contigs

- longest contig: 129695

### bowtie2
- put votu files into bowtie2 files
- load bowtie2
  `module load bowtie2`
  `bowtie2-build votus_10kb_6samples.fna votu_index`
- write slurm script for bowtie2
- upload bowtie outputs to bucket
  `gcloud storage cp [file] gs://gu-biology-dept-class/ClassProject/bam

