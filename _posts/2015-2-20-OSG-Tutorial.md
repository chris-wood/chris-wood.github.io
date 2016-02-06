---
layout: post
title: A Beginner's Guide to the Open Science Grid (OSG)
comments: true
---

# 1. Introduction
I am writing this tutorial because I could not find anything similar online. It is my goal to give you a very simple and hands-on introduction to using the grid so that you can get up and running with large-scale computations quickly. Everything in this document is largely based on my experience using the Open Science Grid (OSG) and working with Mats Rynge, one of the members of the OSG and Pegasus teams. If you find this information helpful, please let me know by sending me a message at woodc1@uci.edu. If it was not helpful, or if you find any mistakes or can offer suggestions for improvement, please also let me know by sending me a message at the same address.

The remainder of this tutorial is organized in the following manner. Section 2 starts with an introduction to OSG, describing why you might want to use it and some basic requirements for actually needing to use it. Section 3 then goes into the process of connecting to the grid, uploading and downloading files, and maintaining sessions for extended periods of time. Section 4 continues with an overview of how to set up a workflow for a job, submit and monitor a workflow, and analyze the results. Section 5 then finishes with some closing remarks. It is best to read this tutorial in a linear fashion so as to not miss any important details.

# 2. Purpose
The OSG is a massively distributed computation platform that gives researchers an easy and painless way to execute very large compute-bound applications in a significantly reduced amount of time. Running programs on the OSG is a form of result parallelism in which the work (e.g. number of inputs) for a particular program are evenly distributed among many, many computational nodes. For example, given a program that has a regular running time denoted by T, partitioning the input of this program among K nodes will theoretically yield a reduced running time of T/K. The wonderful thing about the OSG is that there is generally no limiting upper bound on K, so we are free to speed up our programs to the largest extent possible.

By dividing the computation among K nodes we are usually left with K distinct output files that need to be processed further. Should these results need to be combined, or reduced, you simply need to walk all of the output files using their own reduction operation (e.g. sum the contents of each output file) to obtain a final result, cumulative result. As another example, if one OSG workflow (a workflow is conceptually equivalent to a set of jobs run on the grid) outputs a single integer value from each computational node and you want to find the minimum of all such outputs, you may walk each output file, keeping track of the minimum value observed thus far, and return the final minimum value upon completion. Clearly, the combination or reduction process for these output files is specific to your particular application, and can be done online or offline depending on what kind of analysis is actually required.

With a general description of how the grid is used to speed up jobs with a lot of computation, you may now wonder how easy it is to actually use the grid. Fortunately, leveraging the resources of the OSG is easier than one may think. It only requires the following:

1. A working program that may or may not require input,
2. An existing data set, if said data set is expensive or difficult to obtain online, or Python code that is capable of generating the data set online, and
3. Patience when configuring the workflow!

In what follows we will walk through the complete process of signing into the main OSG node, setting up and initializing a workflow, checking on the status of the workflow, and finally, collecting the output files when complete. Again, I stress that this tutorial is only meant to be an introduction. For more help, please see the more technical OSG documentation available online or contact Mats Rynge on the OSG/Pegasus team.

# 3. First Steps: Interacting with the Grid (SSH, SFTP, and so on)

The first step to using the OSG is to actually connect to the main server node: osg-xsede.grid.iu.edu. User authentication is done using a traditional RSA-based public/private key pair. That is, you must provide your public key to the server, which will then be used to authenticate you with your private key. I will not go into the details of generating public/private key pairs as there are many tutorials available online. Alternatively, you may consult the openssl man page for some basic information.

With an appropriate public/private key pair and OSG username, which should be configured in cooperation with Mats Rynge, one may use SSH to open a remote shell on the osg-xsede.grid.iu.edu node using the following command:

ssh -v -i /path/to/private-key username@osg-xsede.grid.iu.edu

Using other programs such as sftp to transfer files can be used in a similar fashion:

sftp -v -i /path/to/private-key username@osg-xsede.grid.iu.edu

