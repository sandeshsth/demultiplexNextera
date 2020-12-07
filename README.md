#### 1. Demultiplexing skim-sequencing (Nextera) data into individual samples

USAGE:

```
demultiplexNextera.pl file.R1.fastq file.R2.fastq i1.fastq i2.fastq barcode.txt
```

Files:
1. Read 1 fastq file: `file.R1.fastq`
2. Read 2 fastq file: `file.R2.fastq` 
3. i7 barcode fastq file: `i1.fastq`
4. i5 barcode fastq file: `i2.fastq` 
5. Barcode file: `barcode.txt`

Barcode file (tab separated without header):

```
Sample1 TTCCTCCT_AAGACTGG
Sample2 AACCACTC_AAGACTGG
```

#### 2. Variant Calling Pipeline

Two parents aligned to the reference genome:
```
hisat2-2.1.0/hisat2 -p 10 -x 170831_Landmark_pseudomolecules_v1 -1 Landmark.R1.fq -2 Landmark.R2.fq --no-spliced-alignment --no-unal -S Landmark.sam
hisat2-2.1.0/hisat2 -p 10 -x 170831_Landmark_pseudomolecules_v1 -1 Stanley.R1.fq -2 Stanley.R2.fq --no-spliced-alignment --no-unal -S Stanley.sam
```

Retriving concordant unique reads:
```
cat <(samtools view -H Landmark.sam) <(awk '/YT:Z:CP/ && /NH:i:1/' Landmark.sam) | samtools sort -o Landmark.s.bam
samtools index -c Landmark.s.bam
cat <(samtools view -H Stanley.sam) <(awk '/YT:Z:CP/ && /NH:i:1/' Stanley.sam) | samtools sort -o Stanley.s.bam
samtools index -c Stanley.s.bam
```
Variant calling:
```
bcftools1.10.2/bin/bcftools mpileup --annotate AD,DP,INFO/AD --skip-indels -f 170831_Landmark_pseudomolecules_v1.fasta -b bamFilesList.txt -B | bcftools1.10.2/bin/bcftools call -m --variants-only  --skip-variants indels --output-type v -o LandmarkStanley.vcf --group-samples -
```

