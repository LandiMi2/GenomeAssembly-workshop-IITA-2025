##  Quality control and trimming

We will perform FastQC analysis on one cell of HiFi reads and OmniC reads of Star apple samples. Then using fastp we will clean the data based on the discussion we will have in class after discussing parameters to filter out. 

Before we start we will learn how to run our analysis on background using screen. For information about [screen](https://www.geeksforgeeks.org/screen-command-in-linux-with-examples/). 
The basic commands for screen that we will always use in the entire tutorial

*Start a New Screen Session* <br>
`screen -S back` then hit enter, this will open a new terminal window <br>

Once you have opened the new screen, print "This is my first screen" using this command `echo This is my first screen`  <br>

*Detach from a Screen Without Closing It* <br>
press `control+A+D` all at once <br>
*List All Running or Detached Screen Sessions* <br>
`screen -ls` <br>
*Reattach to a Screen Session* <br>
`screen -r back` <br>

*Create another screen and print "This is my second screen"*  <br>

If you want to delete screen <br>
- Within the screen you can type `exit` then hit enter <br>
- Outside screen, you can delete screen using `screen kill 27732` (this time you have to specify the screen ID) <br>

Now that we are comfortable with screen let us run below analysis on background. 

**Running QC using fastqc**

On your home directory create QC folder 

`mkdir -p QC`

We have subseted 50,000 reads to run QC on the HiFi reads as the whole data set will take some time to run. 

`fastqc -o QC -t 2 cel1_50K_hifi.fastq`









