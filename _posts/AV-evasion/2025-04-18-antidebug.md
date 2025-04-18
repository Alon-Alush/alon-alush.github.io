---
title: "AV Evasion: Anti-debug tricks"
classes: wide
header:
  teaser: /assets/images/evadingavs/anti-debug/image.jpg
ribbon: DodgerBlue
description: "Learn how malware evades anti-viruses (includes code examples)"
categories:
  - AV-evasion
toc: true
---

# Reducing AV detections

One of the most common techniques malware authors use to create **FUD** (Fully Undetectable) malware is **anti-debugging**. 

Practically *every* anti-virus solution **opens the target executable with *some* form of debugging or inspection enabled**— Sometimes obvious, sometimes very stealthy.

And I haven't heard of any reverse-engineer / malware-analyzer who doesn't have x64dbg or practically any debugging tool lying around 😅
Surprisingly,  believe it or not: it's often the *least* suspicious reactions (like a program immediately exiting without modifying anything), that trigger antivirus detection the most. 

Because of this, **instead of simply terminating when a debugger is detected**, it's usually better to **reroute execution and behave like an entirely *different* (but stil "normal-looking"**) program.

For example: **instead of exiting** the program when a debugger is detected, **route** execution to an entirely different "legit" routine that acts as a random number generator, for example, and *only then* close the program.

![Video results for searching "FUD Crypter" on Youtube](/assets/images/evadingavs/videos.png)

![FUD crypter marketplace on Telegram](/assets/images/evadingavs/marketplace.png)

And here's an example of how one of these FUD crypters look like:

![Video results for searching "FUD Crypter" on Youtube](/assets/images/evadingavs/crypter.png)

# How do they do this?

It's a combination of many different techniques. In this post, we'll cover the most basic ones:

- Custom WinAPI function implementations

- String Encryption

- Packing

Let's start with custom WinAPI function implementations:

As you may know, AVs look for programs that use "suspicious" WinAPI functions. We're talking `GetModuleHandle`, `GetProcAddress`, `VirtualAlloc`, etc.

Of course, these functions have legitimate uses, **but they're frequently used by malware authors**.

# The solution

Instead of directly calling a CRT function like `VirtualAlloc`, malware authors will create their own "custom implementations" of these functions.

For example, instead of calling `GetModuleHandleW` directly from the CRT (which is *hooked to death* by AV solutions):

![Direct WinAPI function usage](/assets/images/evadingavs/crtfunction.png)

They will create a custom function (`myGetModuleHandle`) that is meant to **functionally replicate** the original `GetModuleHandleW`, so that, instead of calling `GetModuleHandleW`, we will call `myGetModuleHandle` which looks a lot less suspicious, since it looks like a normal function in our code.

Here's an example of a custom `myGetModuleHandle` implementation that I used in my custom .exe packer:

![Custom WinAPI function implementation](/assets/images/evadingavs/customfunction.png)

As you can see, it manually walks over the linked list of modules (inside the PEB), copies the target module name (`ModuleName`) and the current module's full path (`FullDllName`) into buffers, and checks if the current module name contains (or matches) the target name (case-insensitive partial match).

If it matches, we return the base address (DllBase) of the loaded module:
```c
return (HMODULE)pEntry->DllBase;
```
If it walks through all modules without a match:
```
return NULL;
```

We can then call this custom function in our code. So, instead of doing:
```c
GetModuleHandleW(L"ntdll.dll") // ❌ <-- Hooked!
```
We will call our custom function:
```c
myGetModuleHandleW(L"ntdll.dll") // ✅ <-- Looks like a normal function!
```
For my custom packer, I implemented these custom functions: `myVirtualAlloc`, `MyLoadLibrary`, `myGetModuleHandle`, `myGetProcAddress`.

Below is the code for them:

Custom `GetProcAddress`:
```c
FARPROC myGetProcAddress(HMODULE hModule, LPCSTR lpProcName) {
	PIMAGE_DOS_HEADER dosHeader = (PIMAGE_DOS_HEADER)hModule;
	PIMAGE_NT_HEADERS64 ntHeaders = (PIMAGE_NT_HEADERS64)((BYTE*)hModule + dosHeader->e_lfanew);
	PIMAGE_EXPORT_DIRECTORY exportDirectory = (PIMAGE_EXPORT_DIRECTORY)((BYTE*)hModule + ntHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);

	DWORD* addressOfFunctions = (DWORD*)((BYTE*)hModule + exportDirectory->AddressOfFunctions);
	WORD* addressOfNameOrdinals = (WORD*)((BYTE*)hModule + exportDirectory->AddressOfNameOrdinals);
	DWORD* addressOfNames = (DWORD*)((BYTE*)hModule + exportDirectory->AddressOfNames);
	for (DWORD i = 0; i < exportDirectory->NumberOfNames; i++) {
		if (strcmp(lpProcName, (const char*)hModule + addressOfNames[i]) == 0) {
			return (FARPROC)((BYTE*)hModule + addressOfFunctions[addressOfNameOrdinals[i]]);
		}
	}
	return NULL;

}
```

