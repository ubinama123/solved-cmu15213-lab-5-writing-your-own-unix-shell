Download Link: https://assignmentchef.com/product/solved-cmu15213-lab-5-writing-your-own-unix-shell
<br>
<span class="kksr-muted">Rate this product</span>

The purpose of this assignment is to become more familiar with the concepts of process control and sig- nalling. You’ll do this by writing a simple Unix shell program that supports job control.

Logistics

You may work in a group of up to two people in solving the problems for this assignment. The only “hand- in” will be electronic. Any clarifications and revisions to the assignment will be posted on the course Web page.

Hand Out InstructionsSITE-SPECIFIC: Insert a paragraph here that explains how the instructor will hand out

the shlab-handout.tar file to the students. Here are the directions we use at CMU.Start by copying the file shlab-handout.tar to the protected directory (the lab directory) in which

you plan to do your work. Then do the following:

• Type the command tar xvf shlab-handout.tar to expand the tarfile.• Type the command make to compile and link some test routines.• Type your team member names and Andrew IDs in the header comment at the top of tsh.c.

Looking at the tsh.c (tiny shell) file, you will see that it contains a functional skeleton of a simple Unix shell. To help you get started, we have already implemented the less interesting functions. Your assignment

1

is to complete the remaining empty functions listed below. As a sanity check for you, we’ve listed the approximate number of lines of code for each of these functions in our reference solution (which includes lots of comments).

<ul>

 <li>eval: Main routine that parses and interprets the command line. [70 lines]</li>

 <li>builtin cmd: Recognizes and interprets the built-in commands: quit, fg, bg, and jobs. [25lines]</li>

 <li>do bgfg: Implements the bg and fg built-in commands. [50 lines]</li>

 <li>waitfg: Waits for a foreground job to complete. [20 lines]</li>

 <li>sigchld handler: Catches SIGCHILD signals. 80 lines]</li>

 <li>sigint handler: Catches SIGINT (ctrl-c) signals. [15 lines]</li>

 <li>sigtstp handler: Catches SIGTSTP (ctrl-z) signals. [15 lines]Each time you modify your tsh.c file, type make to recompile it. To run your shell, type tsh to the command line:unix&gt; ./tshtsh&gt; [type commands to your shell here]General Overview of Unix ShellsA shell is an interactive command-line interpreter that runs programs on behalf of the user. A shell repeat- edly prints a prompt, waits for a command line on stdin, and then carries out some action, as directed by the contents of the command line.The command line is a sequence of ASCII text words delimited by whitespace. The first word in the command line is either the name of a built-in command or the pathname of an executable file. The remaining words are command-line arguments. If the first word is a built-in command, the shell immediately executes the command in the current process. Otherwise, the word is assumed to be the pathname of an executable program. In this case, the shell forks a child process, then loads and runs the program in the context of the child. The child processes created as a result of interpreting a single command line are known collectively as a job. In general, a job can consist of multiple child processes connected by Unix pipes.If the command line ends with an ampersand ”&amp;”, then the job runs in the background, which means that the shell does not wait for the job to terminate before printing the prompt and awaiting the next command line. Otherwise, the job runs in the foreground, which means that the shell waits for the job to terminate before awaiting the next command line. Thus, at any point in time, at most one job can be running in the foreground. However, an arbitrary number of jobs can run in the background.For example, typing the command line</li>

</ul>

tsh&gt; jobs

2

causes the shell to execute the built-in jobs command. Typing the command line tsh&gt; /bin/ls -l -d

runs the ls program in the foreground. By convention, the shell ensures that when the program begins executing its main routine

int main(int argc, char *argv[])the argc and argv arguments have the following values:

• argc == 3,• argv[0] == ‘‘/bin/ls’’, • argv[1]== ‘‘-l’’,• argv[2]== ‘‘-d’’.

Alternatively, typing the command line

tsh&gt; /bin/ls -l -d &amp;runs the ls program in the background.

