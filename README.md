Download Link: https://assignmentchef.com/product/solved-csci-3753-operating-systems-programming-assignment-one
<br>
<strong>Introduction</strong>

Welcome to the first programming assignment for CSCI 3753 – Operating Systems. In this assignment we will go over how to compile and install a modern Linux kernel, as well as how to add a custom system call. Most importantly you will gain skills and set up an environment that you will use in future assignments.




You will need the following software:

<ol>

 <li>CSCI 3753 Fall 2016 Virtual Machine</li>

</ol>




All other software and files should be installed on the virtual machine image. It is important that you do not update the virtual machine image as all the assignments are designed and written using the software versions that the VM is distributed with. Also make note that this assignment will require you to recompile the kernel at least twice, and you can expect each compilation to take at least half an hour and up to 5 hours on older machines.




After installing virtualbox, run <strong>sudo apt-get install cu-cs-csci-3753</strong> to install all the necessary packages required for the course.




<strong>Assignment Steps </strong>

<ol>

 <li>Configure the Grub</li>

 <li>Compile the kernel</li>

 <li>Add a system call</li>

 <li>Write a test program that uses the system call</li>

</ol>




<strong>Configuring the Grub </strong>

Grub is the boot loader installed with Ubuntu 14.04. It provides configuration options to boot from a list of different kernels available on the machine. By default Ubuntu 14.04 suppresses much of the boot process from users; as a result, we will need to update our Grub configuration to allow booting from multiple kernel versions and to recover from a corrupt installation. Perform the following:




Figure 1: Linux Grub Boot Menu <strong>Step 1: </strong>

From the command line, load the grub configuration file:

<strong>sudo emacs /etc/default/grub </strong>

(feel free to replace emacs with the editor of your choice).

<strong>Step 2: </strong>

Make the following changes to the configuration file:

<ol>

 <li>Comment out:</li>

</ol>

GRUB_HIDDEN_TIMEOUT=0

<ol start="2">

 <li>Comment out:</li>

</ol>

GRUB_HIDDEN_TIMEOUT_QUIET=true

<ol start="3">

 <li>Comment out:</li>

</ol>

GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash” and add the following new line directly below it:

GRUB_CMDLINE_LINUX_DEFAULT=””

<ol start="4">

 <li>Save updates <strong>Step 3: </strong></li>

</ol>

From the command line, update Grub: <strong>sudo update-grub</strong>.

<strong>Step 4: </strong>

Reboot your virtual machine and verify you see a boot menu as shown in Figure 1.

<strong>Downloading Linux Source Code </strong>

First, you have to download the linux source code that you will be using for this assignment where you will be adding your new system call, compiling the source and then running the kernel with the added system call. For this execute the following commands in the terminal.

<ol>

 <li><strong>cd /home </strong></li>

 <li><strong>sudo mkdir kernel </strong></li>

 <li><strong>cd kernel </strong></li>

 <li><strong>sudo apt-get source linux-image-$(uname -r) </strong></li>

 <li><strong>sudo apt-get install cu-cs-csci-3753</strong> (command to install all the packages necessary for this course)</li>

</ol>

The command <strong>uname –r</strong> gives you the current version of the kernel you are using. So that means you are downloading the same kernel as your current kernel. The command <strong>uname –a</strong> will give you the system architecture information (for example if your OS is 32 or 64 bit).

<strong>Compiling the Kernel Step 1:  </strong>

Typically it takes a long time to compile (2-3) hours to compile the kernel source code. To get around this, install ccache by running the following command <strong>sudo apt-get install ccache </strong>

ccache is a compiler cache. It is used to speed up recompilation by caching previous compilations and detecting when the same compilation is being done again. The first compilation will take the usual long time but after you make the necessary changes, the recompilations will be much faster after you install ccache.

<strong>Step 2: </strong>

Now you have to change permission of the debian scripts required to compile the source code. Just run the following commands in the linux source code directory.