In cases where large data sets need to be reliably uploaded from a local machine to the grid for use in a workflow, one may wish to use the rsync program. This program automatically detects timeouts and restarts file transfers in the event of failures, ensuring that all files from one source directory ~/src/dir/ on the local machine are transferred to some remote directory osg-xsede.grid.iu.edu:/local-scratch/username/some/remote/dir, as follows:

rsync -a -v -i /path/to/private-key -e ssh osg-xsede.grid.iu.edu:/local-scratch/username/some/remote/dir/. ~/src/dir/.

Another useful program that may be of value is screen, which is a program for creating persistent sessions on a remote machine that keep running even when the user logs out remotely. This is useful if a program must be run on the remote machine for an extended period of time and you cannot reliably maintain a connection throughout the entire duration of the its running time. There are many extensive tutorials for screen online, but for simplicity I summarize some of the most useful features below.

First, check to see that you have screen installed and in your shell's path. This can be done by typing 'screen', as follows:

$ screen

If you have screen installed, you will meet the screen shown in Figure 1 (or something similar). Note that all OSG servers have screen installed, as far as I know.

![Screen welcome page](/images/osg_image1.png)

Every program (including the shell) runs in a window, and each window is given a unique identifier. This can be used to later restore a screen that was started after you log out of the system. Once all programs in a screen window are exited, the window will close, and when all windows close, screen closes as well. The following commands show you how to create windows and move between them:

C-a c (create window)
C-a N (go to window identified by N, where 0 < N < 10)

Note that C-a is synonymous for the key combination "control-a." If you forget which window is which, simply type "C-a w" to see a list of available windows appear in the bottom of your terminal window, as shown in Figure 2.

![Sample list of windows with two bash shells running](/images/osg_image2.png)

Now assume that you have a couple windows (sessions) open and are running programs that might take some time to finish. You don't want to close the program to exit the session. Rather, you want to detach the window from the session, which effectively decouples the session from the terminal. To do this, simply type "C-a d." At this point, if there are no other windows running, screen will close. To reattach to the session, simply run screen as follows:

$ screen -r

and you will be brought right back to where you left off. Figures 3 and 4 below illustrate the effectiveness of this technique.

![Start of a long program before detachment](/images/osg_image3.png)

![Continued run of a long program after detachment and then reattachment. I had a terrible Internet connection when running this, which is why you see so many request timeouts.](/images/osg_image4.png)

There are many other useful features that come with screen, but these basic features should be more than sufficient for all tasks you will need to accomplish on the main OSG node. If you need further assistance, search for a more comprehensive screen tutorial online.

# 4. Workflow setup
With the ability to connect to the main OSG node and configure a session, upload and download files, and more, we are now ready to create a workflow. Before going any further, I need to stress that you should not start a workflow without permission from Mats or any other member from the OSG/Pegasus team. Novice users who spawn one too many jobs will use precious cycles that might be better suited for other jobs.

To get started, I will describe the basic set of files that are included in the majority of all workflows. The hierarchy of these files, shown from the "base" directory of the workflow, is shown below.

- dax-generator.py
- dax.xml
- job-wrapper
- pegasusrc
- sites.xml
- submit
- inputs
-  ...

As a user you should mainly be concerned with the dax-generator.py script, job-wrapper script, and input directory. The purpose of these files is outlined below.

### dax-generator.py
This script creates and configures the OSG workflow, which involves partitioning the job workload appropriately among however many computational nodes is needed, wiring input files to jobs (including program binaries and data files), and specifying all output files that will be created on each computational node (including error files).
### job-wrapper
This script contains the code that will execute your program for one particular slice of the input. How you determine an input slice is up to you; it may be a range of indices that your program will loop over, a set of files that the program will read from, and so on. This is specific to your application and workflow.
### inputs
This directory contains the input files used in your workflow. It is common to place the program binaries (e.g. the class files for a Java program) in this folder.

Before looking at actual code for these files, let's first devise a simple application that one might need to run on the grid. Computations in the field of extremal graph theory commonly require many, many CPU hours to compute. For example, recent work by Lange et al. [1] used approximately 200,000 CPU hours on the grid. Imagine how long that would take on a simple machine!

