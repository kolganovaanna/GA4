
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
git add .gitignore
git commit -m "Adding a Gitignore file"
```

5. As a starting script, store the code below in a shell script scripts/trimgalore.sh. This is a script to run TrimGalore, which should be very similar to your final script from last week’s exercises.

I used the following command:

```bash
cd scripts/
touch trimgalore.sh
```

I then copied the provided code into the trimgalore.sh file I just created. Then, I committed to the trimgalore.sh 

```bash
git add scripts/trimgalore.sh README.md
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

I changed the trim_galore command in the script. My logic was that the number of cores first needs to be given as a variable. Then, this becomes part of the report. Since I assigned 8 to my "cpus-per-task", I am just going to use this as an assigned value for --cores. Here I am allowed to use more than 1 because I am submitting a batch job: 

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

Submitted batch job 37845773
```

9. Monitor the job, and when it’s done, check that everything went well (if it didn’t, redo until you get it right). In your README.md, explain your monitoring and checking process. Then, remove all outputs (Slurm log files and TrimGalore output files) produced by this test-run.

I used the following command:

```bash
squeue -u $USER -l
```

The output was:

```bash
Sat Oct 18 16:11:59 2025
JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
37845773       cpu trimgalo kolganov  RUNNING       0:09     30:00      1 p0023
37845767       cpu ondemand kolganov  RUNNING       2:11   2:00:00      1 p0224
```

I saw both my VS code session and my slurm batch job id in the "Running" status. 

Once it was done running, the files appeared on the left side bar. Then, I proceeded to check 1) if the files are there using "ls"; 2) if the job has been successfully finished using "tail" ; 3) if the main FastQC output files are present using "ls -lh"; 4) checked again with "squeue -u $USER -l" if the job is still there (it wasn't, which means it was done); 5) checked my email for FAIL (wasn't there)

I used the following commands:

```bash
ls

 tail slurm-fastqc*.out
 tail slurm-fastqc*.err

 ls -lh results/fastqc
 ```

 The outputs were:

 ```bash
 data  README.md  results  scripts  slurm-fastqc-37845773.err  slurm-fastqc-37845773.out

# TrimGalore version:

  Quality-/Adapter-/RRBS-/Speciality-Trimming
  [powered by Cutadapt]
  version 0.6.10

  Last update: 02 02 2023

# Successfully finished script trimgalore.sh
Sat Oct 18 04:12:14 PM EDT 2025
Approx 80% complete for ERR10802863_R2_val_2.fq.gz
Approx 85% complete for ERR10802863_R2_val_2.fq.gz
Approx 90% complete for ERR10802863_R2_val_2.fq.gz
Approx 95% complete for ERR10802863_R2_val_2.fq.gz
Deleting both intermediate output files ERR10802863_R1_trimmed.fq.gz and ERR10802863_R2_trimmed.fq.gz

====================================================================================================

INFO:    Using cached SIF image
INFO:    gocryptfs not found, will not be able to use gocryptfs