Custom `GetModuleHandle`:

```c
HMODULE myGetModuleHandle(LPCWSTR ModuleName) {
	PEB* pPeb = (PEB*)__readgsqword(0x60);
	PEB_LDR_DATA* Ldr = pPeb->Ldr;
	LIST_ENTRY* ModuleList = &Ldr->InMemoryOrderModuleList;
	LIST_ENTRY* pStartListEntry = ModuleList->Flink;
	WCHAR inputModule[MAX_PATH] = { 0 };
	WCHAR targetModule[MAX_PATH] = { 0 };
	for (LIST_ENTRY* pListEntry = pStartListEntry; pListEntry != ModuleList; pListEntry = pListEntry->Flink) {
		
		LDR_DATA_TABLE_ENTRY* pEntry = (LDR_DATA_TABLE_ENTRY*)((BYTE*)pListEntry - sizeof(LIST_ENTRY));
		wcscpy_s(targetModule, MAX_PATH, ModuleName);
		wcscpy_s(inputModule, MAX_PATH, pEntry->FullDllName.Buffer);

		if (StrStrW(inputModule, targetModule) != NULL) {
			return (HMODULE)pEntry->DllBase;
		}
	}
	return NULL;
}
```

Custom `LoadLibrary`:
```c
typedef NTSTATUS(NTAPI* pLdrLoadDll) (
	PWCHAR PathToFile,
	ULONG Flags,
	PUNICODE_STRING ModuleFileName,
	PHANDLE ModuleHandle
	);
HMODULE MyLoadLibrary(LPCWSTR lpFileName) {
	UNICODE_STRING ustrModule;
	HANDLE hModule = NULL;

	HMODULE hNtdll = myGetModuleHandle((LPCWSTR)L"ntdll.dll");
	pRtlInitUnicodeString RtlInitUnicodeString = (pRtlInitUnicodeString)myGetProcAddress(hNtdll, "RtlInitUnicodeString");

	RtlInitUnicodeString(&ustrModule, lpFileName);

	pLdrLoadDll myLdrLoadDll = (pLdrLoadDll)myGetProcAddress(myGetModuleHandle(L"ntdll.dll"), "LdrLoadDll");
	if (!myLdrLoadDll) {
		return NULL;
	}

	NTSTATUS status = myLdrLoadDll(NULL, 0, &ustrModule, &hModule);
	return (HMODULE)hModule;
}
```

Custom `VirtualAlloc`:

```c
typedef NTSTATUS(NTAPI* pNtAllocateVirtualMemory)(
	HANDLE ProcessHandle,
	PVOID* BaseAddress,
	ULONG_PTR ZeroBits,
	PSIZE_T RegionSize,
	ULONG AllocationType,
	ULONG Protect
	);

LPVOID myVirtualAlloc(SIZE_T size, DWORD allocationType, DWORD protect) {
	HMODULE ntdll = myGetModuleHandle((LPCWSTR)L"ntdll.dll");
	if (ntdll == NULL) {
		printf("Failed to get address of NtAllocateVirtualMemory\n");
		return NULL;
	}
	pNtAllocateVirtualMemory myAllocateVirtualMemory = (pNtAllocateVirtualMemory)myGetProcAddress(ntdll, "NtAllocateVirtualMemory");
	if (!myAllocateVirtualMemory) {
		printf("Failed to get address of NtAllocateVirtualMemory\n");
		return NULL;
	}
	PVOID baseAddress = NULL;
	SIZE_T regionSize = size;

	NTSTATUS status = myAllocateVirtualMemory(
		GetCurrentProcess(),
		&baseAddress,
		0,
		&regionSize,
		allocationType,
		protect
	);
	return baseAddress;

}
```

Combined with some other tricks, these custom functions reduced the detections of my packer to around **5 detections** on VirusTotal:

![Direct WinAPI function usage](/assets/images/evadingavs/detections.png)

Before the custom functions, my packer was at around **19 detections**.









