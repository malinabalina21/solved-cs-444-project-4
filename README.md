Download Link: https://assignmentchef.com/product/solved-cs-444-project-4
<br>
<h1></h1>

In class, we added the mult.c and mfork.c programs into the xv6 system (by adding the C code, and editing the Makefile). I need you to make sure those programs are a regular part of your xv6 system. You will use these programs to test the correct functioning of your code and they will be used to test your code when grading your assignment. The C files mult.c and mfork.c can be found in my Lab4 directory, ~chaneyr/Classes/cs444/Labs/Lab4 The purpose of the mult program is to just take a long time to complete. The purpose of the mfork program is to just fork a number of processes in xv6. Add these into the UPROGS macro in your Makefile.

<h1>Part 2 – Add some tracking information to each process</h1>

Add four members to the struct proc data structure (found in proc.h). Remember, you are the kernel developer and master level C programmer, so you should be comfortable manipulating the kernel structures. Of course, I put all this stuff within #ifdef blocks using a macro called PROC_TIME.

<ol>

 <li>Add a member type struct rtcdate. You’ll find the definition of that structure in the h file. I like to call this member begin_date.</li>

 <li>Add 3 members of type unsigned int. Good names for those 3 members are:

  <ol>

   <li>ticks_total – this will represent the total number of time ticks that the process has run.</li>

   <li>ticks_begin – this will be used to help calculate the total number of time ticks the process has used.</li>

   <li>sched_times – this will be used to count the number of times the process has been scheduled to run.</li>

  </ol></li>

</ol>

You can find the definition of struct rtcdate in the date.h file. While you are in the date.h file, make sure it has multiple-include protection.

Now that you have the new data members in the struct proc data structure, it’s time to put them to use. When a process is first <strong><em>allocated</em></strong>, set the begin_date to the “current” date/time. How do you know where a process is first allocated? You might look for a function called something like allocproc(). The qemu clock seems to synchronize to UTC time at startup. If you really want it set to the current Pacific time, ask me how (it is a little make magic). Finding the right call to get the date/time is a little awkward. I recommend you look in the lapic.c file for the function cmostime(). However, you probably don’t want to spend too much time looking at that in that file, it’s a bit icky. Calling the cmostime() function is a wee bit awkward, because you must pass an address or pointer. Remember that putting an &amp; in front of a variable/structure when calling a function will pass the address of the variable/structure. Decide to ignore the goto and the label found in that function; we won’t talk about them.

Once you set the begin_date member, set ticks_total , ticks_begin, and sched_times to zero in the same area of the code (when a new process is allocated).

Now let’s look at the scheduler() code (in proc.c). You’ll notice that the scheduler runs as an infinite loop (see the for(;;) ?). There are 2 places where you want to add a few lines of code into the scheduler() routine. One is just before the newly chosen process is scheduled and the other is just after that process returns back to the scheduler. The first place (before the newly chosen process runs), is pretty easy to find. Look for the place where the state of the process is set to RUNNING. I dropped an #ifdef block just before that. That block does 2 things, it increments the sched_times member for the chosen process, and sets the begin_ticks member to the current number of ticks that have accumulated for the vm. How do you find out how many “ticks” the system has been up? That is an excellent question. My recommendation is to look for a function called uptime(). <em>Unfortunately</em>, because of how the kernel function uptime() is defined and implemented (as sys_uptime()), you cannot directly call it. You have a choice to either 1) replicate the capability of uptime() in the scheduler() code (which I <strong>do not</strong> like), or 2) make a new function that returns the same thing (which I <strong>do</strong> like). If you decide to do the second option, I recommend you have sys_uptime() directly call your new function. I went with option 2; I don’t like to duplicate code.

Now, about that second place to drop in a few lines of code. What do you think the following 2 lines of code from the scheduler() function do?

swtch(&amp;(c-&gt;scheduler), p-&gt;context); switchkvm();

ln addition, notice that immediately after those lines of code, the c-&gt;proc variable is set to 0 (aka NULL). So, might this be that when a new process is actually scheduled to run by calling swtch() and switchkvm(), and it returns back to scheduler following those calls? Looks like a good place to update the total_ticks member of the process (after return from switchkvm()).

<h1>Part 3 – Modify ps to show time tracking information</h1>

Now that you have all this great time tracking information for each process, modify your ps program to display it.

One of the fun things you’ll notice is that you may need to put a zero in where a value is represented as a single digit. Notice that the month shown in the start time is 05, not just 5. It’s a simple trick, but worth learning. If you had a fully functioning printf() (or for this example cprintf()), it would be just a change to the format string. However, your printf() is not capable of that (and don’t spend the month or 6 working on the format characters in the xv6 version of printf()).

<h1>Part 4 – Add a rand() function</h1>

<h1>You will need to use a function the generates random numbers. More specifically, pseudorandom numbers. This does not need to be cryptographically secure random numbers, just something reasonable, such as the standard Unix/Linux rand() function returns.</h1>

Amazingly, if you happened to read to the bottom of the man page for rand, in section 3 of the man pages, you can see some source code for a version of rand() that works just fine for this purpose. Differing from the man page example, you will call your functions rand() and srand(), <strong>NOT</strong> myrand() and mysrand(). If you prefer to use some other swanky random number generator, that’s fine but not required.