Assume we want to tackle a related Ramsey-like computation on the grid. Namely, we want to run a program that decides whether or not for every red/blue (binary) edge coloring of a complete graph G on n vertices it holds that G contains a blue or red triangle. Furthermore, assume we want to find the smallest such n that this is true. These types of problems are often the subject of Ramsey theory. Fortunately, this problem (i.e. finding the smallest n such that this condition holds) is already solved - it is known to be 6. However, for the sake of illustration, assume we didn't know that the answer was 6, and we instead wanted to check if it was true for n = 8. A naive algorithm for answering the previous question is to do the following:

For each 2^28 edge colorings:
	Color the edges of G
	If there does not exist a red or blue triangle, output no. Otherwise, continue
Output yes.

Note that for a complete graph on 8 vertices there are exactly (8*(8-1) / 2) = (8*7 / 2) = 28 edges, and so there are 2^28 possible edge colorings. A Java program that implements this algorithm given an integer n is shown in Listing 1. You may run this program with n = 5 and n = 6 to see that in fact the desired value of n is 6, as shown below:

$ java ArrowDecider 5
false
$ java ArrowDecider 6
true

{% gist 61440af44d5b434f9068 %}

You do not need to understand all the details (I merely included it because this type of work is fascinating), but you do need to understand four key elements:

