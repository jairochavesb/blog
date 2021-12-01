---
layout: default
title: Jairo's Blog
---

<center>
<img src="https://github.com/jairochavesb/blog/images/golangLogoGopher.png" style="width:300px;height:150px;">
</center>
<br>
<h6>November 29<sup>th</sup>, 2021</h6>
<h2>Calling Windows APIs with Go</h2>
I took the decision to learn a new programming language, after looking for a while I did choose Go.

In order to practice this new languge I tried to write a simple malware and this exercise did lead
me to wonder... how I can call Windows APIs with Go?

I did a little research and found how to do it, let's see!

The package that will do the magic is the <a href="https://pkg.go.dev/golang.org/x/sys/windows">windows</a> package and can be added to your project with the following command:

<pre><code>go get golang.org/x/sys/windows</code></pre>
<br>
<h4>Example #1. Calling IsDebuggerPresent.</h4>

```go
package main

import (
   "fmt"
   "syscall"
   "unsafe"

   "golang.org/x/sys/windows"
)

func main() {

   fmt.Println("Calling IsDebuggerPresent from kernel32.dll")
   kernel32DLL := windows.NewLazyDLL("kernel32.dll")

   isDebuggerPresent := kernel32DLL.NewProc("IsDebuggerPresent")

   debuggerPresent, _, _ := isDebuggerPresent.Call()
   fmt.Println("Result: ", debuggerPresent)
}
```

In this example, the API will return a 0 because it is not running in a debugger, otherwise the
output will be 1.
<br>
<br>
<h4>Example #2. Calling MessageBoxW.</h4>

```go
package main

import (
   "fmt"
   "syscall"
   "unsafe"

   "golang.org/x/sys/windows"
)

func main() {
   fmt.Println("Calling MessageBoxW from user32.dll")

   user32dll := windows.NewLazyDLL("user32.dll")
   messageBoxW := user32dll.NewProc("MessageBoxW")

   showMessageBox, _, _ := messageBoxW.Call(
      uintptr(0),
      uintptr(unsafe.Pointer(syscall.StringToUTF16Ptr("I am a message box! :)"))),
      uintptr(unsafe.Pointer(syscall.StringToUTF16Ptr("Hi!"))),
      uintptr(0x00000000),
   )
}
```

This one is a little bit complicated because the MessageBoxW API will need at least 3 arguments.

As it can be observed, calling Windows APIs from Go is not difficult, maybe the most tedious task is to
read the API documentation to check which arguments it will use.

Thanks for reading!







[back](https://github.com/jairochavesb/blog/)