Your implementation of rand() must be done in 2 files: rand.c and rand.h. The rand.c file will contain the implementation of rand() and srand(). The rand.h file will contain the declarations (aka prototypes) of rand() and srand() <strong>AND</strong> must contain the macro RAND_MAX, yep you need a macro.

Based on the code in the man page, you’d establish RAND_MAX to be 32767 (2^15), but we are going to use 2^31. So, your RAND_MAX macro should be (1 &lt;&lt; 31). Yes, use parenthesis around the shift operation. Do you see in the code in the man page where the random value is returned? It does a % 32767. You will replace the 32767 with your RAND_MAX macro.

Great, you’ve written your code for rand() and srand(), but you need to get them into the kernel. They must be callable from within the kernel, so they must be linked with the kernel. Luckily, this is very easy. Up at the top of the Makefile, there is a variable called OBJS. At the end of the list of .o files in the OBJS variable, just add rand.o (and don’t omit the trailing backslash). Assuming that your rand.c and rand.h files don’t have any compilation errors, simply running make should rebuild the xv6 kernel with your new calls in it.

<h1>Part 5 – Create the rand command</h1>

Since we have our great kernel function rand(), let’s go ahead and create a command that  makes use of the rand() function. This is a command in the same way that cps and getppid are new commands that we’ve added to xv6. Since we already have a file called rand.c, we will call our new command random and implement it in a file called random.c.

If you use grep to look for where and how getppid() and cps() functions are created and used in commands, you’ll see how to create the random command. Think about the following command to find things:

grep getppid *.[chS]

Be sure you add random in the PROGS variable in the Makefile.

You want to make sure you understand what the argint() (and its friends argstr() and argptr()) function does with the functions in the sysproc.c file.

By the way, a full implementation of the rand() functions would allow us to generate repeatable sequence of random numbers, per application. This one may not do that, but that is okay for this project.

<h1>Part 6 – Modify the scheduler function</h1>

Now you going to make some real changes to the scheduler function. You are going to implement <strong>lottery scheduling</strong>. This would be an excellent time to review/read <u>chapter 9 from</u> <u>the OSTEP book</u>. This is another terrific time to use #ifdef blocks in your code to make it easy

to go back and forth between a previous working version and a new version. I used LOTTERY_SCHED as the #define in my code.

In the proc.h file, you are going to define 3 macros: DEFAULT_NICE_VALUE,

MAX_NICE_VALUE, and MIN_NICE_VALUE. The values to use for these macros are: 20, 40, and 1. In the struct proc data structure, you need to add an additional member, called nice_value. The values we are going to use for the lottery scheduling will vary from 1 to 40, with 20 as the default “nice” value. A higher value means that the process has a higher probability to be scheduled. A lower nice value means the process has a lower probability to be scheduled.

Typically, when a process is allocated from them shell, it is assigned the default nice value (20 aka DEFAULT_NICE_VALUE). <strong>However, when a child process is created via </strong><strong>fork(), the child process inherits the nice value from its parent process</strong>.

Implementing lottery scheduling is pretty easy. In the scheduler() function, sum the nice values for all the RUNNABLE processes. Generate a random number (using your brand spanking new random number generator) between 1 and the sum of nice values (put your mod hat on, there’s a high % you’ll need it). Loop through the process table for all RUNNABLE processes, summing the nice values (this is different from the initial sum of nice values before). As soon as the sum of nice values exceeds the random number, schedule that process. Easy peasy.

I will warn you about a condition that slowed me down. There will be times when there are not any RUNNABLE processes (i.e. the sum of nice values for all RUNNABLE processes is zero). When this is true, just keep looping in the scheduler.

<h1>Part 7 – Write a new system function called renice()</h1>

Just as your created system functions for getppid(), cps() and rand(), you need to create a system function called renice(). The renice() system function <strong>takes 2 arguments</strong>, a pid and a new nice value. The process with the given pid has its nice_value changed to the given new value. Make sure the new nice value is between MAX_NICE_VALUE, and MIN_NICE_VALUE.

If the nice value is out of bounds, renice() returns a 1 and leaves the nice value of the process unchanged. If the pid given to renice() does not exist, return a 2. If renice() succeeds, return a 0.

<h1>Part 8 – Write programs called renice and nice</h1>

The new program nice takes 2 command line options: the nice value as arrgv[1]] and the name of the program to exec as argv[2]. It should first set its own nice value (using renice()) and then exec the program on the command line, in argv[2]. See the image below.

The program renice takes a new nice value as argv[1] and applies that to <strong>all</strong> the pids following on the command line. See the image below.

The other place to look for information about nice and renice are the man pages. Your implementation of nice and renice should behave a lot like Unix/Linux commands of the same name.

<h1>Part 9 – Modify ps to show the nice values</h1>

Now that you have nice values for all your programs and can change them, you need to add the ability to ps to view the nice values.

The following image shows you what this should look like.

When you have processes with large differences in nice values, you should notice that the processes with large nice values get scheduled more frequently and accumulate more time/ticks on the CPU. If you don’t notice this, something is wrong with your implementation.

When you are done with the Lab4, submit your code to TEACH. Remember how we used the command “make teach” to produce a tar and gzipped file that you can submit into TEACH? Do that and be done.





