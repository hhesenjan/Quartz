---
title: "Windows Kernel Driver Handling Inputs"
created: 2026-03-17
modified: 2026-03-17
tags: ["C++", "WINDOWS"]
draft: false
---

## Introduction

I recently have been learning about programming kernel drivers to dig further into the Windows operating system. I have learnt a lot about Windows while building kernel drivers and I decided to share that knowledge by building a simple kernel driver which handles inputs from the user-mode process. Now let's get our hands dirty and start building a kernel driver.

## Symbolic Link, Device Object, and Driver Object

The device object and symbolic link enables the user-mode process to communicate with the kernel driver. And the driver object is responsible for handling requests from user-mode process and cleaning up the memory when the kernel driver is unloaded. It's crucial to understand that a user-mode process must create a handle using the symbolic link that will reference the device object to communicate with the kernel driver. 

## Kernel Driver

I'll be building a simple kernel driver which allows the user-mode process to append and remove data structures from a linked list inside of the kernel driver. The data structure will be used to store process names and in future chapters I'll be going through blocking process from execution depending on their process name. The source code for the kernel driver we are going to build is available at [Github](https://github.com/HusenjanDev/LKernel.git).

### IOCTL

The I/O Control Codes (IOCTL) in kernel driver is used to define the operations the kernel driver should perform when the IOCTL is called by the user-mode process using `DeviceIoControl()` function. The IOCTL is used with the combination of `IRP_MJ_DEVICE_CONTROL` to handle the custom requests.  

```cpp title="IOCTL"
#define DRIVER 0x8010
#define IOCTL_ADD_PROCESS CTL_CODE(DRIVER, 0x8001, METHOD_BUFFERED, FILE_ANY_ACCESS)
#define IOCTL_REMOVE_PROCESS CTL_CODE(DRIVER, 0x8002, METHOD_BUFFERED, FILE_ANY_ACCESS)
```

The driver is assigned the identifier number `0x8010` because the values `0x8000` to `0xFFFF` is reserved for vendors. Each IOCTL is assigned an identifier number and all of them are within the range reserved for vendors `0x800` to `0xFFF`.

### Data Structures

The most common approach to handle data in kernel drivers is by using a linked list as it allows us to append and remove data from the memory which is crucial while working with kernel drivers because if we fail to clean up the memory it could lead to failure in other components or bluescreen of death (BSOD).

```cpp
LIST_ENTRY g_ProcessListHead;

struct ProcessList {
	LIST_ENTRY ListEntry;
	UNICODE_STRING ProcessName;
};
```

The `g_ProcessListHead` is the head of the linked list and the `ProcessList` is the data structure which holds the data. The list links the `ListEntry` members together and the `CONTAINING_RECORD` is used to get the `ProcessList` data structure. It's important to note that since we are using `UNICODE_STRING` to handle process name we must allocate memory space for `ProcessName.Buffer` and deallocate the memory space when it's no longer needed.

### Predefined Functions

The predefined functions will be used for different purposes such as unloading driver, completing requests, adding processes, removing processes, and iterating through these processes.

```cpp title="Predefined Functions"
VOID DriverUnload(PDRIVER_OBJECT DriverObject);
NTSTATUS CompleteRequest(PIRP Irp, NTSTATUS status, ULONG_PTR info);
NTSTATUS DriverCreateClose(PDEVICE_OBJECT DeviceObject, PIRP Irp);
NTSTATUS InsertProcess(UNICODE_STRING ProcessName);
NTSTATUS DriverControl(PDEVICE_OBJECT DeviceObject, PIRP Irp);
NTSTATUS RemoveProcess(UNICODE_STRING ProcessName);
NTSTATUS IterateProcesses();
```

I'll be going deeper into these functions in the upcoming chapters so don't worry about their purposes right now. 

### Driver Entry

The `DriverEntry()` function is the entry point for the kernel driver and it's used for initializing symbolic link, device object, and handling the different requests using the driver object. The `DriverEntry()` can be viewed as `main()` entry point in user-mode applications.

```cpp title="DriverEntry"
extern "C" NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryKey) {
	UNREFERENCED_PARAMETER(DriverObject);
	UNREFERENCED_PARAMETER(RegistryKey);

	UNICODE_STRING symName		= RTL_CONSTANT_STRING(L"\\??\\LKernel");
	UNICODE_STRING devName		= RTL_CONSTANT_STRING(L"\\Device\\LKernel");
	PDEVICE_OBJECT deviceObject = NULL;
	NTSTATUS status				= NULL;

	status = IoCreateDevice(DriverObject, NULL, &devName, FILE_DEVICE_UNKNOWN, NULL, NULL, &deviceObject);

	if (!NT_SUCCESS(status)) {
		KdPrint(("[!] Failed to initialize deivce object.\n"));
		return status;
	}

	status = IoCreateSymbolicLink(&symName, &devName);

	if (!NT_SUCCESS(status)) {
		IoDeleteDevice(deviceObject);
		KdPrint(("[!] Failed to initialize symbolic link.\n"));
		return status;
	}

	DriverObject->DriverUnload							= DriverUnload;
	DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]	= DriverControl;
	DriverObject->MajorFunction[IRP_MJ_CREATE]			= DriverCreateClose;
	DriverObject->MajorFunction[IRP_MJ_READ]			= DriverCreateClose;
	DriverObject->MajorFunction[IRP_MJ_CLOSE]			= DriverCreateClose;

	InitializeListHead(&g_ProcessListHead);
	IterateProcesses();
	return STATUS_SUCCESS;
}
```

Inside the `DriverEntry()` the symbolic link and the device object is initialized and the driver object is used for handling different requests such as `IRP_MJ_CREATE` for creating handles, `IRP_MJ_CLOSE` to close the handle, `IRP_MJ_READ` to read data and the `IRP_MJ_DEVICE_CONTROL` to handle the different IOCTL requests.

### Complete Request

The `CompleteRequest()` is a wrapper function which completes the I/O Request Packet (IRP). It simplifies the process by setting specific status and information fields and then it calls `IoCompleteRequest` to complete the request. 

```cpp title="Complete Request"
NTSTATUS CompleteRequest(PIRP Irp, NTSTATUS status = STATUS_SUCCESS, NTSTATUS info = 0) {
	Irp->IoStatus.Status = status;
	Irp->IoStatus.Information = info;
	IoCompleteRequest(Irp, IO_NO_INCREMENT);
	return status;
}

NTSTATUS DriverCreateClose(PDEVICE_OBJECT DeviceObject, PIRP Irp) {
	UNREFERENCED_PARAMETER(DeviceObject);
	UNREFERENCED_PARAMETER(Irp);
	return CompleteRequest(Irp);
}
```

The `DriverCreateClose` uses the `CompleteRequest()` function to complete IRP requests for `IRP_MJ_CREATE`, `IRP_MJ_READ`, and `IRP_MJ_CLOSE` which are commonly used by kernel drivers to perform specific operations such as establishing a handle, releasing the handle, and reading data.

### Insert Process

The `InsertProcess()` function will append a new entry to the linked list using the `ProcessList` data structure. It will then allocate memory for the `ProcessName.Buffer` and set the `ProcessName.Length` and the `ProcessName.MaximumLength` to prevent memory corruption by preventing out-of-bounds reads and writes.

```c++ title="InsertProcess()"
NTSTATUS InsertProcess(UNICODE_STRING ProcessName) {
    ProcessList* entry = (ProcessList*)ExAllocatePool2(POOL_FLAG_PAGED, sizeof(ProcessList), 'c0rp');

    if (!entry) {
        KdPrint(("[!] Failed to allocate ProcessList in InsertProcess()\n"));
        return STATUS_INSUFFICIENT_RESOURCES;
    }

    entry->ProcessName.Buffer = (PWCH)ExAllocatePool2(POOL_FLAG_PAGED, ProcessName.Length, 'c0rp');
    
    if (!entry->ProcessName.Buffer) {
        ExFreePoolWithTag(entry, 'c0rp');
        KdPrint(("[!] Failed to allocate entry->ProcessName.Buffer in InsertProcess()\n"));
        return STATUS_INSUFFICIENT_RESOURCES;
    }
    
    entry->ProcessName.Length = ProcessName.Length;
    entry->ProcessName.MaximumLength = entry->ProcessName.Length;
	RtlCopyUnicodeString(&entry->ProcessName, &ProcessName);

    KdPrint(("[+] Adding - ProcessName: %ws\n", entry->ProcessName.Buffer));
    InsertTailList(&g_ProcessListHead, &entry->ListEntry);

    return STATUS_SUCCESS;
}
````

The `InsertTailList()` function is responsible for appending the object to the end of the linked list and once the function executes successfully it returns the `STATUS_SUCCESS` code.

### Remove Process

The `RemoveProcess()` function will remove a node from the linked list and delete the allocated data structure. The way it deletes the node is by going through all the nodes in the linked list and then compare the process names.

```c++ title="RemoveProcess()"
NTSTATUS RemoveProcess(UNICODE_STRING ProcessName) {
	LIST_ENTRY* e = g_ProcessListHead.Flink;

	while (e != &g_ProcessListHead) {
		ProcessList* entry = CONTAINING_RECORD(e, ProcessList, ListEntry);
		LIST_ENTRY* next = e->Flink;

		if (RtlCompareUnicodeString(&entry->ProcessName, &ProcessName, TRUE) == 0) {
			RemoveEntryList(e);

			KdPrint(("[-] RemoveProcess - ProcessName : %ws\n", entry->ProcessName.Buffer));

			if (entry->ProcessName.Buffer) {
				ExFreePoolWithTag(entry->ProcessName.Buffer, 'c0rp');
			}

			ExFreePoolWithTag(entry, 'c0rp');
		}

		e = next;
	}

	return STATUS_SUCCESS;
}
```

The `CONTAINING_RECORD()` function is responsible for getting the data structure that belongs to the node while the `RemoveEntry()` function is responsible for unlinking the node from the linked list. The `ExFreePoolWithTag()` function is used to deallocate the node and data structure. 

### Iterate Processes

The `IterateProcesses()` function will be used to go through the linked list and print out the process name. I mainly made this for debugging purposes as it will allow us to view the data inside of our linked list.

```c++ title="IterateProcess()"
NTSTATUS IterateProcesses() {
	LIST_ENTRY* e = g_ProcessListHead.Flink;

	while (e != &g_ProcessListHead) {
		ProcessList* entry = CONTAINING_RECORD(e, ProcessList, ListEntry);
		KdPrint(("[#] IterateProcess - Process Name: %ws\n", entry->ProcessName.Buffer));
		e = e->Flink;
	}

	return STATUS_SUCCESS;
}
```

### DeviceControl

The `DeviceControl()` function is responsible for handling the I/O Control Codes (IOCTL) requests. This function allows us to declare the way the kernel should respond to the different IOCTL requests.

```cpp title="DeviceControl()"
NTSTATUS DriverControl(PDEVICE_OBJECT DeviceObject, PIRP Irp) {
	UNREFERENCED_PARAMETER(DeviceObject);
	UNREFERENCED_PARAMETER(Irp);

	auto irp = IoGetCurrentIrpStackLocation(Irp);
	auto& dic = irp->Parameters.DeviceIoControl;

	switch (dic.IoControlCode) {
		case IOCTL_ADD_PROCESS: {
			if (dic.InputBufferLength == 0) {
				KdPrint(("[!] DriverControl->IOCTL_ADD_PROCESS : Invalid input buffer length\n"));
				return CompleteRequest(Irp, STATUS_INVALID_BUFFER_SIZE);
			}

			auto input = (WCHAR*)Irp->AssociatedIrp.SystemBuffer;

			if (input == NULL) {
				KdPrint(("[!] DriverControl->IOCTL_ADD_PROCES: The input data is invalid"));
				return CompleteRequest(Irp, STATUS_INSUFFICIENT_RESOURCES);
			}

			UNICODE_STRING ProcessName;
			ProcessName.Buffer = input;
			ProcessName.Length = (USHORT)dic.InputBufferLength;
			ProcessName.MaximumLength = ProcessName.Length;
			KdPrint(("[#] Reading Process Name (ADDING): %ws\n", ProcessName.Buffer));

			InsertProcess(ProcessName);
			return CompleteRequest(Irp);
		}
		case IOCTL_REMOVE_PROCESS: {
			if (dic.InputBufferLength == 0) {
				KdPrint(("[!] Driver->IOCTL_REMOVE_PROCESS : Invalid input buffer\n"));
				return CompleteRequest(Irp, STATUS_INVALID_BUFFER_SIZE);
			}

			auto input = (WCHAR*)Irp->AssociatedIrp.SystemBuffer;

			if (input == NULL) {
				KdPrint(("[!] Driver->IOCTL_REMOVE_PROCESS : The input data is invalid"));
				return CompleteRequest(Irp, STATUS_INSUFFICIENT_RESOURCES);
			}

			UNICODE_STRING ProcessName;
			ProcessName.Buffer = input;
			ProcessName.Length = (USHORT)dic.InputBufferLength;
			ProcessName.MaximumLength = ProcessName.Length;
			KdPrint(("[#] Reading Process Name (REMOVAL): %ws\n", ProcessName.Buffer));

			RemoveProcess(ProcessName);
			return CompleteRequest(Irp);
		}
	}

	return CompleteRequest(Irp);
}
```

When the user-mode process uses the `IOCTL_ADD_PROCESS` a node is added into the linked list and when the user-mode process uses `IOCTL_REMOVE_PROCESS` the node is removed from the linked list. 

### Driver Unloading

The `DriverUnload()` function is always executed when the driver is going to be unloaded because it's responsible for deallocating symbolic link, device object, and other memory pools which were in use by the kernel driver.

```cpp title="DriverUnload()"
VOID DriverUnload(PDRIVER_OBJECT DriverObject) {
	while (!IsListEmpty(&g_ProcessListHead)) {
		LIST_ENTRY* e = RemoveHeadList(&g_ProcessListHead);
		ProcessList* entry = CONTAINING_RECORD(e, ProcessList, ListEntry);
		KdPrint(("[-] Removing - ProcessName: %ws\n", entry->ProcessName.Buffer));

		if (entry->ProcessName.Buffer) {
			ExFreePoolWithTag(entry->ProcessName.Buffer, 'c0rp');
		}

		ExFreePoolWithTag(entry, 'c0rp');
	}

	UNICODE_STRING symName = RTL_CONSTANT_STRING(L"\\??\\LKernel");
	IoDeleteSymbolicLink(&symName);
	IoDeleteDevice(DriverObject->DeviceObject);
}
```

In the beginning of the code the nodes of the linked list is being deleted and then the `IoDeleteSymbolicLink()` function is called to delete the symbolic link and the `IoDeleteDevice()` is called to delete the device object.

## User-Mode Process

The user-mode process will communicate with the kernel driver by creating a handle and then use that handle with the `DeviceIoControl()` function using the IOCTL `IOCTL_ADD_PROCESS` to call the operation that will allow us to add a process name to the linked list.

```cpp title="main.cpp"
#include <Windows.h>
#include <iostream>
#include <vector>

#define DRIVER 0x8010
#define IOCTL_ADD_PROCESS CTL_CODE(DRIVER, 0x8001, METHOD_BUFFERED, FILE_ANY_ACCESS)

int main() {
	HANDLE hDevice = CreateFile(L"\\\\.\\LKernel", GENERIC_READ | GENERIC_WRITE, NULL, NULL, OPEN_EXISTING, NULL, NULL);

	if (hDevice == INVALID_HANDLE_VALUE) {
		printf("[!] hDevice is a invalid handle value.\n");
		return -1;
	}

	std::vector<std::wstring> m_Processes;

	m_Processes.push_back(L"brave.exe");
	m_Processes.push_back(L"cmd.exe");

	for (int i = 0; i < m_Processes.size(); ++i) {
		if (DeviceIoControl(hDevice, IOCTL_ADD_PROCESS, (LPVOID)m_Processes[i].c_str(), sizeof(WCHAR) * m_Processes[i].size(), NULL, NULL, NULL, NULL)) {
			printf("[+] Successfully sent %ws to kernel driver.\n", m_Processes[i].c_str());
		}
	}

	return 0;
}
```

The reason a vector with wstring is declared is because the kernel driver uses UNICODE strings otherwise we would need to convert all the ASCII characters to UNICODE characters which requires additional code.

## Conclusion

Our simple kernel driver can now handle input requests and perform specific operations requested by the user-mode process. Additionally, the kernel driver is also cleaning up the memory before unloading itself which helps with preventing memory corruption and BSOD. In future chapters I'll go through blocking processes from being executed using the process names from the linked list.