1. The program requires a single input integer n that is used to construct the complete graph.
2. The program outputs a single Boolean value, true or false, depending on whether or not the edge coloring condition was upheld.
3.The program has no external library dependencies (if it did, you would need to work with the OSG/Pegasus team to make sure these libraries would be available on the grid's computational nodes).

Since we don't want a single job to perform all 2^28 computations by itself, as this would defeat the purpose of using the grid, we must modify the program so that it only checks a specific range of edge colorings. We do this by specifying two more input parameters for the lower and upper bound of coloring indices. That is, two integers i and j such that 0 <= i < j < 2^28, where i is the lower bound and j is the upper bound. The modified program that uses these bounds to check edge colorings is shown in Listing 2.

{% gist 7e1bbcc55d382b92b793 %}

This program is now ready to be run on the grid. As previously mentioned, we need to modify the dax-generator.py and job-wrapper files to run our workflow. A heavily annotated version of the dax-generator.py script is shown in Listing 3. You need only be concerned with the highlighted lines for now, as they are responsible for tying in input and output files and configuring each job to run with the appropriate arguments. Be sure to carefully read this file so you understand how input files are registered with the grid, output files are specified, and the parameters for each particular job are configured. As previously mentioned, our Java class files are assumed to be located in the ./inputs directory where this workflow is based. You should follow this pattern for your workflows too.

{% gist 2c4a5c57802f850830c6 %}

Now that we've configured the workflow creation script (dax-generator.py), we need to set up the job-wrapper script that will actually run each job. Recall that in the dax-generator.py script we added a set of jobs to the DAX catalogue with a different set of parameters. Those parameters are passed into the job-wrapper script as command line arguments, which the script then parses and uses to run the Java program, as shown in Listing 4.

{% gist 7b3af36b9543bb06b31e %}

At this point, we are finally ready to submit our workflow. If you're in the workflow base directory, you may do so as follows:

$ ./submit

Once the job is successfully submitted, you will be prompted with the output shown in Figure 6.

![OSG/Pegasus output after submitting a job.](/images/osg_image5.png)

In addition, you will also receive a friendly email letting you know the job is at the start phase, as shown below.

***** Pegasus Workflow Event ****
Time:     2013-08-28T21:21:49+0000
Workflow: /local-scratch/caw/workflows/caw/pegasus/ramsey-arrowing-test/20130828T212141+0000
Job id:   49766786-e17d-4e01-9389-d7306bc14bfa
Event:    start

pegasus-status:

UNREADY   READY     PRE  QUEUED    POST SUCCESS FAILURE %DONE
 2,057       0       0       3       0       0       0   0.0
Summary: 1 DAG total (Running:1)

Once a job has been submitted, you can monitor and change its status using the pegasus CLI. You can view the possible commands by typing "pegasus-" in the terminal and hitting the tab key. The most important of these commands are shown below for emphasis.

### pegasus-status
Check on the status of an existing workflow (similar to Figure 7 below). You can view how many jobs are queued, running, complete, and how many have failed. This is very useful for making sure your jobs are running smoothly and not halting indefinitely, which is a huge problem in this shared grid.
### pegasus-remove
Remove an existing workflow from the grid. This effectively cancels and abandons all work in progress, so only use this if you are absolutely sure you do not need the output files.
### pegasus-statistics
Gather various statistics about a workflow. This is useful for informational or debugging purposes.

![Sample output from pegasus-status.](/images/osg_image6.png)

Once a job is finished, you will receive yet another friendly email, similar to the following:

***** Pegasus Workflow Event ****
Time:     2013-08-28T22:15:22+0000
Workflow: /local-scratch/caw/workflows/caw/pegasus/ramsey-arrowing-test/20130828T212141+0000
Job id:   49766786-e17d-4e01-9389-d7306bc14bfa
Event:    at_end
Status: 0

pegasus-status:

UNREADY   READY     PRE  QUEUED    POST SUCCESS FAILURE %DONE
     0       0       0       0       0   2,060       0 100.0
Summary: 1 DAG total (Success:1)

It's finally time to examine the results. In more realistic computations, we would probably need to download the files from the server and do some sort of offline processing. However, since the output of our program is a simple Boolean value, we will check the correctness of the output using grep. Recall that we have already shown the optimal answer for n to be 6. Therefore, the output from running this program for n = 8 should be "yes" or true, as well. To check, we first need to navigate to the output directory, which is contained in /local-scratch/username/. For me, this directory has the following subdirectories: data, outputs, and workflows. We are interested in the output, so we change to that directory and drill down to our specific workflow, which is identified by the workflow name specified in the dax-generator.py script and the time when the job was submitted. Once inside this output directory, you may notice that the output is divided into a set of directories. This is done so as to limit the number of files in a single directory, which can make processing a large amount of files very difficult. The output for my "base" directory, as well as the contents of one of these subdirectories, is shown in Figure 8.

![Contents of the "base" directory, a single subdirectory, and a file within that subdirectory.](/images/osg_image7.png)

Now, from the base directory, we simply utilize the grep program to check to see if any of the outputs returned "no" or false, which would mean that we've disproved a result that's been known for decades, or, more likely, we have a bug in our program! The output of running grep for the 'false' string through all subdirectories in a recursive fashion is below.

$ grep 'false' -r *
$

Nothing! Fantastic! This is what we hoped for. Now use grep again to search for outputs containing "true". You will see something similar to the following.

$ grep 'true' -r *
000/out_50855936_50987008:true
000/out_48627712_48758784:true
000/out_195166208_195297280:true
000/out_91488256_91619328:true
000/out_50331648_50462720:true
...
016/out_57147392_57278464:true
016/out_54394880_54525952:true
016/out_57540608_57671680:true
016/out_55443456_55574528:true
016/out_57409536_57540608:true
$

Therefore, our program computed the desired result and in a much faster time than if we did it on our own machine.

And that's it, folks. You should now possess a basic understanding of the OSG workflow necessary to get your large-scale computations up and running with your large-scale jobs much quicker.

# 5. Wrapping Up
In this tutorial we've gone over the basics of using the OSG resources for large scale computations. You should be able to find your way around and configure other similar workflows with this background knowledge and little bit of scripting expertise, but please consult myself (woodc1@uci.edu), Mats Rynge, or another member of the OSG/Pegasus team should you have any specific questions about setting up a workflow. Also, I want to stress that I would not have been able to write this tutorial without Mats' help throughout my studies. He truly is the ultimate guru!

# 6. References
[1] Ivan Livinsky, Alexander Lange, and Stanislaw Radziszowski. Computation of the Ramsey Numbers R(C_4, K_9) and R(C_4, K_10).
