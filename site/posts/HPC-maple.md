<!--
.. title: High Performance Computing with Maple
.. author: Mike Croucher
.. slug: HPC-Maple
.. date: 2016-05-26 12:40:19 UTC+01:00
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

Many people who use [Maple](http://www.maplesoft.com/) on Sheffield's High Performance Computing (HPC) cluster do so interactively. They connect to the system, start a graphical X-Windows session and use Maple in exactly the same way as they would use it on their laptop. Such usage does have some benefits: giving access to more CPU cores and memory than you'd get on even the most highly specified of laptops.

Interactive usage on the HPC system also has problems. Thanks to [network latency](https://en.wikipedia.org/wiki/Latency_(engineering)), using a Graphical User Interface over an X-Windows connection can be a painful experience. Additionally, long calculations can tie up your computer for hours or days and if anything happens to the network connection during that time, you risk losing it all!

If you spend a long time waiting for your Maple calculations to run, it's probably time to think about moving to batch processing.

## Batch processing

The idea behind batch processing is that you log in to the system, send your computation to a queue and then log out and get on with your life. The HPC system will process your computation when resources become available and email you when it's done. You can then log back in, transfer the results to your laptop and continue your analysis.

So, batch processing frees up your personal computer but it can also significantly increase your throughput. With batch processing, you can submit hundreds of computations to the queue simultaneously.  The system will automatically process as many of them as it can in parallel -- allowing you to make use of dozens of large computers at once.

Batch processing is powerful but it comes at a price and that price is complexity.

## Converting interactive worksheets to Maple language files

You are probably used to interacting with Maple via richly formatted worksheets and documents. These have the file extension **.mw** or **.maple**. Unfortunately, it is not possible to run Maple worksheets in batch mode so it is necessary for us to convert them to **maple language files** instead.

A [Maple Language File](http://www.maplesoft.com/support/help/maple/view.aspx?path=Formats%2FMPL) has the extension **.mpl** and is a pure text file. To convert a worksheet to a Maple Language File, open the worksheet and click on  **File->Export As->Maple Input**

![Convert to mpl](/images/convert_to_mpl.png)

## An example

Here is an example **.maple** worksheet and the corresponding **.mpl** Maple Language File, created using the conversion process detailed above. We also have a **Job submission script** that will be explained later.

* [series_example.maple](/maple/hpc1/series_example.maple) - Original Maple worksheet
* [series_example.mpl](/maple/hpc1/series_example.mpl) - Maple Language File
* [run_maple_job.sh](/maple/hpc1/run_maple_job.sh) - Job submission script

If you look at the **.mpl** file in a text editor, you will see that it contains plain text versions of all the Maple input commands.
```
myseries := series(sin(x), x = 0, 10);
poly := convert(myseries, polynom);
plot(poly, x = -2*Pi .. 2*Pi, y = -3 .. 3);
```

This is the file that we can run on the HPC system in batch mode.

The **job submission script** is a set of instructions to the HPC system's scheduler. It tells the system how much memory you want to use, what program you want to run and so on.  Here is its contents

```
!/bin/bash
# Request 4 gigabytes of real memory (mem)
# and 4 gigabytes of virtual memory (mem)
#$ -l mem=4G -l rmem=4G

#Make Maple 2015 available
module load apps/binapps/maple/2015

#Run Maple with the input file, **series_example.mpl**
maple < series_example.mpl
```

To run the example on the system:

* Transfer **series_example.mpl** and **run_maple_job.sh** to a directory on the HPC system. They both need to be in the same directory.
* Log in to the system using a command line terminal and **cd** to the directory containing the files.
* Use the **ls** command to confirm you really are in the right directory. You should see something like this:
```
[ab1test@testnode02 maple_example]$ ls

run_maple_job.sh  series_example.maple  series_example.mpl
```
* Submit the job to the queue with the **qsub** command
```
qsub run_maple_job.sh
```

* You should see something like
```
[ab1test@testnode02 maple_example]$ qsub run_maple_job.sh

Your job 1734126 ("run_maple_job.sh") has been submitted
```

The job number will differ from the one above. It is automatically allocated by the system and uniquely identifies the job.

* At this point, you could log off the system and do something else if you wished but, since this is such a short job, it won't be long before the results appear. A few seconds to a minute under normal conditions.

* Run the **ls** command again to see the results files.

```
[fe1mpc@testnode02 maple_example]$ ls
run_maple_job.sh  run_maple_job.sh.e1734126  run_maple_job.sh.o1734126  series_example.maple  series_example.mpl
```

There are two new files:

* run_maple_job.sh.e1734126 - contains any error messages. Hopefully empty here
* run_maple_job.sh.o1734126 - Contains the output of your job

The numbers at the end refer to the job number.