total 43M
-rw-rw----+ 1 kolganovaanna PAS2880 2.4K Oct 18 16:11 ERR10802863_R1.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 675K Oct 18 16:12 ERR10802863_R1_val_1_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 348K Oct 18 16:12 ERR10802863_R1_val_1_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 18 16:12 ERR10802863_R1_val_1.fq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 2.3K Oct 18 16:12 ERR10802863_R2.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 676K Oct 18 16:12 ERR10802863_R2_val_2_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 341K Oct 18 16:12 ERR10802863_R2_val_2_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  21M Oct 18 16:12 ERR10802863_R2_val_2.fq.gz
```

All of the outputs have indicated to me that the scipt was succesfully run. I didn't get any FAIL emails. For final check, I uses this command again but modified (just to try it with my actual username):

```bash
squeue -u kolganovaanna -l
```

The output contained only my VS code session at this time (37845767, I had multiple because was doing the assignment for multiple days). This means my slurm batch job was completed. 

*Note*: the cleanup wasn't done until question 10 was answered (because I needed to look at the files to make my conclusions). 

To clean up, I used the following command:

```bash
rm -r results/fastqc slurm-fastqc*
```
I saw how the files on the left side bar disappeared

10. Illumina sequencing uses colors to distinguish between nucleotides as they are being added during the sequencing-by-synthesis process. However, newer Illumina machines (Nextseq, Novaseq) use a different color chemistry than older ones, and this newer chemistry suffers from an artefact which can produce strings of Gs with high quality scores that really should be Ns, especially at the end of reverse (R2) reads. In the FastQC outputs for the R2 file that you just produced with TrimGalore (recall that it runs FastQC after trimmming!), do you see any evidence for this problem? Explain.

I used the following command:

```bash
ls -lh results/fastqc*
```

The output for the command was:

```bash
total 43M
-rw-rw----+ 1 kolganovaanna PAS2880 2.4K Oct 18 16:11 ERR10802863_R1.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 675K Oct 18 16:12 ERR10802863_R1_val_1_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 348K Oct 18 16:12 ERR10802863_R1_val_1_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 18 16:12 ERR10802863_R1_val_1.fq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 2.3K Oct 18 16:12 ERR10802863_R2.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 676K Oct 18 16:12 ERR10802863_R2_val_2_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 341K Oct 18 16:12 ERR10802863_R2_val_2_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  21M Oct 18 16:12 ERR10802863_R2_val_2.fq.gz
```

To see if there's evidence of the described problem, I first looked at the html file for R2. I clicked on the file with "control" pressed and donwloaded it. The per base N content plot shows 0% for all reads. Per sequence quality scores tend to increase towards the end. The per base sequence quality doesn't seem to have anythign wrong with it as far as I can tell because everything is in the green zone. In the per base sequence content plot, it looks like %G is pretty stable. 
Next, I used the following command to look at the fq.gz file

```bash
zcat results/fastqc/ERR10802863_R2_val_2.fq.gz | less
```

The last lines in the output were:

```bash
AAAAAEEEEEEEEEEEEEAEEEEEAEEEEEEEEEEEEEEEEEEE6EEEEEEEEEEEEEAEEEEEEEEEEAAEE
@ERR10802863.30245032 30245032 length=74
AACTCGGTCATAAAGTCCAGCTTCTCGGCGGGGCTGTCATCCTGGAGGCAGGCCTGGTACGCCTCGTGCGTCTT
+
AAAAAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEAAEEEEEEEEEEEEE
@ERR10802863.30427015 30427015 length=72
ACGTCGTGTTTAAGAAAACAGAAATAAATCATTTAAGTGTATATCATTAGCACAGTCAATGAATCAATAC
+
AAAAAEEEEEEEEEEEEEEEEEEAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEA
@ERR10802863.24760709 24760709 length=74
TGTCCCTTCGCGTACTTTTTTGTTGGAGTGTTGCTCGGTAGAATGTCGCGTACAAACGGTACAATTAATTACCG
+
AAAAAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEAEEAEEEE
@ERR10802863.1023046 1023046 length=74
ACCTTCTGGGCAGCAGCTCCTCCGTAACCGCCGGCAGCGTGGCCACCACCGTAGCCGGCCGCAGCAGGGGCGGC
```
I think that if there was actually a poly-G problem, the end of the sequence would only contain a bunch of Gs. However, here this is not the case because we can still see E, C, A mixed with Gs. So, I think for R2 data they used older machines and there's no evidence of poly-G issue. 

11. We’ll assume that the data was indeed produced with the newer Illumina color chemistry. In the TrimGalore help info, find the relevant TrimGalore option to deal with the poly-G probelm, and again add the relevant line(s) from the help info to your README.md. Then, use the TrimGalore option you found, but don’t change the quality score threshold from the default.

I used the following command:

```bash
apptainer exec oras://community.wave.seqera.io/library/trim-galore:0.6.10--bc38c9238980c80e \
  trim_galore --help
```
I found this option: --quality INT.  Trim low-quality ends from reads in addition to adapter removal. For RRBS samples, quality trimming will be performed first, and adapter trimming is carried in a second round. Other files are quality and adapter trimmed in a single pass. The algorithm is the same as the one used by BWA INT from all qualities; compute partial sums from all indices to the end of the sequence; cut sequence at the index at which the sum is minimal). Default Phred score: 20.

And I also found this option: 
--nextseq INT This enables the option '--nextseq-trim=3'CUTOFF' within Cutadapt, which will set a quality cutoff (that is normally given with -q instead), but qualities of G bases are ignored.
This trimming is in common for the NextSeq- and NovaSeq-platforms, where basecalls without any signal are called as high-quality G bases. This is mutually exlusive with '-q INT'.

Interestingly, it also has a --polyA option. But not --polyG. 

I think that the best way is to use --nextseq and we're gonna use 20 by default because this command is mutually exclusive with '-q INT'. I added it into the tringalore.sh code and now the main part of the code looks like this: 

```bash
# Run TrimGalore
apptainer exec "$TRIMGALORE_CONTAINER" \
    trim_galore \
    --paired \
    --fastqc \
    --nextseq 20 \
    --cores "$SLURM_CPUS_PER_TASK" \
    --output_dir "$outdir" \
    "$R1" \
    "$R2"
