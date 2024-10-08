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




