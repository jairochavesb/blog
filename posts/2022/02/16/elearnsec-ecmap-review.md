---
layout: default
title: Jairo's Blog
---

<center>
<img src="https://cdn-web-pro.elearnsecurity.com//assets/images/certification/ecmap_certificate_sm.png">
</center>
<h6>February 16<sup>th</sup>, 2022</h6>

# eLearnSecurity - Certified Malware Analysis Professional.

These has been days full of anxiety because of the waiting period, but today I was informed that I passed the certification exam! :)

In this post I want to share my experience with this certification exam and how I did prepare myself for the exam.

## Before the course.
My knowledge about malware was more like from a bird's eye than something detailed, it was very general. 

I had notion about the different malware categories like trojans, ransomware... but my understanding of the malware inner workings was zero, also I didn't have knowledge about reversing, assembly, windows APIs... etc.

When I started the course I already had experience in cybersecurity and I have been working on the field for a couple of years, this experience helped me to lay the foundations (By the way, experience in infosec is not needed to take the exam but it certainly helps).

## During the course.
The information provided in the course is excellent, very informative and with a lot of resources to investigate and extend your knowledge in a topic.

The main course goal is:
* Work with realistic malware samples created to prepare you for real-world samples
* Analyze real-world samples: ransomware, botnets, rats, etc.
* Explore an entire module dedicated to x64 bit assembly
* Dive into the TLS method
* Understand how malware uses Windows APIs to achieve their malicious activity
* Debug samples using different debuggers

The course have 30+ labs and 10 samples to practice reverse engineering, additionally it have slides with the theory and videos explaining key topics.

Below is the course syllabus:

### SECTION 1: MALWARE ANALYSIS
* Module 1: Introduction to Malware Analysis
* Module 2: Static Analysis Techniques
* Module 3: Assembly Crash Course
* Module 4: Behavior Analysis
* Module 5: Debugging and Disassembly Techniques
* Module 6: Obfuscation Techniques

### SECTION 2: REVERSE ENGINEERING
* Module 1: The Necessary Theory: Part 1
* Module 2: The Necessary Theory: Part 2
* Module 3: The Necessary Theory: Part 3
* Module 4: VA/RVA/OFFSET & PE File Format
* Module 5: String References & Basic Patching
* Module 6: Exploring the Stack
* Module 7: Algorithm Reversing
* Module 8: Windows Registry Manipulation
* Module 9: File Manipulation
* Module 10: Anti-Reversing: Part 1
* Module 11: Anti-Reversing: Part 2
* Module 12: Anti-Reversing: Part 3
* Module 13: Code Obfuscation
* Module 14: Analyzing Packers & Manual Unpacking
* Module 15: Debugging Multi-Thread Applications

If you want to look more details refer to this <a href="https://dsxte2q2nyjxs.cloudfront.net/Syllabus_MAPV1.pdf">link</a>.

You can rest assured that the course covers everything needed to pass the exam, but I strongly recommend to read the extra resources provided at the end of the slides as these will help you with further and useful information.

### After the course.
When I finished the course, I was satisfied with the content and it was enough to tackle the exam, but I was afraid I didn't have the right amount of practical experience even if I did all the labs, yeah, call me paranoid. I was assuming the exam had tricks not covered in the course, but this is not the case, once again, all the topics covered in the course are enough to pass the exam.

But if you are like me, a little bit paranoid person and if you want to go for the extra mile to learn and investigate more, I will recommend you the book: "Malware Analsysis and Detection Engineering", this book is a massive +900 pages Behemoth (I will review it in other post). This book covers also the same as the course did, and gave me extra interesting tips and topics. 

<center>
<img src="https://kbimages1-a.akamaihd.net/91771305-b4c1-463d-9ca0-00896f44ca33/1200/1200/False/malware-analysis-and-detection-engineering.jpg" width="450" height="600">
</center>


The next important thing is practice. Practice as much as you can so you can be confident and also perform the analysis tasks almost automatically.

What I did was to code the book samples into Golang (I also wanted to code and practice Go :P), code your own PoC, for example, I did investigate how to call APIs from Go and I implemented a tool to edit the Window's registry, I also coded a program to perform process injection; then I did analyze those programs using the methodology taught in the course and the book.

When you are more confortable with the analysis process, then go and dissect a real malware! Create your dedicated lab for it, I reused an old pc for my malware anlysis lab and created a dedicated isolated network away from my other devices.

### Before and during the exam.
Is up to you when to start, but I begun the exam a friday after work, so I had a lot of time for it. You will only have 7 days to complete the exam. I did frequent stops to stretch my back, make coffee, eat and play with my dogs to rest a little of the exam. Try to keep your mind clear, do not stress thinking about the end date, just focus on the challenge you have and enjoy it, the exam is an awesome challenge.

If you did the course labs then you will be ready for the exam, and if you did read the book and went the extra mile, then you will be 200% ready. 

Take screenshots, a lot of them, because if there is no evidence you did something, then it did not happen. eLearnSecurity recommends to use kali during the labs and exam, so I installed <a href="https://flameshot.org/">flameshot</a> to take the screenshots, this is an amazing tool that supports highlights, notes, colors... etc. 

Explain what you do clearly and concise, but do not over explain and do dozens of pages only on one aspect of the sample being analyzed. To take notes I recommend <a href="https://www.giuspen.com/cherrytree/">CherryTree</a>. These last 2 tools became now my favorites tools and I use those almost daily.

Regarding the reporting format, the course provides some samples, but you can use any template you want. 

### After the exam.
Be patient, they will send you a notification to your email once the exam results are ready, but this will take a couple of weeks, in the meantime try to focus on anything else, maybe start a new certification ;)

When eLearnSecurity informs you about your success, go and celebrate! You did study hard and deserves it.





