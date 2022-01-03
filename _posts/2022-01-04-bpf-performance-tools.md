---
layout:     post
title:      "BPF peformance tools review"
date:       2022-01-04 20:00:00 +0100
categories: review
---
# Book information
* Title: [BPF Performance Tools](https://www.brendangregg.com/bpf-performance-tools-book.html)
* Subtitle: Linux System and Application Observability
* Author: Brendan Gregg
* ISBN: 978-0136554820
* Date of publication: December 2019

# Target audience
The audience of this book are software professionals that are concerned about performance and troubleshoot.

# Brief summary
The aim of the book is to be a guide to system performance, introducing new methods and tools for doing analysis that leads to more robust, reliable, and safer code. It gives a high overview of the bcc toolkit, presents more than 100 bpftrace tools that can be used without any programming and it also explains bpftrace interfaces to create our own scripts. All these tools can be used to analyze any subsystem from CPUs and networking to memory, disks, file systems or the Linux kernel itself.

# Finding
The author is able to expose in a clear and simple way how to do performance analysis. The first chapters contain an introduction to the terminology, methods and tools used in the rest of the book. The intermediate chapters focus on different subsystems and how to measure their performance. This includes general purpose commands and specific tooling for the current subsystem. They all follow the same structure, so that you get a sense of familiarity with the underlying methodology. Moreover, if you just want to focus on a subsystem, you can read it directly and skip the previous chapters. Finally, the last section focuses on more advanced topics like some tips and tricks or the bpftrace cheat sheet.

# Strengths
* The figures used throughout the book help a lot to understand the context. Moreover, the figure on the cover contains a summary of all the tools that can be used to perform system performance analysis.
![BPF performance tools](/assets/figures/2022-01-04-bpf-performance-tools.png)
* The structure repetition of the chapters focused on different subsystems help the audience get a sense of familiarity with the underlying methodology.
* The same tools are presented several times in different chapter to solve different kind of problems.
* Several subsystems contain [optional exercises](https://github.com/ikerexxe/bpf_playground/tree/main/bpf_performance_tools_exercises) that help to understand how the tools are used.
* Performance analysis is under heavy development and the book contains references to web pages that include additional information.

# Limitations
* I've had some problems with the API that can be used with bpftrace. For several optional exercises I was unable to find the related probe as they don't contain any type of documentation. This was the case for the kernel and bash shell sections. Maybe this is because my knowledge in these areas is low, but I definitely think that some documentation for the probes would be really helpful.
* The explanation on how to load the debuginfo for system applications that lack the stack traces is poor. Even though I installed the additional packages bpftrace was unable to show the complete stack trace.

# Rating
## **4/5**
