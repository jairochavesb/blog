---
layout: default
title: Jairo's Blog
---

<center>
<img src="https://upload.wikimedia.org/wikipedia/commons/5/53/Go_gopher_five_years.jpg">
<h6> Gopher image by Renee French, licensed under Creative Commons 3.0 Attributions license. </h6>
</center>
<h6>December 31<sup>st</sup>, 2021</h6>

## Crackmes.one - Challenge by Disip.

Due the holidays I haven't dedicate the time to write here, so I will write about a quick crackme challenge.

The objective of this challenge is to reverse engineer the program and create a serial generator to generate valid passwords based
on a username.

Ok, let's get to work on it.

Running the program for the first time we can see a small dialog box as follows:

<img src="http://127.0.0.1:4000/images/cracking-challenge-by-disip/img001.png">

Providing a random text will generate an error (of course! we don't know the correct serial).

<img src="http://127.0.0.1:4000/images/cracking-challenge-by-disip/img002.png">

I will assume the code in charge to process the user input could be close this messagebox call.
Let's check the code in x64dbg.

I went to the program entry point and searched for referenced strings.

<img src="http://127.0.0.1:4000/images/cracking-challenge-by-disip/img003.png">

Nice, the error messagebox text was located easily. I clicked on it to go to the disassembly code using this string is located.

<img src="http://127.0.0.1:4000/images/cracking-challenge-by-disip/img004.png">

As seen in the last image, the message for the good serial is also there, and scrolling a little bit up I saw the API calls to get the text boxes data and the code to generate the serial based on the name.

The code that is of our interest is the following:

<img src="http://127.0.0.1:4000/images/cracking-challenge-by-disip/img005.png">

I will explain each part briefly.

Below the second breakpoint at address <b>00401109</b> start a series of validations to check if the provided serial is correct, and also one character of the username is moved into bl.

> mv bl, byte ptr ds:[eax+403014]

Then it checks that any of the characters in the name are not 'Z', 'z' or '9', in this case any character stored in bl, if these characters are found, then these are converted to the previous char in the alphabet/number. 

For example, if the username contains a 'Z', this letter will be converted to 'Y', if it is a '9' then it will become an '8' and so on, this will be done with the <b>dec bl</b> instructions.

In this example, eax is acting as a counter, in other words, as per every username character checked, eax will increment by 1, and on every loop, eax then added <b>61</b>, for example:

> Loop 0: eax = 0; eax + 61 = 61
>
> Loop 2: eax = 1; eax + 61 = 62
>
> Loop 3: eax = 2; eax + 61 = 63

Now on address <b>1040112A</b>, we can consider that as a concatenation, the value stored in eax (a counter value + 61) is moved to bh, and in bl we have one of the username chars, let's check the next example:

> eax = 0
>
> eax + 61
>
> mv bh,al	//move 61 to bh

So, this will result in something like 616A (aj in ascii) in the first loop, then some lines below, the bl register will be incremented by 1 making the 'j' char to be a 'k', leaving the register with the hex value of 616B (ak in ascii).

Then, the program takes 2 bytes from the serial and compares it with the 616B value, so if I provide a serial of '1234567890', these 2 bytes, in the first loop will be 3132 (1 and 2 in ascii).

Because this comparison fails, the program will display the wrong serial message.

Ok, having figured out what happens in the first loop, I can code a small program to generate a valid serial from a username, what we saw until here, will be reapeted on every username character.

Here the serial generator in Go.

```golang
package main

import (
	"fmt"
	"os"
)

func main() {
	// Checks if there is an username provided as argument
	if len(os.Args) < 2 {
		fmt.Println("Provide a username.")
		os.Exit(1)
	}

	var username string

	// Here it checks that any of the username characters are not 'Z', 'z' or '9' and
	// if these exist, then they will be replaced by the previous one.
	for _, v := range os.Args[1] {
		if v == 'Z' || v == 'z' || v == '9' {
			username += string(v - 1)
		} else {
			username += string(v)
		}
	}

	// Iterates over the username letters and performs the respective additions
	// Because of little endianess, I had to swap values c2 and c to get the
	// correct serial output (in the debugger we had those inverted).
	for i, v := range []rune(username) {
		c := rune(i) + 0x61
		c2 := rune(v) + 1

		fmt.Printf("%c%c", c2, c)
	}
}

``` 

Let's test if this works...

<img src="http://127.0.0.1:4000/images/cracking-challenge-by-disip/img006.png">

Awesome! It generates valid serials!

By the way, I found that usernames longer than 5 don't work. So the cracking challenge will only work with usernames between 3 and 5, no more, no less...

Thanks for reading!






