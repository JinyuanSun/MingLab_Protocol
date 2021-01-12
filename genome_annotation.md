# genome annotation without RNA-seq
## Install of software *(Did not pass tests!!!)*
**Maker2**  
This is the best way to install maker sotfware and all others will be needed(SNAP, RepeatMasker...)  
**Warning: You should build the Maker2 from source!!!!!**  
``` bash
conda create --name MAKER python=2.7
conda activate MAKER
conda install -c bioconda/label/cf201901 maker #This will take minutes to hours depend on your web connection.
```
**BUSCO**
Because of lacking species-sepcific RNA data, the conserved proteins in higher taxonomy level will be used as evdience.
``` bash
conda create -n busco -c bioconda -c conda-forge busco=4.1.4
```

## Workflow
The overall directory structure:
```
dir/
    genome.fasta
    protein.fasta
    - maker_rnd1/
      - genome.maker.output/
    - snap_rnd1/
      - training/
      - prediction/
```
Now you shoud in the directory with the `genome.fasta` and `protein.fasta` file.  
**Maker2**
``` bash
mkdir maker_rnd1
cd maker_rnd1
cp ../genome.fasta ./
cp ../protein.fasta ./
maker -CTL
```
Edit the `maker_opts.ctl` file:
```
genome=genome.fasta #the fasta file of genome
protein=protein.fasta #the fasta file of homologous or conserved proteins
protein2genome=1 #set this option to 1
```
Run maker
``` bash
nohup maker > maker.out &
```
After the maker finished:
``` bash
cd genome.maker.output/
gff3_merge -s -d genome_master_datastore_index.log > genome_rnd1.busco.maker.gff
fasta_merge -d genome_master_datastore_index.log
```
**SNAP**
``` bash
cd ../../
mkdir snap_rnd1
cd snap_rnd1
cp ../genome.fasta ./
mkdir training
cp genome.fasta training
mkdir prediction
cp genome.fasta prediction
cd training
maker2zff -n -d ../../maker_rnd1/genome.maker.output/genome_master_datastore_index.log
fathom genome.ann genome.dna -gene-stats 
fathom genome.ann genome.dna -validate
fathom genome.ann genome.dna -categorize 1000
fathom uni.ann uni.dna -export 1000 -plus
mkdir params
cd params
forge ../export.ann ../export.dna
cd ..
hmm-assembler.pl genome.fasta params > genome.hmm
cp genome.hmm ../prediction
cd ../prediction
nohup snap -aa genome.snap.rnd1.protein.fasta -tx genome.snap.rnd1.transcripts.fasta -name snap Am_genome.hmm genome.fasta -gff > genome.snap.rnd1.gff &
```