```
12. Rerun TrimGalore with the added color-chemistry option. Check all outputs and confirm that usage of this option made a difference. Then, remove all outputs produced by this test-run again.

I used these commands:

```bash
sbatch scripts/trimgalore.sh "$R1" "$R2" results/fastqc

For monitoring: squeue -u kolganovaanna -l

ls -lh results/fastqc*

zcat results/fastqc/ERR10802863_R2_val_2.fq.gz | less
```

The outputs were:

```bash
Submitted batch job 37845795

JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
37845795       cpu trimgalo kolganov  RUNNING       0:10     30:00      1 p0023
37845767       cpu ondemand kolganov  RUNNING      44:27   2:00:00      1 p0224

total 42M
-rw-rw----+ 1 kolganovaanna PAS2880 2.5K Oct 18 16:54 ERR10802863_R1.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 679K Oct 18 16:54 ERR10802863_R1_val_1_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 356K Oct 18 16:54 ERR10802863_R1_val_1_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 18 16:54 ERR10802863_R1_val_1.fq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 2.3K Oct 18 16:54 ERR10802863_R2.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 681K Oct 18 16:54 ERR10802863_R2_val_2_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 345K Oct 18 16:54 ERR10802863_R2_val_2_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 18 16:54 ERR10802863_R2_val_2.fq.gz
```
I again downloaded the html. file. I was able to see a difference in the per base sequence content plot. The %G actually went down at the end of the reads. I think this is a good evidence that the adjustment worked. 

The last lines of my output for the last command used were:

```bash
AAAAAEEEEEEEEEEEEEAEEEEEAEEEEEEEEEEEEEEEEEEE6EEEEEEEEEEEEEAEEEEEEEEEEAAEE
@ERR10802863.30245032 30245032 length=74
AACTCGGTCATAAAGTCCAGCTTCTCGGCGGGGCTGTCATCCTGGAGGCAGGCCTGGTACGCCTCGTGCGTCTT
+
AAAAAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEAAEEEEEEEEEEEEE
@ERR10802863.30427015 30427015 length=72
ACGTCGTGTTTAAGAAAACAGAAATAAATCATTTAAGTGTATATCATTAGCACAGTCAATGAATCAATAC
+
AAAAAEEEEEEEEEEEEEEEEEEAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEA
@ERR10802863.24760709 24760709 length=74
TGTCCCTTCGCGTACTTTTTTGTTGGAGTGTTGCTCGGTAGAATGTCGCGTACAAACGGTACAATTAATTACC
+
AAAAAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEAEEAEEE
@ERR10802863.1023046 1023046 length=74
ACCTTCTGGGCAGCAGCTCCTCCGTAACCGCCGGCAGCGTGGCCACCACCGTAGCCGGCCGCAGCAGGGGCGGC
```

It seems like in the 24760709 line the G is gone from the end of the sequence. It was there in my previous output. This is the only difference I can see here. Might me that --nextseq gets rid of Gs only at the end?

I then cleaned up using this command: 

```bash
rm -r results/fastqc slurm-fastqc*
```


**Bonus**

1. The TrimGalore output FASTQ files are oddly named, ending in _R1_val_1.fq.gz and _R2_val_2.fq.gz – check the output files from your initial run to see this. This is not necessarily a problem, but could trip you up in a next step with these files.Therefore, modify your TrimGalore script to rename the output files after running TrimGalore, giving them the same names as the input files. Then, rerun the script to check that your changes were successful.

Below is the modifies script:

```bash
# Definining the output file names and creating final output files names. Using "mv -v" to finish renaming
sample_id=$(basename "$R1" _R1.fastq.gz)
R1_out_init="$outdir/${sample_id}_R1_val_1.fq.gz"
R2_out_init="$outdir/${sample_id}_R2_val_2.fq.gz"

R1_out="$outdir/$(basename "$R1")"
R2_out="$outdir/$(basename "$R2")"

mv -v "$R1_out_init" "$R1_out"
mv -v "$R2_out_init" "$R2_out"
```

I will check using these commands:

```bash
sbatch scripts/trimgalore.sh "$R1" "$R2" results/fastqc

squeue -u kolganovaanna -l

ls -lh results/fastqc*
```

The outputs were:

```bash
Submitted batch job 37845851

Sat Oct 18 17:34:24 2025
JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
37845851       cpu trimgalo kolganov  RUNNING       0:16     30:00      1 p0112
37845767       cpu ondemand kolganov  RUNNING    1:24:36   2:00:00      1 p0224