<strong>sudo chmod a+x debian/scripts/* sudo chmod a+x debian/scripts/misc/* </strong>




or alternatively you can execute all the commands using <strong>sudo</strong>.

<strong>Step 3: </strong>

<strong> </strong>

The next step is to create the config file. <strong>sudo make localmodconfig</strong> command will create the config file. Run the command in the linux source directory you downloaded and extracted.

<strong>Step 4: </strong>




In the same folder, execute the command <strong>sudo make menuconfig</strong> . A pop up will be generated. Select the general set up (Figure 2)  and then select the option append to the release (Figure 3) and enter the name you want to give to the kernel you will be compiling ,installing and running (for example revision1). This name will appear when you reboot and from the grub  you can select it and check your new system call.

Figure 2: Configuring the menuconfig

Figure 3: Configuring the menuconfig

<strong>Step 5: </strong>

<strong> </strong>Now just run the following commands. <strong>sudo make -j2 CC=”ccache gcc” sudo make -j2 modules_install sudo make -j2 install </strong>

<strong>sudo reboot </strong>

When the grub option shows up, select the version you have installed with the new name appended.

<strong>Adding System Call Step 1: </strong>

Now you have to write the actual system call. First you will see that the linux source code is downloaded in the kernel folder. Go to that folder (for example linux-3.13.0) from the terminal. Then follow these steps.

<ol>

 <li><strong>sudo gedit arch/x86/kernel/helloworld.c</strong> and copy paste the following code snippet and save it</li>

</ol>

#include               &lt;linux/kernel.h&gt;              #include                &lt;linux/linkage.h&gt;             asmlinkage                long       sys_helloworld(void)

{

printk(KERN_ALERT “hello world
”);   return 0;

}




Explanation:

<ol>

 <li>Now kernel.h header file makes us able to use the many constants and functions that are used in kernel hacking, including the printk function.</li>

 <li>h defines macros that are used to keep the stack safe and ordered.</li>

 <li>Asmlinkage is a #define for some gcc magic that tells the compiler that the function should not expect to find any of its arguments in registers (a common optimization), but only on the CPU’s stack. It is defined in the linkage.h header file</li>

 <li>We have named the function sys_helloworld because all system calls’ name start with sys_ prefix.</li>

 <li>The function printk is used to print out kernel messages, and here we are using it with the KERN EMERG macro to print out “Hello World!” as if it were a kernel emergency. Note that depending on your system settings, printk message may not print to your terminal. Most of the time they only get printed to the kernel syslog (/var/log/syslog) system. If the severity level is high enough, they will also print to the terminal. Look up printk online or in the kernel docs for more information.</li>

</ol>

<strong>Step 2: </strong>




Now we have to tell the build system about our kernel call.  Open the file <strong>arch/x86/kernel/Makefile</strong>. Inside you will see a host of lines that begin with obj+=. After the end of the definition list, add the following line (do not place it inside any special control statements in the file) <strong>obj-y+=helloworld.o </strong>




<strong>Step 3: </strong>

Now you have to add that system call in the system table of the kernel. Go to the directory <strong>arch/x86/syscalls</strong> and open the file syscall_64.tbl (if you are using a 32 bit system then syscall_32.tbl). Look at the file and use the existing entries to add the new system call. Make sure it is added in the 64 bit system call section and remember the system call number as you will be using that later. Ask google if you find any trouble.

<strong> </strong><strong>Step 4: </strong>

Now you will add the new system call in the system call header file. Go to the location <strong>include/linux</strong> and open the file <strong>syscalls.h</strong> and add the prototype of your system call at the end of the file before the endif. Check the file structure or google if you have trouble.

<strong>Step 5: </strong>

Now recompile the kernel using the instructions given in the previous section.

<strong>Step 6: </strong>

Now that you have recompiled the kernel and rebooted into the installed kernel, you will be able to use the system call. Write a test c program (check google or type man syscall) to see how to call a system call and what header files to include and what are the arguments. The first argument a system call takes is the system call number we talked about before. If everything succeeds a system call returns 0 otherwise it returns -1. Check sudo tail /var/log/syslog to or type dmesg to check the printk outputs.

<strong>Step 7: </strong>

If everything until now has been perfect, now you have to write a new system call named simple_add that will take two  integer arguments namely number1, number2 and an integer pointer *result. You have to pass these arguments to the system call from your test program. In your system call implementation, you have to printk the numbers to be added, then you have to add those two numbers, store the result in the integer pointer result, printk the result and then print the result again in the test program that you will be running in the userspace. Do all the changes necessary from section 3’s step 1 to step 6 again to test this new system call.

<strong><u>References:</u></strong>

<ol>

 <li>Textbooks</li>

 <li><u>https://help.ubuntu.com/community/Grub2</u></li>

 <li><u>https://help.ubuntu.com/community/Kernel/Compile</u></li>

 <li><u>http://kernelnewbies.org/</u></li>

 <li>/kernels/linux-3.13.0/Documentation <strong>Grading </strong></li>

 <li>10 percent for compiling the first kernel</li>

 <li>10 percent for adding the necessary snippets for new system call</li>

 <li>10 percent for compiling with the new system call</li>

 <li>10 percent for the test program in userspace that will call the new system call,</li>

 <li>The following files have to be submitted by the due date

  <ol>

   <li>arch/x86/kernel/Simple_add.c</li>

   <li>arch/x86/kernel/Makefile</li>

   <li>arch/x86/syscalls/syscall_64.tbl</li>

   <li>include/linux/syscalls.h</li>

   <li>/var/log/syslog</li>

   <li>Source code for your test program</li>

   <li>A README with your contact information, information on what each file contains information on where each file should be located in a standard build tree, instructions for building and running your test program, and any other pertinent info.</li>

  </ol></li>

</ol>