Unix shells support the notion of job control, which allows users to move jobs back and forth between back- ground and foreground, and to change the process state (running, stopped, or terminated) of the processes in a job. Typing ctrl-c causes a SIGINT signal to be delivered to each process in the foreground job. The default action for SIGINT is to terminate the process. Similarly, typing ctrl-z causes a SIGTSTP signal to be delivered to each process in the foreground job. The default action for SIGTSTP is to place a process in the stopped state, where it remains until it is awakened by the receipt of a SIGCONT signal. Unix shells also provide various built-in commands that support job control. For example:

• jobs: List the running and stopped background jobs.• bg &lt;job&gt;: Change a stopped background job to a running background job.• fg &lt;job&gt;: Change a stopped or running background job to a running in the foreground. • kill &lt;job&gt;: Terminate a job.

The tsh SpecificationYour tsh shell should have the following features:

• The prompt should be the string “tsh&gt; ”.3

<ul>

 <li>The command line typed by the user should consist of a name and zero or more arguments, all sepa- rated by one or more spaces. If name is a built-in command, then tsh should handle it immediately and wait for the next command line. Otherwise, tsh should assume that name is the path of an executable file, which it loads and runs in the context of an initial child process (In this context, the term job refers to this initial child process).</li>

 <li>tsh need not support pipes (|) or I/O redirection (&lt; and &gt;).</li>

 <li>Typing ctrl-c (ctrl-z) should cause a SIGINT (SIGTSTP) signal to be sent to the current fore- ground job, as well as any descendents of that job (e.g., any child processes that it forked). If there is no foreground job, then the signal should have no effect.</li>

 <li>If the command line ends with an ampersand &amp;, then tsh should run the job in the background. Otherwise, it should run the job in the foreground.</li>

 <li>Each job can be identified by either a process ID (PID) or a job ID (JID), which is a positive integer assigned by tsh. JIDs should be denoted on the command line by the prefix ’%’. For example, “%5” denotes JID 5, and “5” denotes PID 5. (We have provided you with all of the routines you need for manipulating the job list.)</li>

 <li>tsh should support the following built-in commands:

  <ul>

   <li>–  Thequitcommandterminatestheshell.</li>

   <li>–  Thejobscommandlistsallbackgroundjobs.</li>

   <li>–  Thebg&lt;job&gt;commandrestarts&lt;job&gt;bysendingitaSIGCONTsignal,andthenrunsitin the background. The &lt;job&gt; argument can be either a PID or a JID.</li>

   <li>–  Thefg&lt;job&gt;commandrestarts&lt;job&gt;bysendingitaSIGCONTsignal,andthenrunsitin the foreground. The &lt;job&gt; argument can be either a PID or a JID.</li>

  </ul></li>

 <li>tsh should reap all of its zombie children. If any job terminates because it receives a signal that it didn’t catch, then tsh should recognize this event and print a message with the job’s PID and a description of the offending signal.Checking Your WorkWe have provided some tools to help you check your work.Reference solution. The Linux executable tshref is the reference solution for the shell. Run this program to resolve any questions you have about how your shell should behave. Your shell should emit output that is identical to the reference solution (except for PIDs, of course, which change from run to run).Shell driver. The sdriver.pl program executes a shell as a child process, sends it commands and signals as directed by a trace file, and captures and displays the output from the shell.Use the -h argument to find out the usage of sdriver.pl:</li>

</ul>

4

unix&gt; ./sdriver.pl -hUsage: sdriver.pl [-hv] -t &lt;trace&gt; -s &lt;shellprog&gt; -a &lt;args&gt; Options:

<pre>-h-v-t &lt;trace&gt;-s &lt;shell&gt;-a &lt;args&gt;-g</pre>

Print this messageBe more verboseTrace fileShell program to testShell argumentsGenerate output for autograder

We have also provided 16 trace files (trace{01-16}.txt) that you will use in conjunction with the shell driver to test the correctness of your shell. The lower-numbered trace files do very simple tests, and the higher-numbered tests do more complicated tests.

You can run the shell driver on your shell using trace file trace01.txt (for instance) by typing: unix&gt; ./sdriver.pl -t trace01.txt -s ./tsh -a “-p”(the -a “-p” argument tells your shell not to emit a prompt), orunix&gt; make test01

Similarly, to compare your result with the reference shell, you can run the trace driver on the reference shell by typing:

