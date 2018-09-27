# Simon-lab-workshop


Today, I would like for all of us to get a handle on the basics of working and navigating on the UConn cluster as well as working with NextGen sequencing data and familiarity with some of the commonly used tools. I think each of us should work on a single dataset at first and then maybe today or next week, we will be able to get some of the raw reads assembled and can start actually doing stuff with those files. 

### Table of Contents
* [Basics](https://github.com/erg55/Simon-lab-workshop/blob/master/README.md#basics)  
* [Cluster things](https://github.com/erg55/Simon-lab-workshop/blob/master/README.md#cluster-things)  
* [Assembling sequence data](https://github.com/erg55/Simon-lab-workshop/blob/master/README.md#assembling-sequence-data)
* [Querying assembly](https://github.com/erg55/Simon-lab-workshop/blob/master/README.md#querying-assembly)
* [Useful scripts](https://github.com/erg55/Simon-lab-workshop/blob/master/README.md#useful-scripts)
* [Tools](https://github.com/erg55/Simon-lab-workshop/blob/master/README.md#tools)


## BASICS

Let's start with logging into the cluster: 
```
    ssh USERNAME@xanadu-submit-ext.cam.uchc.edu
```

Enter your password. 

Next, create a working directory:
```
    mkdir ThursdayFun
```
Navigate into the working directory...try using tab to autocomplete your suggestions after typing a capital "T":
```
    cd ThursdayFun
```
Now lets copy some data files from my folder (which has had it's permission modified so others can access it) into your current folder...let's try these three samples P0075_CS_I27897_S125, P0075_CS_I27898_S126 and P0075_CS_I27899_S127. Please be courteous in my folder and don't delete anything. 
```
    cp /home/CAM/egordon/Dropbox/P0075_CS_I27897_S125_L001_R1_001.fastq.gz .
```
```
    cp /home/CAM/egordon/Dropbox/P0075_CS_I27897_S125_L001_R2_001.fastq.gz .
```
You could also use the wildcard to match anything and copy all matching files at once. 
```
    cp /home/CAM/egordon/Dropbox/P0075_CS_I27897_S125_L00* .
```
Use head to look into the files you downloaded:
```
    head P0075* 
```

Oops they are still zipped, Press Control+C to interrupt a command.

Instead you can try use zcat to print the file and then pass it to the head command with the "|" symbol:
```
    zcat P0075* | head
```

That's what fastq files look like! Neat!

In the future, if you want to work with these files you can point to their path in my folder and work with them the same way as if they were in your current folder. Doing this will make sure we are not wasting too much storage space on the cluster. 

If you want to read more information or look at the options you can use with various shell commans try running the command with the double flag --help:

```
    head --help
```

There are many other commonly used commands in bash, here are some of other most used ones. Try them out!

**ls** prints out all the contents of the folder you're in or that you list.

**gunzip** will unzip a file.

**cat**, short for concatenate, will print out the contents of an unzipped file.

**du**, disk utility, will tell you the size of a file.

**touch** will create a blank text document of the name you give it.

**nano** will allow you to edit or create a text document based on the name you give.

**pwd** will Print your current Working Directory. 

<img src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" data-canonical-src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" width="25" height="40"> EASTER EGG <img src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" data-canonical-src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" width="25" height="40">

Let's do something so we barely every have to use that **pwd** command. Go up one folder to your home directory: 
```
    cd ..
```
or
```
    cd ~
```
or just
```
    cd 
```
Create a hidden file called .bash_profile: 
```
    touch .bash_profile
```
Go into that file with **nano**:
```
    nano .bash_profile
```
And paste the following into the file: 
```
    export PS1="username@xanadu-submit-ext.cam.uchc.edu:  \w:"
```
Exit with control+X, then "Y"

Log out of and then back into the cluster:
```
    exit
    ssh USERNAME@xanadu-submit-ext.cam.uchc.edu
```
Now you can see where you are all the time! Neat! 

## CLUSTER THINGS

One of the benefits of using the cluster is that commonly used programs are already installed on it and thus easy to use. To activate the commands associated with these programs use **module load** and the program name. You can use tab to see all the programs available. Try typing:

```
    module load b
```
And use tab to see all the programs starting with the lowercase letter "b" already installed. Let's activate blast:

```
    module load blast
```
Now if we type blastn --help we can see how to use this command:
```
    blastn --help
```
Before loading this module this wouldn't have worked. 

The other more major benefit of the cluster is the ability to use the large amounts of computing resources to speed up complex computing tasks. To use these resources, you must submit a script that specifies certain things in a standard way. Below is an example of the first several lines needed in a script with annotations. 

```
#!/bin/bash
#This specifies this script should be read as a bash script as opposed to some other language

#SBATCH --job-name=trinity
#This is the name given to the job for your own reference later

#SBATCH -N 1
#This refers to the number of nodes you want for your job. Almost always 1 is appropriate. To expand a job across more than one node, you generally need to incorporate a multi-threading program like MPI and often the amount of speed gained is not worth the effort. This program is hard to use. 

#SBATCH -n 1
#This is a useful parameter when using multithreading programs like MPI. Otherwise keep it one. 

#SBATCH -c 30
#This is the number of processors you want for the job. The more you devote the faster the job will run...but sometimes it can be harder to get the resources to run a job if you request too much.  

#SBATCH --mem=200G
This is the amount of memory you need for the job. Assemblers often require a high amount of memory. 

#SBATCH --partition=general
#This is the partition you want to submit a job too. They are several partitions where the nodes have a different amount of memory and processors. 

#SBATCH --qos=general
#This was newly added....not sure exactly why but you need it now

#SBATCH --mail-type=END
#This says to send you an email when the job is finished.

#SBATCH --mail-user=email@uconn.edu
#Email address

#SBATCH -o myscript_%j.out
#This is the file name created from the standard output of the job in the folder in which you ran the script.

#SBATCH -e myscript_%j.err
#This is the file name created from the standard error of the job in the folder in which you ran the script.

```
For quick running jobs you can start interactive sessions specifying resources with the same flags as the earlier bash script. It is a good idea to do this before running any commands on the cluster to avoid running things on the head node.

```
srun --pty --qos=general --partition=general --mem=1G bash
```

Much more information about the Xanadu resource can be found online [here](https://bioinformatics.uconn.edu/resources-and-events/tutorials-2/xanadu/)

Create a script named test.sh with the following text using **nano test.sh**:

```
#!/bin/bash
#SBATCH --job-name=test
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 1
#SBATCH --partition=general
#SBATCH --qos=general
#SBATCH --mail-type=END
#SBATCH --mem=1G
#SBATCH --mail-user=diler.haji@uconn.edu
#SBATCH -o myscript_%j.out
#SBATCH -e myscript_%j.err
echo "Hello World!"
```
Now you can use **sbatch test.sh** to submit the script:
```
    sbatch script.sh
```
Use **squeue** to check the status of the job requesting jobs run by only your username:
```
    squeue -u USERNAME
```
Or you can look at all the jobs running or in line to run:
```
squeue
```


```
sinfo -s
```

## ASSEMBLING SEQUENCE DATA
##### Assess quality with fastqc

For reasons that I cannnot determine, these gz compressed files are not compatible with fastqc as they are. First we should **gunzip** them and then to save space **gzip** again. After that they work. Mysterious. Remember to work with the dataset you picked and change the files names below as appriopriate.
```
gunzip P0075_CS_I27897_S125_L001_R1_001.fastq.gz
gzip P0075_CS_I27897_S125_L001_R1_001.fastq
gunzip P0075_CS_I27897_S125_L002_R1_001.fastq.gz
gzip P0075_CS_I27897_S125_L002_R1_001.fastq
```
Then we can run fastqc on each of the files. It is better to do this before combining them since the two lanes may have behaved differently:
```
module load fastqc
fastqc P0075_CS_I27897_S125_L001_R1_001.fastq.gz
fastqc P0075_CS_I27897_S125_L002_R1_001.fastq.gz
```

Download the fastqc file with **rsync**. First open a new terminal window but don't log onto the cluster. Navigate to the folder you want this file to download in and run this command:
```
rsync --progress USERNAME@xanadu-submit-ext.cam.uchc.edu:~/ThursdayFun/*.html .
```

##### For Windows
Open the folder containing the file you want to upload, open windows powershell or equivalent to type prompt and pscp

```
pscp username@biocluster.ucr.edu:clusterdirectory/testfolder/ local_folder_name\ 
```

How do the fastqc reports look? Pretty pristine for me. Perhaps they have already been trimmed? We shall see. 




### Week II 



The sequence data is split across two lanes. Let's combine each set into the twopaired ends reads before assembly:
```
    cat P0075_CS_I27897_S125_L001_R1_001.fastq.gz P0075_CS_I27897_S125_L002_R1_001.fastq.gz > S125_R1.fastq.gz
    cat P0075_CS_I27897_S125_L001_R2_001.fastq.gz P0075_CS_I27897_S125_L002_R2_001.fastq.gz > S125_R2.fastq.gz
```

##### Deduplication

For this we can use a tool in bbmap called clumpify. Deduplication makes assembly faster by getting rid of optical or pcr duplicates which don't contribute to coverage. 
```
module load bbmap
clumpify.sh in1=S125_R1.fastq.gz in2=S125_R2.fastq.gz out1=S125_dedup_R1.fastq.gz out2=S125_dedup_R2.fastq.gz dedupe
```
This may take a little while...about 5 minutes.

##### Trimming

For this we use Trimmomatic. We will also need to download a file with the sequence of the Illumina adaptor we think was used into our working directory. This command tells trimmomatic to remove any sequences matching Illumina adaptors, remove low quality (< 3 quality score) trailing or leading bases, using a sliding window of 4 bases removing windows where the quality score is less than 20 on average and finally discarding any read less than 50 bp long after all trimming.
```
module load Trimmomatic
cp ../../egordon/metagenomes/files/TruSeq3-PE.fa .
trimmomatic-0.36.jar PE -phred33 S125_dedup_R1.fastq.gz S125_dedup_R2.fastq.gz S125_forward_paired.fq.gz S125_forward_unpaired.fq.gz S125_reverse_paired.fq.gz S125_reverse_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:50;
```
This also takes a while. It looks like bases and reads were trimmed. 


##### Merging

Next is merging the paired reads. This can only occur if the insert size was short enough that the two 150 bp reads on either side overlapped. I have been told merging about half is a good percentage.
```
module load bbmap
bbmerge.sh in1=S125_forward_paired.fq.gz in2=S125_reverse_paired.fq.gz out=S125_merged.fq.gz outu1=S125_unmergedF.fq.gz outu2=S125_unmergedR.fq.gz
```

Combine all single read files for assembly:
```
cat S125_unmergedF.fq.gz S125_unmergedR.fq.gz S125_forward_unpaired.fq.gz S125_reverse_unpaired.fq.gz > S125_allsinglereadscombined.fq.gz
```

##### Run SPAdes assembler

Let's first just make sure the syntax is right for this to work. Run the command below specifying only one thread under -t:
```
/home/CAM/egordon/spades/SPAdes-3.12.0-Linux/bin/spades.py -t 1 --merged S125_merged.fq.gz -s S125_allsinglereadscombined.fq.gz -o S125trimmedspades.assembly/
```
If it looks like it's running correctly, stop it with control+C and lets try and submit it as a job because it may require some time and more memory than available on the head node. Generally you shouldn't run a bunch heavy tasks on this node. 

Make a script like below named spades.sh (make sure to update your email):
```
#!/bin/bash
#SBATCH --job-name=spades
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -c 16
#SBATCH --partition=general
#SBATCH --qos=general
#SBATCH --mail-type=END
#SBATCH --mem=100G
#SBATCH --mail-user=diler.haji@uconn.edu
#SBATCH -o myscript_%j.out
#SBATCH -e myscript_%j.err
/home/CAM/egordon/spades/SPAdes-3.12.0-Linux/bin/spades.py -t 16 --merged S125_merged.fq.gz -s S125_allsinglereadscombined.fq.gz -o S125trimmedspades.assembly/
```

Submit it:
```
sbatch spades.sh 
```


You can check progress on the spades.log file in the assembly folder or in the .out file. SPAdes first implements a read error correction tool before it starts to assemble based on a set of increasing k-mer sizes. 

Let's stop here today as we wait for our assembly to finish (should take about a half hour!












# WEEK III

## ASSESSING AND QUERYING ASSEMBLY

##### QUAST

We can use a program to get some basic assembly stats. This can be useful comparing the effectiveness of various programs or parameters. 
```
module load quast
quast.py contigs.fasta
```

Let's try something to view these files without downloading them. Log out and back into the cluster this time using the -X flag. 

```
exit
ssh -X username@xanadu-submit-ext.cam.uchc.edu
```
This should open up a program called X11 for Mac users. For Windows users, follow the instructions below: 

Download [Xming](https://sourceforge.net/projects/xming/) here. This gives you an X server for windows.
Next, you need to install the X11 apps.
```
sudo apt-get install x11-apps
```
Now you can log into the cluster using the previous -X flag. You will get an error message about not having an .Xauthority file, but it automatically creates one for you.

The program below will alow you to view image files like jpg, png and tiff. There should be a similar program for html and pdf files but I can't find any installed on the cluster at the moment.

```
module load imageMagick
display filename.jpg
```

##### BLAST 

Let's use BLAST to find some target genes. On Genbank find the [COI sequence of a cicada](https://www.ncbi.nlm.nih.gov/nuccore/?term=Kikihia+AND+COI) and create a file on the cluster with that sequence in fasta format. Then try and BLAST it against our assembly after making our assembly into a nucleotide BLAST database: 

```
module load blast
makeblastdb -in contigs.fasta -dbtype nucl
blastn -query COI.fasta -db contigs.fasta
```
Not a great format for automating things. Try this next:
```
blastn -query COI.fasta -db contigs.fasta -outfmt 6 
```
You can also add an evalue cutoff and output it into a file next.Make sure to include the parameter to search for the invertebrate mitochondrial genetic code.
```
tblastx -query COI.fasta -db contigs.fasta -outfmt 6 -out COI.res -evalue 1e-50 -query_gencode 5 -db_gencode 5
```

Where you able to find (a) matching contig(s)? To check you can query the contig using grep and request N number of lines after it to print out the full sequence:

```
grep -A N CONTIGNAME contigs.fasta
```

You can blast this online against the (nr database)[https://blast.ncbi.nlm.nih.gov/Blast.cgi?PROGRAM=blastn&PAGE_TYPE=BlastSearch&LINK_LOC=blasthome] in order to better confirm what exactly it is. 

Ok lets try and find the whole mitochondrion searching in amino acid space.  Copy the file from my folder first:

```
cp xx .
tblastx -query mito.fas -db contigs.fasta -outfmt 6 -evalue 1e-50 -out mito.res -query_gencode 5 -db_gencode 5
```

Hmm looks like several contigs? It would be tedious to pull them all out using grep. Lets' try this custom script that outputs them into a result.fas file. You can also specify a stricter evalue cutoff if you need to. 

```
module load python/2.7.8
python ~/scripts/ericblparserspades.py mito.res . 1
```

Rename the result.fas file to mito.contigs:

```
mv result.fas mito.contigs
```


##### BWA 
Alright, lets see if we can assemble the mitochondrial genome better. What we can try is mapping the reads back to our assembled contigs which may allow us to extend the edges. We can use this with a read mapper like BWA or MIRA. Let's try BWA first mapping the deduplicated reads back.
```
module load bwa
bwa index mito.contigs
bwa mem -t 2 -k 50 -B 10 -O 10 -T 90 mito.contigs ../S190_dedup_R1.fastq.gz ../S190_dedup_R1.fastq.gz > bwafile
```

t is threads, k is match length needed, B is mismatch penalty, O is gap opening penalty, T is minimal alignment score. We output that file. Take a peek at that file with head. This file includes more than we need so

##### SAMTOOLS
This file includes more than we need so we can use samtools to only view the mapped reads and then pull those reads out specifically.
```
module load samtools
samtools view -b -F 4 bwafile > mapped.bam
samtools fastq mapped.bam > mapped.fastq
```

Then we can try to create an assembly for those reads only:

```
/home/CAM/egordon/spades/SPAdes-3.12.0-Linux/bin/spades.py -t 2 -s mapped.fastq -o mito.spades.assembly/
```

##### Bandage

This should go very quickly...then we can view the assembly graph (.gfa file) in [Bandage](https://rrwick.github.io/Bandage/). Download it with rsync or copy and paste it locally. 


##### Geneious

Then we can also download mapped reads (fastq) and map them back onto a reference mitochondrial genome in Geneious. If you don't have Geneious installed we can try and get that working afterwards. 




##### MITObim

Another strategy we can try is an iterative read mapper like MITObim which uses mira:

```
nano Seed.fasta
cat allsinglereadscombined.fq.gz merged.fq.gz >> all.fq.gz
module load mira
~/MITObim/MITObim.pl -start 17 -end 20 -sample Opiss -ref Opis -readpool ./RCW5085-READC.fastq --quick ./Seed.fasta -NFS_warn_only
```




<img src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" data-canonical-src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" width="25" height="40"> EASTER EGG <img src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" data-canonical-src="https://www.clker.com/cliparts/C/v/B/g/z/i/easter-egg-md.png" width="25" height="40">

You can also include things in the .bash_profile file that you want to run everytime you log onto the cluster. For examle you can paste module load blast into this file as well as various other commonly used programs. 

This is also where you can set up aliases for running a command with a short cut. 

```
alias [alias_name]="[command_to_alias]"
```

```
alias ll="ls -lhaG"
alias h='history'
alias du='du -bh'
```
You can check which aliases are active by typing in alias.

My .bash_profile file now looks like this. 

```
export PS1="egordon@xanadu-submit-ext.cam.uchc.edu:  \w:"
alias ll="ls -lhaG"
alias h='history'
alias du='du -bh'
module load blast
module load bwa
module load samtools
module load python/2.7.8
module load seqtk
module load quast
module load Trimmomatic
module load bbmap
```

# WEEK IV


##### Seqtk
module load seqtk
seqtk seq -a in.fastq.gz > out.fasta



# WEEK V
## PHYLOGENOMICS



## USEFUL SCRIPTS

Most of these are stolen from my collaborator [Alex Knyshov](https://github.com/AlexKnyshov/main_repo)

#### BLAST Parser Script


Found [here](https://github.com/AlexKnyshov/main_repo/blob/master/blast/mainblparser.py)

You can git clone it:

```
git clone https://github.com/AlexKnyshov/main_repo/blob/master/blast/mainblparser.py
```
or 
```
wget https://raw.githubusercontent.com/AlexKnyshov/main_repo/master/blast/mainblparser.py
```

#### Align

#### Alignment viewer 

### Dataset manipulation scripts

#### Tools:  
[FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)  
[BLAST](https://www.ncbi.nlm.nih.gov/books/NBK279690/)  
[BBmap](https://sourceforge.net/projects/bbmap/)  
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)  
[Spades](http://spades.bioinf.spbau.ru/release3.9.0/manual.html)  
[QUAST](http://bioinf.spbau.ru/en/quast)
[BWA](http://bio-bwa.sourceforge.net/bwa.shtml)  
[SeqTK](https://github.com/lh3/seqtk)  
[MITObim](https://github.com/chrishah/MITObim)  
[Bandage](https://rrwick.github.io/Bandage/)  

