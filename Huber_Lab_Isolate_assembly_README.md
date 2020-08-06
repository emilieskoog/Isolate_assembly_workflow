# README

####   


SO you just got back a bunch of raw isolate sequences from the sequencing center. Here's a simple workflow of how I assembled and quality-checked these sequences.

Onwards! :rocket: 



## :memo: Trimmomatic

First, you want to trim the adapters from your raw sequences. Sometimes the sequencing center will do this for you. Sometimes you don't know what adapters you used because you lost the email or the information was lost at some point. But THAT'S (kind of) OKAY! Trimmomatic is your friend.

### Step 1: Create an environment for trimmomatic

```
conda create --name trimmomatic

conda activate trimmomatic

conda install -c bioconda trimmomatic
```

### Step 2: Add adapter file with all adapters 

If you are uncertain of what adapters were used, here is a list of them that trimmomatic can go through and test each one. If you are unsure of whether or not adapters were already removed, performing this step would not hurt regardless.

1. Find path for trimmomatic using:
```
conda info --envs 
```

2. Follow the outputted path that will lead you to the trimmomatic directory

"cd" into this trimmomatic directory and create a folder named 'adapters'
```
mkdir adapters
```

3. Create a file with list of all adapters: all_adapters.fa
```
nano all_adapters.fa
```
4. Copy and past the following into this newly-created all_adapters.fa file

```
>PrefixNX/1
AGATGTGTATAAGAGACAG
>PrefixNX/2
AGATGTGTATAAGAGACAG
>Trans1
TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG
>Trans1_rc
CTGTCTCTTATACACATCTGACGCTGCCGACGA
>Trans2
GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG
>Trans2_rc
CTGTCTCTTATACACATCTCCGAGCCCACGAGAC>PrefixPE/1
AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
>PrefixPE/2
CAAGCAGAAGACGGCATACGAGATCGGTCTCGGCATTCCTGCTGAACCGCTCTTCCGATCT
>PCR_Primer1
AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
>PCR_Primer1_rc
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTAGATCTCGGTGGTCGCCGTATCATT
>PCR_Primer2
CAAGCAGAAGACGGCATACGAGATCGGTCTCGGCATTCCTGCTGAACCGCTCTTCCGATCT
>PCR_Primer2_rc
AGATCGGAAGAGCGGTTCAGCAGGAATGCCGAGACCGATCTCGTATGCCGTCTTCTGCTTG
>FlowCell1
TTTTTTTTTTAATGATACGGCGACCACCGAGATCTACAC
>FlowCell2
TTTTTTTTTTCAAGCAGAAGACGGCATACGA
>TruSeq2_SE
AGATCGGAAGAGCTCGTATGCCGTCTTCTGCTTG
>TruSeq2_PE_f
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT
>TruSeq2_PE_r
AGATCGGAAGAGCGGTTCAGCAGGAATGCCGAG
>PrefixPE/1
TACACTCTTTCCCTACACGACGCTCTTCCGATCT
>PrefixPE/2
GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT
>PE1
TACACTCTTTCCCTACACGACGCTCTTCCGATCT
>PE1_rc
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTA
>PE2
GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT
>PE2_rc
AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC>PrefixPE/1
TACACTCTTTCCCTACACGACGCTCTTCCGATCT
>PrefixPE/2
GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT>TruSeq3_IndexedAdapter
AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC
>TruSeq3_UniversalAdapter
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTA
```
### Step 3: Run trimmomatic! 

Create an executable file. Let's name it trim.sh 

```
nano trim.sh
```
Write your command within this file. See my example below.

```
#!/bin/bash
#SBATCH -p sched_mit_g4nier
#SBATCH -N 1
#SBATCH -n 20
#SBATCH --exclusive
#SBATCH --mem=125G                      
#SBATCH --time=7-00:00:00
#SBATCH -J trim
#SBATCH --output=trim_output
#SBATCH --error=trim_error

source activate trimmomatic

trimmomatic PE S1/MBA17_S1_R1_001.fastq.gz S1/MBA17_S1_R2_001.fastq.gz S1/paired_output_MBA17_S1_R1_001.fastq.gz S1/unpaired_output_MBA17_S1_R1_001.fastq.gz S1/paired_output_MBA17_S1_R2_001.fastq.gz S1/unpaired_output_MBA17_S1_R2_001.fastq.gz ILLUMINACLIP:/home/skoog/anaconda3/envs/trimmomatic/adapters/all_adapters.fa:2:40:15 CROP:140 LEADING:10 TRAILING:10 SLIDINGWINDOW:25:10 MINLEN:50
```
> Note: Make sure to change your path to reflect where your all_adapters.fa file is located.

 ## :memo: Quality check
 
 Now you want to check the quality of these trimmed reads. I used FastQC.
 
