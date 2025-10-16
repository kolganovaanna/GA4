
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

*Note*: below, my steps will go in the order of my coding because to me it made more sense to first create README, scripts, results, and copy data before making a .gitignore. So, I basically will combine those steps and show my commands. 

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
git add scripts/* README.md
git commit -m "Part A done"
```
*Note* I had to use -f as the terminal told me because I don't want this tringalore.sh file to be ignored

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

sbatch scripts/trimgalore.sh "$R1" "$R2" results/fastqc
```
The outputs were:

```bash
-rw-rw----+ 1 kolganovaanna PAS2880 21M Sep  9 13:45 ../garrigos-data/fastq/ERR10802863_R1.fastq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 22M Sep  9 13:45 ../garrigos-data/fastq/ERR10802863_R2.fastq.gz

Submitted batch job 37821007
```

9. Monitor the job, and when it’s done, check that everything went well (if it didn’t, redo until you get it right). In your README.md, explain your monitoring and checking process. Then, remove all outputs (Slurm log files and TrimGalore output files) produced by this test-run.

I used the following command:

```bash
squeue -u $USER -l
```

The output was:

```Tue Oct 14 12:31:39 2025
             JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
          37821007       cpu trimgalo kolganov  RUNNING       0:07     30:00      1 p0215
          37820928       cpu ondemand kolganov  RUNNING      32:55   4:00:00      1 p0223
```

Once it was done running, the files appeared on the left side bar. Then, I proceeded to check 1) if the files are there using ls; 2) if the job has been successfully finished using tail ; 3) if the main FastQC output files are present using ls -lh

I used the following commands:

```bash
ls

 tail slurm-fastqc*.out
 tail slurm-fastqc*.err

 ls -lh results/fastqc
 ```

 The outputs were:

 ```bash
 data  README.md  results  scripts  slurm-fastqc-37821007.err  slurm-fastqc-37821007.out

# TrimGalore version:

                        Quality-/Adapter-/RRBS-/Speciality-Trimming
                                [powered by Cutadapt]
                                  version 0.6.10

                               Last update: 02 02 2023

# Successfully finished script trimgalore.sh
Tue Oct 14 12:31:55 PM EDT 2025

Approx 80% complete for ERR10802863_R2_val_2.fq.gz
Approx 85% complete for ERR10802863_R2_val_2.fq.gz
Approx 90% complete for ERR10802863_R2_val_2.fq.gz
Approx 95% complete for ERR10802863_R2_val_2.fq.gz
Deleting both intermediate output files ERR10802863_R1_trimmed.fq.gz and ERR10802863_R2_trimmed.fq.gz

====================================================================================================

INFO:    Using cached SIF image
INFO:    gocryptfs not found, will not be able to use gocryptfs

total 43M
-rw-rw----+ 1 kolganovaanna PAS2880 2.4K Oct 14 12:31 ERR10802863_R1.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 675K Oct 14 12:31 ERR10802863_R1_val_1_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 348K Oct 14 12:31 ERR10802863_R1_val_1_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 14 12:31 ERR10802863_R1_val_1.fq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 2.3K Oct 14 12:31 ERR10802863_R2.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 676K Oct 14 12:31 ERR10802863_R2_val_2_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 341K Oct 14 12:31 ERR10802863_R2_val_2_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  21M Oct 14 12:31 ERR10802863_R2_val_2.fq.gz
```
All of the outputs have indicated to me that the scipt was succesfully ran. For final check, I will use this command again but modified (just to try it with my username):

```bash
squeue -u kolganovaanna -l
```

*Note*: the cleanup wasn't done until question 10 was answered (because I needed to look at the file to make my conclusions). Additionally, my job was completed so fast that I only saw it in the "Running" status. 

To clean up, I used the following command:

```bash
rm -r results/fastqc slurm-fastqc*
```

I saw how the files on the left side bar disappeared. 

10. Illumina sequencing uses colors to distinguish between nucleotides as they are being added during the sequencing-by-synthesis process. However, newer Illumina machines (Nextseq, Novaseq) use a different color chemistry than older ones, and this newer chemistry suffers from an artefact which can produce strings of Gs with high quality scores that really should be Ns, especially at the end of reverse (R2) reads. In the FastQC outputs for the R2 file that you just produced with TrimGalore (recall that it runs FastQC after trimmming!), do you see any evidence for this problem? Explain.

less slurm-fastqc-37821007.out
/R2
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
