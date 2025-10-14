
# Graded Assignment #4

## General information

- Author: Anna kolganova
- Date: 2025-10-19
- Environment: Pitzer cluster at OSC via VS Code
- Working dir: `/fs/ess/PAS2880/users/kolganovaanna/GA4`

## Assignment background

This assignemnt targets practicing with slurm batch jobs

## Protocol

**PART A**

1. Start a VS Code session at OSC in the folder /fs/ess/PAS2880/users/$USER. Create a new dir for this assignment, /fs/ess/PAS2880/users/$USER/GA4, and switch to that folder in VS Code using the “Open Folder” option.

I used the following commands: 

```bash
mkdir GA4/
pwd
```

The output was:

```bash
/fs/ess/PAS2880/users/kolganovaanna/GA4
```

*Note*: below, my steps will go in the order of my coding because to me it made more sense to first create README, scripts, results, and copy data before making a .gitignore. So, I basically will combined those steps and show my commands. 

2. Initialize a Git repository. Commit to the repo throughout the assignment as you see fit, but at least once for each “Part” of this assignment. Use and commit a .gitignore file as appropriate.
3. Inside dir GA4, create a README.md file and open it in the VS Code editor. Use this file throughout the assigment to add your answers.
4. Inside dir GA4, also create dirs scripts and results. 


I used the following commands:

```bash
#!/bin/bash

touch README.md

mkdir scripts results

cp -rv ../garrigos-data/fastq data

git init
git add README.md
git commit -m "Committing to README to set up"
echo "results/" > .gitignore
echo "data/" >> .gitignore
echo "scripts/" >> .gitignore
git add .gitignore
git commit -m "Adding a Gitignore file"
```

5. As a starting script, store the code below in a shell script scripts/trimgalore.sh. This is a script to run TrimGalore, which should be very similar to your final script from last week’s exercises.

I used the following command:

```bash
cd scripts/
touch trimgalore.sh
```

I then copied the provided code into the trimgalore.sh file I jsut created. Then, I committed to the trimgalore.sh 

```bash
git add -f scripts/* README.md
git commit -m "Part A done"
```


**PART B**

6. Add Sbatch options to the top of the TrimGalore shell script to specify:

- The account/project you want to use
- The number of cores you want to reserve: use 8
- The amount of time you want to reserve: use 30 minutes
- The desired file name of Slurm log
- That Slurm should email you upon job failure
- Optional: you can try other options you’d like to test

For optional, I chose to do --error

```bash
#SBATCH --account=PAS2880
#SBATCH --cpus-per-task=8
#SBATCH --time=00:30:00
#SBATCH --output=slurm-fastqc-%j.out
#SBATCH --mail-type=FAIL
#SBATCH --error=slurm-fastqc-%j.err
```

I put all of that into the trimgalore.sh file 


7. Find the option that tells TrimGalore how many cores it can use and add the relevant line(s) from the TrimGalore help info to your README.md. In the script, change the trim_galore command accordingly to use the available number of cores.

The option I found is --cores:

```bash
apptainer exec oras://community.wave.seqera.io/library/trim-galore:0.6.10--bc38c9238980c80e \
  trim_galore --help
```

The relevant output was:

```bash
-j/--cores INT          Number of cores to be used for trimming [default: 1].
```

I changed the trim_galore command in the script. My logic was that the number of cores first needs to be given as a variable. Then, this becomes part of the report: 

```bash
echo "# Cores to use: $SLURM_CPUS_PER_TASK"

apptainer exec "$TRIMGALORE_CONTAINER" \
    trim_galore \
    --paired \
    --fastqc \
    --cores "$SLURM_CPUS_PER_TASK" \
    --output_dir "$outdir" \
    "$R1" \
    "$R2"
```
8. To test the script and batch job submission, submit the script as a batch job only for sample ERR10802863.

I used the following commands:

```bash
R1=../garrigos-data/fastq/ERR10802863_R1.fastq.gz
R2=../garrigos-data/fastq/ERR10802863_R2.fastq.gz

For check: ls -lh "$R1" "$R2"

sbatch scripts/trimgalore.sh "$R1" "$R2" results/trimgalore
```
The outputs were:

```bash
-rw-rw----+ 1 kolganovaanna PAS2880 21M Sep  9 13:45 ../garrigos-data/fastq/ERR10802863_R1.fastq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 22M Sep  9 13:45 ../garrigos-data/fastq/ERR10802863_R2.fastq.gz

Submitted batch job 37820912
```
squeue -u $USER -l



apptainer exec oras://community.wave.seqera.io/library/trim-galore:0.6.10--bc38c9238980c80e \
  trim_galore data/ERR10802863.fastq.gz
sbatch scripts/trimgalore.sh

example_file=data/ERR10802863_R1.fastq.gz
sbatch scripts/trimgalore.sh "$example_file" results/fastqc

squeue -u $USER -l
git init
echo "results/" > .gitignore
echo "data/" >> .gitignore
echo "scripts/" >> .gitignore
git add .gitignore
git commit -m "Add a Gitignore file"

sbatch --account=PAS2880 scripts/trimgalore.sh

```bash
apptainer exec oras://community.wave.seqera.io/library/trim-galore:0.6.10--bc38c9238980c80e \
  trim_galore --help
```