unix&gt; ./sdriver.pl -t trace01.txt -s ./tshref -a “-p”

or

unix&gt; make rtest01

For your reference, tshref.out gives the output of the reference solution on all races. This might be more convenient for you than manually running the shell driver on all trace files.

The neat thing about the trace files is that they generate the same output you would have gotten had you run your shell interactively (except for an initial comment that identifies the trace). For example:

bass&gt; make test15./sdriver.pl -t trace15.txt -s ./tsh -a “-p” ## trace15.txt – Putting it all together#tsh&gt; ./bogus./bogus: Command not found.tsh&gt; ./myspin 10Job (9721) terminated by signal 2tsh&gt; ./myspin 3 &amp;[1] (9723) ./myspin 3 &amp;tsh&gt; ./myspin 4 &amp;

5

[2] (9725) ./myspin 4 &amp;tsh&gt; jobs[1] (9723) Running ./myspin 3 &amp; [2] (9725) Running ./myspin 4 &amp; tsh&gt; fg %1Job [1] (9723) stopped by signal 20 tsh&gt; jobs[1] (9723) Stopped[2] (9725) Runningtsh&gt; bg %3%3: No such jobtsh&gt; bg %1[1] (9723) ./myspin 3 &amp;tsh&gt; jobs[1] (9723)[2] (9725)tsh&gt; fg %1tsh&gt; quitbass&gt;

Hints

<ul>

 <li>Read every word of Chapter 8 (Exceptional Control Flow) in your textbook.</li>

 <li>Use the trace files to guide the development of your shell. Starting with trace01.txt, make sure that your shell produces the identical output as the reference shell. Then move on to trace file trace02.txt, and so on.</li>

 <li>The waitpid, kill, fork, execve, setpgid, and sigprocmask functions will come in very handy. The WUNTRACED and WNOHANG options to waitpid will also be useful.</li>

 <li>When you implement your signal handlers, be sure to send SIGINT and SIGTSTP signals to the en- tire foreground process group, using ”-pid” instead of ”pid” in the argument to the kill function. The sdriver.pl program tests for this error.</li>

 <li>One of the tricky parts of the assignment is deciding on the allocation of work between the waitfg and sigchld handler functions. We recommend the following approach:– Inwaitfg,useabusylooparoundthesleepfunction.– Insigchldhandler,useexactlyonecalltowaitpid.While other solutions are possible, such as calling waitpid in both waitfg and sigchld handler, these can be very confusing. It is simpler to do all reaping in the handler.</li>

 <li>In eval, the parent must use sigprocmask to block SIGCHLD signals before it forks the child, and then unblock these signals, again using sigprocmask after it adds the child to the job list by calling addjob. Since children inherit the blocked vectors of their parents, the child must be sure to then unblock SIGCHLD signals before it execs the new program.</li>

</ul>

<pre>RunningRunning</pre>

<pre>./myspin 3 &amp;./myspin 4 &amp;</pre>

<pre>./myspin 3 &amp;./myspin 4 &amp;</pre>

6

The parent needs to block the SIGCHLD signals in this way in order to avoid the race condition where the child is reaped by sigchld handler (and thus removed from the job list) before the parent calls addjob.

<ul>

 <li>Programs such as more, less, vi, and emacs do strange things with the terminal settings. Don’t run these programs from your shell. Stick with simple text-based programs such as /bin/ls, /bin/ps, and /bin/echo.</li>

 <li>When you run your shell from the standard Unix shell, your shell is running in the foreground process group. If your shell then creates a child process, by default that child will also be a member of the foreground process group. Since typing ctrl-c sends a SIGINT to every process in the foreground group, typing ctrl-c will send a SIGINT to your shell, as well as to every process that your shell created, which obviously isn’t correct.Here is the workaround: After the fork, but before the execve, the child process should call setpgid(0, 0), which puts the child in a new process group whose group ID is identical to the child’s PID. This ensures that there will be only one process, your shell, in the foreground process group. When you type ctrl-c, the shell should catch the resulting SIGINT and then forward it to the appropriate foreground job (or more precisely, the process group that contains the foreground job).</li>

</ul>