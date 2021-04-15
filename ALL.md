## AntiSMASH

### 0. Install

```bash
conda create -n antismash
conda activate antismash
conda install hmmer2 hmmer diamond fasttree prodigal blast muscle glimmerhmm
wget https://dl.secondarymetabolites.org/releases/5.1.1/antismash-5.1.1.tar.gz
tar -zxf antismash-5.1.1.tar.gz
pip install ./antismash-5.1.1
download-antismash-databases
```

## BUSCO

### 0. Setup

It is recommended to install busco through conda

```bash
conda create -n BUSCO -c bioconda -c conda-forge busco==5.0.0
```

After this, you still need to add some variables for Augustus, add the following lines to your `~/.bashrc`:

```bash
export PATH="/path/to/augustus/augustus-3.2.3/bin:$PATH"
export PATH="/path/to/augustus/augustus-3.2.3/scripts:$PATH"
export AUGUSTUS_CONFIG_PATH="/path/to/augustus/augustus-3.2.3/config/"
```

**Warning: ** `/path/to/augustus/` should be change to real path in your system!

### 1. Run busco

```bash
busco -i scaffolds.fasta -o busco_out -m genome -l ascomycota_odb10 -c 32 -f
```



# Pymol visualization

Black outline with no reflection

```bash
set ray_trace_mode,1
set spec_reflect, 0
set ray_shadows,0
```



## Deep Learning

### 0.Setup

To create a new conda environment and install cudatoolkit, cud, and ternsorflow-gpu.

```bash
conda create -n tfgpu
conda activate tfgpu
conda install cudatoolkit=10.1
conda install cudnn=7.6.5
conda install tensorflow-gpu=2.1
```

**Clean the GPU memory**

```bash
conda install numba
```

```python
#Clean the GPU memory
from numba import cuda 
device = cuda.get_current_device()
device.reset()
```



## ROSETTA Cartesian_ddg Protocol

## 0. Setup

```bash
vim ~/.bashrc
```

Then, go to the end of the file and insert:

```bash
export PATH=$PATH:/ndata/nprograms/opt/rosetta_bin_linux_2020.37.61417_bundle/main/source/bin
```

Svae and exit, then use `source ~/.basrch` to enable the change.

Also, we recommand to add ROSETTADB as a environmental variable. Add the following line to the end of `bashrc`:

```bash
export ROSETTADB=/ndata/nprograms/opt/rosetta_bin_linux_2020.37.61417_bundle/main/database/
```

## README.md of p_rosetta_cart.py

The `p_rosetta_cart.py` is designed to pipeline the relax and cartesian_ddg of rosetta to preform deep mutation scan (DMS) of

 a protein. 

### 1. Relax:

```bash
python3 p_rosetta_cart.py -c relax -s <your_pdb> -nt <number_of_threads>
```

### 2. After the relax:

The output with lowest energy should be selected for ∆∆G calculation, use the `setup` script to setup a subdirectory for cartesian_ddg.

```bash
./setup.sh
```

A new directory named `run_ddg` should be created with 20 subdirectories and a `todo_list.sh` file.

If script find ROSETTADB is not added to environmental variable, it output warings than subdirectories:

```bash
Add ROSETTADB to evironment variable first!
e.g.:
	export ROSETTADB=/path/to/your/rosetta_src_<version>_bundle/main/database
	source ~/.bashrc
Then try again!
```

You should follow the instructions to add ROSETTADB as environmental variable, then run:

```bash
python3 p_rosetta_cart.py -c cartesian_ddg -s <your_pdb> -nt <number_of_threads>
```

After checking the `mtfile` in subdirectories, you can run:

```bash
./todo_list.sh
```

### 3. Select pdbs 

Select mutations below cutoff (-3 kcal/mol in this protocol, change it as you like!):

```bash
python3 ddg_select.py -sn 20
```

**Tips:**

In order to change the cutoff, you need to edit the script `ddg_select.py` at the following lines:

```python
BelowCutOff_df = BelowCutOff(CompleteList_df,-3)

BelowCutOff_SortedByEnergy_df = BelowCutOff(CompleteList_SortedByEnergy_df,-3)

BestPerPositionBelowCutOff_SortedByEnergy_df = BelowCutOff(BestPerPosition_SortedByEnergy_df,-3)

BestPerPositionBelowCutOff_df = BelowCutOff(BestPerPosition_df,-3)
```

The output of ddg_select.py are:

```bash
MutationsEnergies_BelowCutOff.tab
MutationsEnergies_BelowCutOff_SortedByEnergy.tab
MutationsEnergies_BestPerPosition.tab
MutationsEnergies_BestPerPositionBelowCutOff.tab
MutationsEnergies_BestPerPositionBelowCutOff_SortedByEnergy.tab
MutationsEnergies_BestPerPosition_SortedByEnergy.tab
MutationsEnergies_CompleteList.tab
MutationsEnergies_CompleteList_SortedByEnergy.tab
```

