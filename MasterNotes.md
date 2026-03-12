# GUbioinformatics
## Goal for 3/12:
FIRST: dowload reads and trim using trimmomatic
>we have both downloaded ascension number SAMN08784163 into directories labeled fastqc_6 on our respective local directories
>this created a directory called SRR6996008
>we used fasterq-dump *.sra & gzip *.fastq to convert the files to fastqc
>we are making a directory within fastqc_6 labeled raw and moving the raw fastqc into this directory
>cp SRR6996008.sra_*.fastq.gz ../raw/
SECOND: Organize
THIRD: FastQC of raw
FOURTH: Trimmomatic 
FIFTH: FastQC of trimmed
