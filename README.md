# Download SARS-COVI-2 Hifi Omicron samples from Pacbio website
**1. Making directory**
```bash
mkdir -p Hifi_Pacbio_Sarscovi2
```
**2. Download fastq.zip data from Pacbio's website**
```bash
wget  --outdir home/hp/Hifi_Pacbio_Sarscovi2 https://downloads.pacbcloud.com/public/dataset/HiFiViral/Jan_2022/samples.hifi_reads.fastq.zip  
```

**3. Unzip the downloaded fastq.zip file**
```bash
cd home/hp/Hifi_Pacbio_Sarscovi2
unzip  samples.hifi_reads.fastq.zip
```

**4. Create IDs.list file (contain fastq sample files' name after unzip)**
```bash
find /home/hp/Hifi_Pacbio_Sarscovi2 -name "*.hifi_reads.fastq" | sed 's|.*/||' > /home/hp/Hifi_Pacbio_Sarscovi2/IDs.list
chmod +r chmod +r IDs.list
```

# Quality check using multiqc tool
**1. Install multiqc**
```bash
pip install multiqc
```

**2. Run multiqc**
***2.1. create script file to run fastqc***
```bash
nano run_fastqc.sh
```

***2.2. run the script***
```bash
chmod +x run_fastqc.sh
./run_fastqc.sh
```
Read the full multiqc report name **multiqc_report.html**

# Denovo assembly with Canu 2.2
**1. Install canu 2.2**
Follow the instruction in the below github to install canu 2.2**
```bash
https://github.com/marbl/canu
```

**2. Combine all the fastq file**
```bash
cd home/hp/Hifi_Pacbio_Sarscovi2
cat *hifi_reads.fastq > merged_hifi_reads.fastq
```

**2. Run canu 2.2**
```bash
conda activate canu_2_2_env
canu -p hifi_assembly -d output_directory genomeSize=30k minReadLength=500 minOverlapLength=70 \
-pacbio-hifi merged_hifi_reads.fastq -maxThreads=8 corThreads=8
```
The denovo assembly results is in folder name **hifi_assembly**

# Assessment the de novo assembly results by QUAST
**1. Install QUAST tool**
Follow the instruction in the below github to install QUAST tool
```bash
https://github.com/ablab/quast
```

**2. Run QUAST**
```bash
quast hifi_assembly.contigs.fasta -o quast_output
```
Read the QUAST results in QUAST in folder **quast_output** file name **report.html**

# Polishing with Racon tool
**1.Install racon tool**
Follow the instruction in the below github to install racon
```bash
https://github.com/isovic/racon
```

**2. Run racon**
```bash
cd home/hp/Hifi_Pacbio_Sarscovi2/hifi_assembly
minimap2 -ax map-hifi hifi_assembly.contigs.fasta merged_hifi_reads.fastq > aligned_reads.sam
racon merged_hifi_reads.fastq aligned_reads.sam hifi_assembly.contigs.fasta > racon_output/Covid_19.contigs.polished.fasta
```
The polishing results is in **racon_output** folder, file named **Covid_19.contigs.polished.fasta**.

#   QUAST after polishing
```bash
cd home/hp/Hifi_Pacbio_Sarscovi2/hifi_assembly/racon_output
quast Covid_19.contigs.polished.fasta -o quast_after_polishing
```
QUAST results is in **quast_after_polishing** folder - file named **report.html**

# Scaffolding with ragoo tool
**1. Install ragoo**
```bash
conda create -n ragoo_env python=3.8
conda activate ragoo_env
```
```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```
```bash
conda install ragoo
```

**2. Generate paf file**
```bash
minimap2 -x sr -t 8 /home/hp/Hifi_Pacbio_Sarscovi2/assembly_output/racon_output/Covid_19.contigs.polished.fasta /home/hp/Hifi_Pacbio_Sarscovi2/assembly_output/racon_output/merged_hifi_reads.fastq > /home/hp/Hifi_Pacbio_Sarscovi2/assembly_output/racon_output/reads_against_ctg.paf 2> /home/hp/Hifi_Pacbio_Sarscovi2/assembly_output/racon_output/reads_against_ctg.paf.log
```

**3. Download reference genome**
```bash
wget http://hgdownload.soe.ucsc.edu/goldenPath/wuhCor1/bigZips/wuhCor1.fa.gz
```

**4. Run ragoo**
```bash
ragoo -R merged_hifi_reads.fastq -T sr -g 30000 Covid_19.contigs.polished.fasta wuhCor1.fa
```
Generated scaffold named **ragoo.fasta**
# Quast after scaffolding
```bash
quast ragoo.fasta -o quast_after_scaffolding
```

# Using nextclade website to analyse the scaffold**
**1. Using awk to filter scaffold with approriate size**
```bash
awk '/^>/{if (seq && length(seq) < 400000) {print header "\n" seq} header = $0; seq = ""} !/^>/{seq = seq $0} END{if (length(seq) < 400000) print header "\n" seq}' ragoo.fasta > filtered_scaffold.fasta
```

**2. upload to nextclade website to analys phylotree, variants**
upload the **filtered_scaffold.fasta** to the https://clades.nextstrain.org/ to see the clade and variants

# Genome annotation with Prokka
**1. Install Prokka tool**
Follow the below github to install prokka tool
```bash
https://github.com/vdejager/prokka
```
**2. Quality check the scaffold before annotation**
Verify the amount and length of the retained scaffold
```bash
grep -A1 '^>' filtered_scaffold.fasta | grep -v '^--' | awk 'NR % 2 == 0 {print length($0)}' | sort -n
```
Verify the N rate
```bash
grep -A1 '^>' filtered_scaffold.fasta | grep -v '^--' | awk 'NR % 2 == 0 {n_count += gsub(/N/, ""); total_length += length($0)} END {print "N count:", n_count, "Total bases:", total_length, "N ratio:", n_count/total_length}'
```
Clean the N from the scaffold
```bash
awk '/^>/{print; header=$0; seq=""} !/^>/{gsub(/N/, "", $0); seq=seq $0} END{if (seq) print header "\n" seq}' filtered_scaffold.fasta > cleaned_scaffold.fasta
```

**3. Genome annotation with prokka**
```bash
prokka --outdir prokka_cleaned_scaffold --prefix my_genome --cpus 8 --kingdom Viruses cleaned_scaffold.fasta
```
Read the annotation results in **prokka_cleaned_scaffold** folder, file named **my_genome.tsv**
Visualize the result file name my_genome.gff by IGV (Integrative Genomics Viewer)




