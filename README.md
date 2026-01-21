# Blobtools
Blobtools is a visualisation and quality control software suite used primarily to detect and remove contamination from genome assemblies.


## Downloading softwares using conda
```bash

conda create -n diamond
conda activate diamond
conda install -c bioconda diamond

conda create -n blobtk
conda activate blobtk
conda install -c bioconda blobtk

conda create -n btk
conda activate btk
conda install -c bioconda btk
```
## Downloading the nucleotide and protein database 


## Acquiring nucleotide database
```bash

wget https://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nt.gz
```

## Acquiring protein database
```bash

--wrap "wget -O reference_proteomes.tar.gz \
 ftp.ebi.ac.uk/pub/databases/uniprot/current_release/knowledgebase/reference_proteomes/$(curl \
     -vs ftp.ebi.ac.uk/pub/databases/uniprot/current_release/knowledgebase/reference_proteomes/ 2>&1 | \
     awk '/tar.gz/ {print $9}')"

cd $UNIPROT
tar xf reference_proteomes.tar.gz

touch reference_proteomes.fasta.gz
find . -mindepth 2 | grep "fasta.gz" | grep -v 'DNA' | grep -v 'additional' | xargs cat >> reference_proteomes.fasta.gz

printf "accession\taccession.version\ttaxid\tgi\n" > reference_proteomes.taxid_map
zcat */*/*.idmapping.gz | grep "NCBI_TaxID" | awk '{print $1 "\t" $1 "\t" $3 "\t" 0}' >> reference_proteomes.taxid_map

/path/to/diamond/bin/diamond makedb -p 16 --in reference_proteomes.fasta.gz --taxonmap reference_proteomes.taxid_map --taxonnodes $TAXDUMP/nodes.dmp --taxonnames $TAXDUMP/names.dmp -d reference_proteomes.dmnd
```

# Building database using Diamond

## Step 1:
```bash

/path/to/diamond/bin/diamond makedb \
  -p 30 \
  --in reference_proteomes_nodup.fasta.gz \
  --taxonmap reference_proteomes_nodup.taxid_map \
  --taxonnodes /path/to/nodes.dmp \
  --taxonnames /path/to/names.dmp \
  -d reference_proteomes.dmnd
```
## Step 2: Blast your genome to the database
```bash

/pfs/home/kulkarnilab/miniconda3/envs/diamond/bin/diamond blastx \
        --query /path/to/assembled.fasta file \
        --db /path/to/reference_proteomes.dmnd \
        --outfmt 6 qseqid staxids bitscore qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore \
        --sensitive \
        --max-target-seqs 1 \
        --evalue 1e-25 \
        --threads 30 \
        > /path/to/output_diamond.out
```
## Step 3:
```bash

BELL1=/path/to/your/raw.fastq file

ASSEMBLY=/path/to/assembled.fasta file

source activate
conda activate /path/to/blobtk


module load /path/to/minimap2/bin/minimap2
module load /path/to/samtools/bin/samtools

/path/to/minimap2/bin/minimap2 -ax map-ont -t 30 $ASSEMBLY $BELL1 | /path/to/samtools/bin/samtools sort -@ 20 -O BAM -o /path/to/output_minimapout.bam
```
## Step 4:
```bash

conda activate /path/to/busco

busco \
    -i /path/to/assembled.fasta file \
    -o ./output_arthropoda__odb12 \
    -l /path/to/lineages/arthropoda_odb12 \
    -m geno \
    -c 30 \
    -r --offline
```
## Step 5:
```bash

conda activate /pfs/home/kulkarnilab/miniconda3/envs/btk

#module load singularity
/pfs/home/kulkarnilab/miniconda3/envs/btk/bin/blobtools add --threads 30 \
--fasta /path/to/assembled.fasta file \
--hits /path/to/output_diamond.out \
--taxid (taxon_id_number) \
--taxrule bestsumorder \
--taxdump /path/to/taxdump/ \
--cov /path/to/output_minimapout.bam \
--busco /path/to/output_acari_odb12/run_acari_odb12/full_table.tsv \
--busco /path/to/output_arthropoda__odb12/run_arthropoda_odb12/full_table.tsv \
--create /path/to/output/directory/BlobDir
```

## Step 6:
```bash

conda activate /path/to/blobtk

/path/to/blobtk/bin/blobtk plot -d /path/to/output/directory/BlobDir -v blob -o ./blob-plot.svg
/path/to/blobtk/bin/blobtk plot -d /path/to/output/directory/BlobDir -v cumulative -o ./cumulative-plot.svg
/path/to/blobtk/bin/blobtk plot -d /path/to/output/directory/BlobDir -v legend -o ./legend-plot.svg
/path/to/blobtk/bin/blobtk plot -d /path/to/output/directory/BlobDir -v snail -o ./snail-plot.svg
```