### Step 1: Create a FastQC environment 

```
conda create --name fastqc

conda activate fastqc

conda install -c bioconda fastqc
```
### Step 2: Run FastQC! 


Go into folder with files for fastqc-action to be done to them and create a shell file. Again:
```
nano fastqc.sh
```
Add your commands to this file. My example below:
```
#!/bin/bash
#SBATCH -p sched_mit_g4nier
#SBATCH -N 1
#SBATCH -n 20
#SBATCH --exclusive
#SBATCH --mem=125G                      
#SBATCH --time=7-00:00:00
#SBATCH -J fqc1
#SBATCH --output=fqc1_output
#SBATCH --error=fqc1_error

source activate fastqc

fastqc paired_output_MBA17_S1_R1_001.fastq.gz paired_output_MBA17_S1_R2_001.fastq.gz unpaired_output_MBA17_S1_R1_001.fastq.gz unpaired_output_MBA17_S1_R2_001.fastq.gz
```

Run fastqc:
```
sbatch fastqc.sh
```

This will result in multiple .html files that you can open in a web browser and assess. Make sure to adjust parameters to get higher quality reads before continuing on to assembly.

 ## :memo: Assemble

Once I am satisfied with the quality of my reads, I assembled these isolates using SPAdes.

### Step 1: Create a SPAdes environment 


```
conda create --name spades

conda activate spades

conda install -c bioconda spades
```
 ### Step 2: Run SPAdes
 
 I created the script by executing
 ```
 nano spades.sh
 ```
 and writing the following:
 ```
 #!/bin/bash
#SBATCH -p sched_mit_g4nier
#SBATCH -N 1
#SBATCH -n 20
#SBATCH --exclusive
#SBATCH --mem=125G                      
#SBATCH --time=7-00:00:00
#SBATCH -J spades1
#SBATCH --output=spades1_output
#SBATCH --error=spades1_error

source activate spades

spades.py -1 paired_output_MBA17_S1_R1_001.fastq.gz -2 paired_output_MBA17_S1_R2_001.fastq.gz \
          -o spades_default_assembly --only-assembler --isolate

```
 To execute the script:
 ```
 sbatch spades.sh
 ```
 
 Assembled contigs will be found in the "spades_default_assembly" directory in the "contigs.fasta" file. Assembled scaffolds are in the same directory in the "scaffolds.fasta" file. 
 
  ## :memo: Quality assess assembled genomes 
  Next, you want to assess the quality of these assembled genomes. I use CheckM to do so. CheckM gives completeness and contamination information (as well as a whole lot of additional important information) so you can analyze the quality of your assembly before going on to do more data analysis using these assembled genomes. 
  
  ### Step 1: Create a CheckM environment 
  
  ```
  conda create --name checkm
  
  conda activate checkm
  
  conda install -c bioconda checkm-genome
  ```
  
  ### Step 2: Run CheckM  

First, you want to create a directory where you would like the CheckM output to be stored. I named my directory checkm_output. To do so, navigate to whatever directory you would like your output to be in and do the following:

```
mkdir checkm_output
```
For me, the path to this directory is /home/skoog/work/MBA/checkm_output .

Then, I moved all my assembled contigs to one directory named "assembled_contigs". (My path to this folder is now /home/skoog/work/MBA/assembled_contigs.)

Then create an execuatble file with the following:

```
#!/bin/bash
#SBATCH -p sched_mit_g4nier
#SBATCH -N 1
#SBATCH -n 20
#SBATCH --exclusive
#SBATCH --mem=125G                      
#SBATCH --time=7-00:00:00
#SBATCH -J checkm
#SBATCH --output=checkm_out
#SBATCH --error=checkm_error

source activate checkm

checkm lineage_wf -t 8 -x fasta /home/skoog/work/MBA/assembled_contigs /home/skoog/work/MBA/checkm_output
```
 To create this file, I executed 
 
 ```
 nano checkm.sh
 ```
 and pasted the above script.
 > Note: If you are directing the script to output a file as it runs the script (in my case the "checkm_out" file as noted by the 
 > ```
 > "#SBATCH --output=checkm_out" 
 > ```
 > of my script), make sure it does not have the same name as your output directory (in my case "checkm_output"). Otherwise, your script will not run. 

 
 To run the script:
 
 ```
 sbatch checkm.sh
 ```
 
 All of the output files will be found in our output folder, in my case my checkm_output directory. The file with completeness and contamination scores is located in the checkm_output/storage directory in the bin_stats_ext.tsv file.
 
 These are simple scripts run individually. You can create one larger conda environment that runs all of these programs together and make one larger script, but out of fear of compatability/dependency issues and conflicts, I separated them. A snakefile may also be a good idea. Again, #KeepingItSimple.