total 42M
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 18 17:34 ERR10802863_R1.fastq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 2.5K Oct 18 17:34 ERR10802863_R1.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 679K Oct 18 17:34 ERR10802863_R1_val_1_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 356K Oct 18 17:34 ERR10802863_R1_val_1_fastqc.zip
-rw-rw----+ 1 kolganovaanna PAS2880  20M Oct 18 17:34 ERR10802863_R2.fastq.gz
-rw-rw----+ 1 kolganovaanna PAS2880 2.3K Oct 18 17:34 ERR10802863_R2.fastq.gz_trimming_report.txt
-rw-rw----+ 1 kolganovaanna PAS2880 681K Oct 18 17:34 ERR10802863_R2_val_2_fastqc.html
-rw-rw----+ 1 kolganovaanna PAS2880 345K Oct 18 17:34 ERR10802863_R2_val_2_fastqc.zip
```

*Note*: I don't think I accomplished it but it was a good practice for me to at least try. 

I then cleaned up using this command:

```bash
rm -r results/fastqc slurm-fastqc*
```

And I also deleted this adjusted script from trimgalore.sh.

I then used the following commands to update github and commit to files:

```bash
git add scripts/trimgalore.sh README.md
git commit -m "Part B"
```


**Part C**

13. Write a for loop in your README.md to submit a TrimGalore batch job for each pair of FASTQ files that you have in your data dir.

I used the following loop:

```bash
for R1 in data/*_R1.fastq.gz; do
    R2="${R1/_R1.fastq.gz/_R2.fastq.gz}"
    sbatch scripts/trimgalore.sh "$R1" "$R2" results/trim_galore
done 
```


14. Monitor the batch jobs and when they are done, check that everything went well (if it didn’t, redo until you get it right). In your README.md, explain your monitoring and checking process. In this case, it is appropriate to keep the Slurm log files: move them into a dir logs within the TrimGalore output dir.

I ran the following loop and used the command to monitor:

```bash

for R1 in data/*_R1.fastq.gz; do
    R2="${R1/_R1.fastq.gz/_R2.fastq.gz}"
    sbatch scripts/trimgalore.sh "$R1" "$R2" results/trim_galore
done 

squeue -u kolganovaanna -l
```

The outputs were"

```bash
Submitted batch job 37845856
Submitted batch job 37845857
Submitted batch job 37845858
Submitted batch job 37845859
Submitted batch job 37845860
Submitted batch job 37845861
Submitted batch job 37845862
Submitted batch job 37845863
Submitted batch job 37845864
Submitted batch job 37845865
Submitted batch job 37845866
Submitted batch job 37845867
Submitted batch job 37845868
Submitted batch job 37845869
Submitted batch job 37845870
Submitted batch job 37845871
Submitted batch job 37845872
Submitted batch job 37845873
Submitted batch job 37845874
Submitted batch job 37845875
Submitted batch job 37845876
Submitted batch job 37845877


Sat Oct 18 17:55:36 2025
JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
37845856       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0112
37845857       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0008
37845858       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0008
37845859       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0032
37845860       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0033
37845861       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0068
37845862       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0068
37845863       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0102
37845864       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0102
37845865       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0036
37845866       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0035
37845867       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0035
37845868       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0050
37845869       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0049
37845870       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0049
37845871       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0143
37845872       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0142
37845873       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0142
37845874       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0018
37845875       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0018
37845876       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0017
37845877       cpu trimgalo kolganov  RUNNING       0:11     30:00      1 p0017
37845767       cpu ondemand kolganov  RUNNING    1:45:48   2:00:00      1 p0224
```

At the end, the jobs disappered when I used the "squeue -u kolganovaanna -l" command again, indicating they were done running. I also didn't get any FAIL emails. 

To move the outputs into logs dir, I used the following commands:

```bash
mkdir results/trim_galore/logs

mv slurm-*.out results/trim_galore/logs/
mv slurm-*.err results/trim_galore/logs/
```

At the end, I committed to README:

```bash
git add scripts/trimgalore.sh README.md
git commit -m "Part C"
```


**Part D**

15. Create a repository on GitHub, connect it to your local repo, and push your local repo to GitHub.

```bash
git add README.md
git commit -m "final commit"
git branch -M main
git remote add origin git@github.com:kolganovaanna/GA4.git
git push -u origin main
```

16. Create a new issue and tag GitHub users menukabh and jelmerp, asking us to take a look at your assignment.

```bash
Done in GitHub
```












