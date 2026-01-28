---
title: "SEED Lab: Buffer Overflow"
date: 2025-02-20 10:00:00 +0000
categories: [WriteUps, Labs, SeedLab]
tags: [buffer overflow, seedlab]
author: "JT"
pin: false
toc: true
comments: true
---

## Task 1
I started with the initial setup. I had so many problems with VMs across my laptop and desktop computer that I ended up going with the 16.04 version. This worked well, but unlike the example in class, I needed to run one extra command `sudo ln -sf /bin/zsh /bin/sh` This links my sh shell to zsh because on Ubuntu 16.04 there are protections against Set-UID. I also used the `sudo sysctl kernel.randomize_va_space=0` to turn off ASLR. The other vim and touch commands are used to copy files the included `exploit.c` and `stack.c` files.


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250203121832.png)


Next I compiled the `stack.c` file to `stack` using the following commands to ensure there was not stack canaries and it can be executed

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250203121850.png)


Then I ran `gdb` on the stack and set a breakpoint where we knew the code was vulnerable because of the `strcpy` then we found the base pointer `$ebp` is and address of the `&buffer`. That gave us the values below of `$ebp = 0xbfffeb48` and `&buffer = 0xbfffeb28`. Then we calculate the offset which gives us `0x20 = 32`
**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcQQBPQQ8CMwEJKYqmNz02CcVhWsXFkg_X8f3zsTvq3Vo5B_81_P3Zse3WD8IRgyyQ1fO1BzHhB87j9Yl7xUc4KtyKLUIvH2vwyx5tRlcU-8iQebekIDy792fPJ7tY5wQN1pdFaKA?key=M9UqJo6gFwpGru4IKynhiX5z)

The using this information I can use vim to add the information that I need to `exploit.c`. 
The first line is used to overwrite the return address. We use the address we found during debugging to do so `$ebp = 0xbfffeb48`. 
The second line uses `memcpy` to write the shellcode into memory to be executed after we hijack execution flow by modifying the return address.
**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXc1s4OGUSvh_PfhqV1K3yfLqZ0S-OmK7vorLkX-xUqMjisiPxMezDYiSx9W_K6UOG9ubc7zGI0BFs_oyzzNBqKpGapZ1DUAHiBuoA2w8AZlh8-VosI9F-OQ54zU4ZCSohBNRy2x?key=M9UqJo6gFwpGru4IKynhiX5z)**

Now we have our exploit written so lets give it a shot... I did get a shell, but not a root shell, so it's useless, at this point I go back and check my work.
**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfarmDAjL6HQ-1xL9hmqpuA9PMWJSRNmoNN2xK_m7UBfE0pbZSf3Z48eZhajZBmvAwxVEQeye6EGksSXm1Y5BIjMbURcH2QOa0sCFo5Ud0nP2_QXXEDJcQzdYxa-OnD-m0RkIVvrA?key=M9UqJo6gFwpGru4IKynhiX5z)**

I noticed I missed a step and run the following commands in order to give the right permission to the file. By changing the owner to root and giving it `4755` mode it will be able to set the uid to root.
**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfeSJH2T-zICOLl5hPMiHMRRH7uFq3pcS1AKPC1UedZIU_N2hdjp6thFVcLr5fZuCQwSOxO7ikQ0TEVu6uolFA5FruyA4i03vl0IFcgOKuXBT3lQ0EJKM5xO-bxgvDTIAxSm6LWmw?key=M9UqJo6gFwpGru4IKynhiX5z)**

After running it again we have successfully found a root shell!
**![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfrWLAbdNU2jusFdoDGRSapfF5ARb967z-YlJnW_mUAy8TCdUERX0PYinr1LwNwsirYojF49k2VdohSfKKLvhBTDCLMTIc_I6dIdrD2M0iRYuXw0jb4hU_TicQ0UarIt8GqL0zY?key=M9UqJo6gFwpGru4IKynhiX5z)**


## Task 2
After enabling ASLR using the `sudo sysctl -w kernel.randomize_va_space=2`. I was able to gain a root shell by simply running the command until we hit the right address. This makes sense because while the address space is randomized, by brute forcing it we eventually hit the right address and executed the exploit.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250203123145.png)


## Task 3 
Without the `-fno-stack-protector` flag `GCC` is able to detect stack smashing which is what we are doing with our buffer overflow and then stop execution of the program. This is a good countermeasure to stop some of these attacks. It does this by inserting stack canaries to detect the stack smashing. 

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250203125425.png)


## Task 4
After creating a non executable stack, I am not able to get a shell. This makes sense, because our instead of running our shellcode it will raise a seg fault as the CPU is trying to execute code in a nonexecutable region of memory. This is perfect against shellcode injection attacks, but as the lab document states it still can be vulnerable.  

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250203131249.png)
