# Blobtools
Blobtools is a visualisation and quality control software suite used primarily to detect and remove contamination from genome assemblies.

## Downloading the nucleotide and protein database 

## Downloading software using conda
```bash
conda create -n diamond
conda activate diamond
conda install -c bioconda diamond
```

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

/pfs/home/kulkarnilab/miniconda3/envs/diamond/bin/diamond makedb -p 16 --in reference_proteomes.fasta.gz --taxonmap reference_proteomes.taxid_map --taxonnodes $TAXDUMP/nodes.dmp --taxonnames $TAXDUMP/names.dmp -d reference_proteomes.dmnd
```
## Downloading software using conda

```bash
conda create -n hifiasm
conda activate hifiasm
conda install -c bioconda hifiasm
