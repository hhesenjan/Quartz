---
title: "C++ Hooking Methods"
created: 2023-12-27
modified: 2023-12-27
tags: ["CPP", "Hooking Methods"]
draft: false
---

## Introduction

Hooking in C++ is a technique that involves changing the program's control flow by patching instruction bytes. Instead of executing the original function it will execute the hook function which is made by us. The technique is commonly used by anti-virus, malware's, and sandboxes for either detection purpose or illegal purposes.

## Detour Hooking

```plaintext title="Detour Hooking Graph"
                                                                    +-----------------------+
+------------------+       +--------+      +-----------------+      | MessageBoxA_Address:  |
| call MessageBoxA |------>| Hooked |----->| jmp MessageBoxA |----->| mov edi, edi          |
+------------------+       +--------+ No   +-----------------+      | push ebp              |
                                | Yes                               | mov esp, ebp          |
                                |                                   | more...               |
                                |                                   +-----------------------+
                                v
                     +------------------------+       +-----------------------------+
                     | jmp Hooked_MessageBoxA |---+   | Hooked_MessageBoxA:         |
                     +------------------------+   +-->| push ebp                    |
                                                      | mov ebp, esp                |
                                                      | executes our custom code... |
                                                      | ret                         |
                                                      +-----------------------------+
```

Detour Hooking is a technique that changes the jump instruction to jump to the hook function instead of the original function.

```cpp title="Detour Hooking Function"
bool detour_hook(void* source, void* destination, intptr_t length)
{
	// Don't continue if length is less than 5
	if (length < 5)
	{
		return false;
	}
	
	// Will be used to store previous permission
	DWORD old_protect = NULL;
	
	// Calculating relative offset of destination
	DWORD relative_offset = ((DWORD)destination - (DWORD)source) - 5;
	
	// Changing permission of source
	VirtualProtect(source, length, PAGE_EXECUTE_READWRITE, &old_protect);
	
	*(BYTE*)(source) = 0xE9; // The 0xE9 equals JMP
	*(DWORD*)((DWORD)source + 1) = relative_offset; // The relative_offset eqals destination
	
	// Restoring to previous permission
	VirtualProtect(source, length, old_protect, &old_protect);
	
	// Return true if executed properly
	return true;
}
```

If the `length` parameter is less than 5 than the call is returned since detour hooking method requires at least 5 bytes because jump instruction. The formula for calculating the relative offset is `destination - source - 5` and that will be used to jump to the destination where the hook function is at. The `source` where the original function is at can be modified because `VirtualProtect()` function changes the address space permission to read, write, and execute. Once the necessary bytes has been patched the address space permission can be reverted as it's stored on `old_protect`.

In simpler terms the detour hooking method replaces `jmp foo` with `jmp hooked_foo` so it executes the hooked function instead of the original one. Here's a overview of the whole detour hook code:

```cpp title="Detour Example"
bool detour_hook(void* source, void* destination, intptr_t length)
{
	// Don't continue if length is less than 5
	if (length < 5)
	{
		return false;
	}
	
	// Will be used to store previous permission
	DWORD old_protect = NULL;
	
	// Calculating relative offset of destination
	DWORD jump_addr = ((DWORD)destination - (DWORD)source) - 5;
	
	// Changing permission of source
	VirtualProtect(source, length, PAGE_EXECUTE_READWRITE, &old_protect);
	
	*(BYTE*)(source) = 0xE9; // The 0xE9 equals JMP
	*(DWORD*)((DWORD)source + 1) = jump_addr; // The jump_addr eqals destination
	
	// Restoring to previous permission
	VirtualProtect(source, length, old_protect, &old_protect);
	
	// Return true if executed properly
	return true;
}

// Original function
int __stdcall foo(int number_one)
{
	printf("The magical number: %d\n", number_one);
	return 0;
}

// Hook function
int __stdcall hooked_foo(int number_one)
{
	printf("The non-magical number: %d", number_one);
	return 0;
}

int main()
{
	// If hooking was successful
	if (detour_hook(foo, hooked_foo, 5) == true)
	{
		foo(10);
	}
	// If hooking was unsuccessful
	else {
		printf("detour_hook: failed\n");
	}
	return 0;
}
```

```cmd
The non-magical number: 10
```

What does all that code do? All it does is it replaces `jmp foo` with `jmp hooked_foo` and if the detour hook was successful the `foo()` function is called but the `hooked_foo()` function will be executed instead. I recommend trying out detour hooking on your own environment by attaching a debugger onto it as it will help you to understand the code better.

## Inline Hooking

```text title="Inline Hooking Graph"
                                                    Original Function
                                           +--------------------------------+
                                           | MessageBoxA_Address:           |
+------------------+      +--------+    +->| mov edi, edi                   |
| call MessageBoxA |----->| Hooked |----+  | push ebp                       |
+------------------+      +--------+ No    | mov ebp, esp                   |
                               | Yes       | cmp dword ptr fs[00000000], 0  |
                               |           | more...                        |
                               |           +--------------------------------+
                               |
                               |
                               v                          +------------------------------+ 
                +--------------------------------+    +-->| hooked_messageboxa_address:  |
                | MessageBoxA_Address:           |  +-+   | push ebp                     |
                | jmp hooked_messageboxa         |--+     | mov esp, ebp                 | 
Continue  +---->| cmp dword ptr fs[00000000], 0  |        | executes our custom code...  |
Execution |     | more...                        |        | jmp trampoline_addr          |---------+ 
          |     +--------------------------------+        | ret                          |         |
          |                                               +------------------------------+         |
          |                                                                                        |
          |                                                                                        |
          |                                                                                        |
          |                                                                                        |
          |                                                    +------------------------------+    |
          |                                                    | trampoline_addr_address:     |    |
          |                                                    | mov edi, edi                 |<---+
          |                                                    | push ebp                     |
          |                                                    | mov esp, ebp                 |
          +----------------------------------------------------| jmp MessageBoxA              |
                                                               +------------------------------+
```

Inline Hooking (Trampoline Hooking) is a technique which copies the necessary bytes to a memory location before the bytes are replaced with a jump instruction which jumps to hook function. Inside the memory location a jump instruction is placed to jump to the original function to continue the execution flow.

```cpp title="Inline Hooking Function"
LPVOID trampoline_hook(void* source, void* destination, intptr_t length)
{
	// Don't continue if length is less than 5
	if (length < 5)
	{
		return 0;
	}
	
	// Trampoline assembly code
	CHAR trampoline[20] = {};

	// Allocating memory space
	LPVOID trampoline_addr = VirtualAlloc(NULL, 20, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
	
	// Copying bytes from source
	memcpy(trampoline, source, length);
	
	// Calculating original function address
	DWORD original_func_addr = (DWORD)((DWORD)source - (DWORD)trampoline_addr - length);
	
	*(BYTE*)(trampoline + length) = 0xE9;
	*(DWORD*)((DWORD)trampoline + length + 1) = original_func_addr;

	// Writes bytes from trampoline to trampoline_addr
	WriteProcessMemory(GetCurrentProcess(), trampoline_addr, trampoline, 20, NULL);
	
	// Will be used to store previous permission
	DWORD old_protect = NULL;
	
	// Calculating relative offset of destination
	DWORD jump_addr = ((DWORD)destination - (DWORD)source) - 5;
	
	// Changing permission of source
	VirtualProtect(source, length, PAGE_EXECUTE_READWRITE, &old_protect);
	
	*(BYTE*)(source) = 0xE9; // The 0xE9 equals JMP
	*(DWORD*)((DWORD)source + 1) = jump_addr; // The jump_addr eqals destination
	
	// Restoring to previous permission
	VirtualProtect(source, length, old_protect, &old_protect);
	
	// Return true if executed properly
	return trampoline_addr;
}
```

The `trampoline` array will be used to store the bytes and the bytes from original function is copied using `memcpy()` function. A jump instruction is added onto the array and the relative offset is calculated using the formula `source - trampoline_addr - length`. The `WriteProcessMemory` function is used to transfer the bytes from `trampoline` to `trampoline_addr` and once the function exits the `trampoline_addr` address is returned which will be used by us to continue the execution flow. The rest of the function is using detour hooking method to jump to hook function. Here's a overview of Inline Hooking being used on `MessageBoxA()` function.

```cpp title="Inline Hooking Example"
#include <iostream>
#include <Windows.h>

LPVOID trampoline_hook(void* source, void* destination, intptr_t length)
{
	// Don't continue if length is less than 5
	if (length < 5)
	{
		return 0;
	}
	
	// Trampoline assembly code
	CHAR trampoline[20] = {};

	// Allocating memory space
	LPVOID trampoline_addr = VirtualAlloc(NULL, 20, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

	// Calculating original function address
	DWORD original_func_addr = (DWORD)((DWORD)source - (DWORD)trampoline_addr - length);

	// Copying bytes from source
	memcpy(trampoline, source, length);

	*(BYTE*)(trampoline + length) = 0xE9;
	*(DWORD*)((DWORD)trampoline + length + 1) = original_func_addr;

	// Writes bytes from trampoline to trampoline_addr
	WriteProcessMemory(GetCurrentProcess(), trampoline_addr, trampoline, 20, NULL);
	
	// Will be used to store previous permission
	DWORD old_protect = NULL;

	// Calculating relative offset of destination
	DWORD jump_addr = ((DWORD)destination - (DWORD)source) - 5;

	// Changing permission of source
	VirtualProtect(source, length, PAGE_EXECUTE_READWRITE, &old_protect);

	*(BYTE*)(source) = 0xE9; // The 0xE9 equals JMP
	*(DWORD*)((DWORD)source + 1) = jump_addr; // The jump_addr eqals destination

	// Restoring to previous permission
	VirtualProtect(source, length, old_protect, &old_protect);

	// Return true if executed properly
	return trampoline_addr;
}

// Will be used to call original function
typedef int(__stdcall* t_orig_messageboxa)(HWND hwnd, LPCSTR lp_text, LPCSTR lp_caption, UINT u_type);
t_orig_messageboxa messageboxa_trampoline = nullptr;

// Hooked function
int __stdcall hooked_messageboxa(HWND hwnd, LPCSTR lp_text, LPCSTR lp_caption, UINT u_type)
{
	lp_text    = "Hooked!";
	lp_caption = "Hooked!";
	return messageboxa_trampoline(hwnd, lp_text, lp_caption, u_type);
}

int main()
{
	// Getting address of MessageBoxA to replace it with our hooked_messageboxa
	messageboxa_trampoline = (t_orig_messageboxa)trampoline_hook(GetProcAddress(GetModuleHandleA("user32.dll"), "MessageBoxA"), hooked_messageboxa, 5);

	// Calling function MessageBoxA
	MessageBoxA(NULL, "Hi", "Hi", MB_OK);

	// Returning
	return 0;
}
```

The reason `messageboxa_trampoline()` is called inside `hooked_messageboxa()` function because it will continue the execution flow to avoid potential crashes. Inside the `hooked_messageboxa()` function the `lp_text` and `lp_caption` will be changed to *"Hooked"* that shows that the Inline Hooking was successful.

## VMT Hooking

```text title="VMT Hooking Graph"
+---------+      +--------+      +--------------------+
| Class A |----->| Hooked |----->| VTable for Class A |      +---------------------------+
+---------+      +--------+ No   +--------------------+      | function_one_addr:        |
                     | Yes       | function_one       |----->| push ebp                  |
                     |           +--------------------+      | mov esp, ebp              |
                     |           | function_two       |      | sub esp, 40               |
                     |           +--------------------+      | executes original code... |
                     |           | function_three     |      | ret                       |
                     |           +--------------------+      +---------------------------+
                     v 
        +---------------------------+
        | Hooked VTable for Class A |      +-----------------------------+     +-----------------------------+
        +---------------------------+      | hooked_function_one_addr:   |     | hooked_function_three_addr: |
        | hooked_function_one       |----->| push ebp                    |  +->| push ebp                    |
        +---------------------------+      | mov esp, ebp                |  |  | mov ebp, esp                |
        | function_two              |--+   | sub esp, 10                 |  |  | sub esp, 5                  |
        +---------------------------+  |   | executes our custom code... |  |  | executes our custom code... |
        | hooked_function_three     |--++  | ret                         |  |  | ret                         |
        +---------------------------+  |   +-----------------------------+  |  +-----------------------------+
                                       +------------------------------------+
                                       |   
                                       |  +-------------------------------+
                                       +->| function_two_addr:            |
                                          | push ebp                      |
                                          | mov ebp, esp                  |
                                          | sub esp, 10                   |
                                          | executes the original code... |
                                          | ret                           |
                                          +-------------------------------+ 
```

VMT Hooking is a technique that hooks functions in classes but the functions are required to be `virtual` state for it to be hooked. If the function is not in a `virtual` state than the function cannot be hooked since the Virtual Table (VTable) will not be created. VMT Hooking is a useful technique for hooking game functions in classes since the functions are usually in a virtual state.

```cpp title="VMT Hook Class"
class vmt_hook {
private:
	std::unique_ptr<uintptr_t[]> vt_new		  = NULL;
	uintptr_t**					 base_class   = NULL;
	uintptr_t*                   original_vft = NULL;
	int							 nums_funcs   = NULL;
public:
	vmt_hook(void** base_class)
	{
		// The this->base_class points to virtual table
		this->base_class = reinterpret_cast<uintptr_t**>(base_class);
		
		// Counting total amount of virutal functions in a class
		while (reinterpret_cast<uintptr_t*>(this->base_class)[this->nums_funcs])
		{
			++this->nums_funcs;
		}

		// Calculating the size of the total functions
		unsigned int table_size = (this->nums_funcs * 4);

		// The this->original_vft points to this->base_class
		this->original_vft = *this->base_class;
		
		// Allocating space in unique_ptr
		vt_new = std::make_unique<uintptr_t[]>(this->nums_funcs);

		// Copying the virtual table over to vt_new
		memcpy(vt_new.get(), this->original_vft, table_size);

		// The virtual table is replaced with vt_new
		*base_class = vt_new.get();
	}

	// Hooks an function
	bool hook(void* new_fn, int index)
	{
		vt_new[index] = reinterpret_cast<uintptr_t>(new_fn);
		return false;
	}

	// Unhooks a specific function
	bool unhook(int index)
	{
		if (vt_new[index + 1] != NULL)
		{
			vt_new[index + 1] = *this->base_class[index];
			return true;
		}
		return false;
	}

	// Unhooks all the functions
	bool unhook_all()
	{
		if (*this->base_class != NULL)
		{
			*this->base_class = this->original_vft;
			return true;
		}
		return false;
	}

	// Destructor
	~vmt_hook()
	{
		if (*this->base_class != NULL)
		{
			*this->base_class = this->original_vft;
		}
	}
};
```

The constructor will use `vt_new` to replace the Virtual Table Pointer for `class_a` and the destructor will restore the Virtual Table Poiinter to to the original address. The `hook()` function will hook the function and the `unhook()` function will unhook a specific function and the `unhook_all()` function will unhook all the functions. Here's a example of VMT Hooking class in use:

```cpp title="VMT Hooking Example"
#include <iostream>
#include <Windows.h>

// VMT Hook Class
class vmt_hook {
private:
	std::unique_ptr<uintptr_t[]> vt_new		  = NULL;
	uintptr_t**					 base_class   = NULL;
	uintptr_t*                   original_vft = NULL;
	int							 nums_funcs   = NULL;
public:
	vmt_hook(void** base_class)
	{
		// The this->base_class points to virtual table
		this->base_class = reinterpret_cast<uintptr_t**>(base_class);
		
		// Counting total amount of virutal functions in a class
		while (reinterpret_cast<uintptr_t*>(this->base_class)[this->nums_funcs])
		{
			++this->nums_funcs;
		}

		// Calculating the size of the total functions
		unsigned int table_size = (this->nums_funcs * 4);

		// The this->original_vft points to this->base_class
		this->original_vft = *this->base_class;
		
		// Allocating space in unique_ptr
		vt_new = std::make_unique<uintptr_t[]>(this->nums_funcs);

		// Copying the virtual table over to vt_new
		memcpy(vt_new.get(), this->original_vft, table_size);

		// The virtual table is replaced with vt_new
		*base_class = vt_new.get();
	}

	// Hooks an function
	bool hook(void* new_fn, int index)
	{
		vt_new[index] = reinterpret_cast<uintptr_t>(new_fn);
		return false;
	}

	// Unhooks a specific function
	bool unhook(int index)
	{
		if (vt_new[index + 1] != NULL)
		{
			vt_new[index + 1] = *this->base_class[index];
			return true;
		}
		return false;
	}

	// Unhooks all the functions
	bool unhook_all()
	{
		if (*this->base_class != NULL)
		{
			*this->base_class = this->original_vft;
			return true;
		}
		return false;
	}

	// Destructor
	~vmt_hook()
	{
		if (*this->base_class != NULL)
		{
			*this->base_class = this->original_vft;
		}
	}
};

// Class A
class a {
public:
	virtual void function_one()
	{
		printf("a::function_one()\n");
		return;
	}
	void function_two()
	{
		printf("a::function_two()\n");
		return;
	}
	virtual void function_three()
	{
		printf("a::function_three()\n");
		return;
	}
};

// The function which will be used to replace virtual functions
void hooked_function()
	{
	printf("hooked_function() called\n");
}

int main()
{ 
	// Creating classes
	a* class_a = new a();
	vmt_hook* vmt = new vmt_hook((void**)class_a);

	// Hooking a::function_one and a::function_three
	printf("--Hooks Functions---\n");
	vmt->hook(hooked_function, 0);
	vmt->hook(hooked_function, 1);
	
	// Calls all the functions
	class_a->function_one();
	class_a->function_two();
	class_a->function_three();

	// Unhooks everything
	printf("--Unhooks Functions---\n");
	vmt->unhook_all();

	// Calls all the functions
	class_a->function_one();
	class_a->function_two();
	class_a->function_three();

	return 0;
}
```

```text title="VMT Hooking Result"
--Hooks Functions---
hooked_function() called
a::function_two()
hooked_function() called

--Unhooks Functions---
a::function_one()
a::function_two()
a::function_three()
```

## IAT Hooking

```text title="IAT Hooking Graph"
+-------------------+            
| main:             |                                                +---------------------------+
| push ebp          |                                                | MessageBoxA_addr:         |
| mov esp, ebp      |                                           +--->| mov edi, edi              |<-+
| code..            |       +--------+      +-----------------+ |    | push ebp                  |  |
| call MessageBoxA  |------>| Hooked |----->| jmp MessageBoxA |-+    | mov ebp, esp              |  |
| code...           |       +--------+ No   +-----------------+      | executes original code... |  |
| ret               |           | Yes                                | ret                       |  | 
+-------------------+           |                                    +---------------------------+  |
                                |                                                                   |
                                |                                                                   |
                                |                      +---------------------------+                |
                                v              +-----> | hooked_messageboxa_addr:  |                |
                    +------------------------+ |       | push ebp                  |                |
                    | jmp hooked_messageboxa | |       | mov ebp, esp              |                |
                    +------------------------+ |       | executes our code...      |                |
                                |              |       | call MessageBoxA          |----------------+
                                +--------------+       | ret                       |
                                                       +---------------------------+
```

Windows executables has a Import Address Table (IAT) which contains different functionalities that allows us to interact with the operating system. IAT Hooking allows us to replace these functionalities with our own code and then we can call the original function inside of our code to ensure the application is running smoothly. 

```c++ title="IAT Hooking Function"
DWORD iat_hook(void* source, LPCSTR destination)
{
	// No need to get Module Handle
	LPVOID image_base = GetModuleHandleA(NULL);

	// dos_headers is a pointer to DOS_HEADER
	PIMAGE_DOS_HEADER dos_headers = (PIMAGE_DOS_HEADER)image_base;

	// nt_headers is a pointer to NT_HEADERS
	PIMAGE_NT_HEADERS nt_headers = (PIMAGE_NT_HEADERS)((DWORD)image_base + dos_headers->e_lfanew);

	// Will be used to get all the DLL imports 
	PIMAGE_IMPORT_DESCRIPTOR import_descriptor = NULL;

	// imports_directory is a pointer to  "Import Directory"
	IMAGE_DATA_DIRECTORY imports_directory = nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT];

	// Will be used to get library names
	import_descriptor = (PIMAGE_IMPORT_DESCRIPTOR)(imports_directory.VirtualAddress + (DWORD)image_base);

	// Variables
	LPCSTR library_name = NULL;
	HMODULE library = NULL;
	PIMAGE_IMPORT_BY_NAME function_name = NULL;

	// Loops through functions inside of Import Address Table to the function is found
	while (import_descriptor->Name != NULL)
	{
		library_name = (LPCSTR)import_descriptor->Name + (DWORD)image_base;
		library = LoadLibraryA(library_name);

		if (library)
		{
			PIMAGE_THUNK_DATA original_first_thunk = NULL, first_thunk = NULL;
			original_first_thunk = (PIMAGE_THUNK_DATA)((DWORD)image_base + import_descriptor->OriginalFirstThunk);
			first_thunk = (PIMAGE_THUNK_DATA)((DWORD)image_base + import_descriptor->FirstThunk);

			while (original_first_thunk->u1.AddressOfData != NULL)
			{
				function_name = (PIMAGE_IMPORT_BY_NAME)((DWORD)image_base + original_first_thunk->u1.AddressOfData);

				if (std::string(function_name->Name).compare(destination) == 0)
				{
					SIZE_T bytes_written = 0;
					DWORD old_protect = NULL;
					VirtualProtect(&first_thunk->u1.Function, 5, PAGE_READWRITE, &old_protect);

					// Storing original address
					DWORD original_addr =  (DWORD)first_thunk->u1.Function;

					// Overriding MessageBoxA with hooked_messageboxa
					first_thunk->u1.Function = (DWORD)source;

					// Returning address
					return original_addr;
				}
				++original_first_thunk;
				++first_thunk;
			}
		}
		++import_descriptor;
	}
	
	return NULL;
}
```

Inside the `iat_hook()` function the Import Address Table is being searched through to find the specified function in `destination` parameter, and once the function is found the address is stored at `original_addr` variable and then the address is overwritten with `source` parameter. Here's a overview of IAT Hooking in use:

```cpp title="IAT Hooking Example"
#include <iostream>
#include <Windows.h>

DWORD iat_hook(void* source, LPCSTR destination)
{
	// No need to get Module Handle
	LPVOID image_base = GetModuleHandleA(NULL);

	// dos_headers is a pointer to DOS_HEADER
	PIMAGE_DOS_HEADER dos_headers = (PIMAGE_DOS_HEADER)image_base;

	// nt_headers is a pointer to NT_HEADERS
	PIMAGE_NT_HEADERS nt_headers = (PIMAGE_NT_HEADERS)((DWORD)image_base + dos_headers->e_lfanew);

	// Will be used to get all the DLL imports 
	PIMAGE_IMPORT_DESCRIPTOR import_descriptor = NULL;

	// imports_directory is a pointer to  "Import Directory"
	IMAGE_DATA_DIRECTORY imports_directory = nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT];

	// Will be used to get library names
	import_descriptor = (PIMAGE_IMPORT_DESCRIPTOR)(imports_directory.VirtualAddress + (DWORD)image_base);

	// Variables
	LPCSTR library_name = NULL;
	HMODULE library = NULL;
	PIMAGE_IMPORT_BY_NAME function_name = NULL;

	// Will loop to "MessageBoxA" function is found on import table.
	while (import_descriptor->Name != NULL)
	{
		// Getting library name
		library_name = (LPCSTR)import_descriptor->Name + (DWORD)image_base;

		// Getting library
		library = LoadLibraryA(library_name);

		if (library)
		{
			PIMAGE_THUNK_DATA original_first_thunk = NULL, first_thunk = NULL;
			original_first_thunk = (PIMAGE_THUNK_DATA)((DWORD)image_base + import_descriptor->OriginalFirstThunk);
			first_thunk = (PIMAGE_THUNK_DATA)((DWORD)image_base + import_descriptor->FirstThunk);

			while (original_first_thunk->u1.AddressOfData != NULL)
			{
				function_name = (PIMAGE_IMPORT_BY_NAME)((DWORD)image_base + original_first_thunk->u1.AddressOfData);

				if (std::string(function_name->Name).compare(destination) == 0)
				{
					SIZE_T bytes_written = 0;
					DWORD old_protect = NULL;
					VirtualProtect(&first_thunk->u1.Function, 5, PAGE_READWRITE, &old_protect);

					// Storing original address
					DWORD original_addr =  (DWORD)first_thunk->u1.Function;

					// Overriding MessageBoxA with hooked_messageboxa
					first_thunk->u1.Function = (DWORD)source;

					// Returning address
					return original_addr;
				}
				++original_first_thunk;
				++first_thunk;
			}
		}
		++import_descriptor;
	}

	return NULL;
}

typedef int(__stdcall* t_orig_messageboxa)(HWND hwnd, LPCSTR lp_text, LPCSTR lp_caption, UINT u_type);
t_orig_messageboxa messageboxa_iat = nullptr;

int __stdcall hooked_messageboxa(HWND hwnd, LPCSTR lp_text, LPCSTR lp_caption, UINT u_type)
{
	// Changing lp_text and lp_caption values
	lp_text = "Hooked!";
	lp_caption = "hooked!";

	// Calling original MessageBoxA
	return messageboxa_iat(hwnd, lp_text, lp_caption, u_type);
}

int main()
{
	// Hooking MessageBoxA
	messageboxa_iat = (t_orig_messageboxa)iat_hook(hooked_messageboxa, "MessageBoxAAA");

	// Calling MessageBoxA
	MessageBoxA(NULL, "Hello", "Hello", NULL);

	// Exiting
	return 0;
}
```

What is happening inside the code? The `iat_hook()` function is called to hook the `MessageBoxA` function with `hooked_messageboxa` function and then the `MessageBoxA()` function is called to see if the hooking was successful.

## Hardware Breakpoint Hooking

```text title="Hardware Breakpoint Hooking Graph"
debug registers
   +-----+            +----------------------------------------+      +-----+
   | DR0 |----------->| display_message_addr:                  |   +--| Hit |---------+
   | DR1 | Breakpoint | push ebp                               |<--+  +-----+         |
   | DR2 |            | code...                                |                      |
   | DR3 |            | push ecx                               |<--+                  v 
   +-----+            | lea edi, [ebp-0Ch]                     |   |  +------------------------------------+
                      | mov ecx, 3                             |   +--| exception_info->ContextRecord->Ecx |
                      | mov eax, 0CCCCCCCCCCh                  |      +------------------------------------+
                      | pop ecx                                |                      |
                      | mov dword ptr [message], ecx           |                      |
                      | mov ecx, offset _44F27096_FileName@cpp |                      v
                      | call printf                            |       +----------------------------------+
                      | code...                                |       | We can change ECX register value |
                      +----------------------------------------+       +----------------------------------+
```

Intel x86 and x64 architecture provide a set of registers that are useful for debugging. The DR0 to DR3 are considered "Debug Address Registers" since these registers are used to store the address of the hardware breakpoints. The DR7 register is considered as "Debug Control Register" since it's used to enbale and disable the DR0 to DR3 registers. The DR6 register holds information about the DR0 to DR3 registers once a breakpoint has been triggered.

Hardware Breakpoint Hooking requires us to find the main thread and that can be found enumerating all threads to search for the thread with the earliest creation time, that is what the function `get_main_thread_id()` function does:

```cpp title="Get Main Thread Function"
// The function will return the thread with earliest creation time
DWORD get_main_thead_id(const HANDLE process_handle) {

    std::shared_ptr<HPSS> snapshot(new HPSS{}, [&](HPSS* snapshotPtr) {
        PssFreeSnapshot(process_handle, *snapshotPtr);
        });

    if (PssCaptureSnapshot(process_handle, PSS_CAPTURE_THREADS, 0, snapshot.get()) != ERROR_SUCCESS)
    {
        printf("PssCaptureSnapshot failed...");
    }

    std::shared_ptr<HPSSWALK> walker(new HPSSWALK{}, [&](HPSSWALK* walkerPtr) { PssWalkMarkerFree(*walkerPtr); });

    if (PssWalkMarkerCreate(nullptr, walker.get()) != ERROR_SUCCESS)
    {
        printf("PssWalkMarkerCreate failed...");
    }

    DWORD main_thread_id{};
    FILETIME lowest_create_time{ MAXDWORD, MAXDWORD };

    PSS_THREAD_ENTRY thread{};

    // Iterate through the threads and keep track of the one
    // with the lowest creation time.
    while (PssWalkSnapshot(*snapshot, PSS_WALK_THREADS,
        *walker, &thread, sizeof(thread)) == ERROR_SUCCESS) {
        if (CompareFileTime(&lowest_create_time, &thread.CreateTime) == 1) {
            lowest_create_time = thread.CreateTime;
            main_thread_id = thread.ThreadId;
        }
    }

    return main_thread_id;
}
```

The `set_debug_breakpoint()` function will set a hardware breakpoint on `target_addr` parameter and enable hardware breakpoints. 

```cpp title="Set Debug Breakpoint Function"
bool set_debug_breakpoint(const HANDLE& main_thread_handle, const void* const target_addr)
{
    CONTEXT context{
        .ContextFlags = CONTEXT_DEBUG_REGISTERS,
        .Dr0 = (DWORD)target_addr,
        .Dr7 = (1 << 0)
    };

    // Set the main threads context
    if (!SetThreadContext(main_thread_handle, &context)) {
        printf("SetThreadContext failed...\n");
        return false;
    }

    // Resume the thread after setting its context
    ResumeThread(main_thread_handle);

    return true;
}
```

The `exception_handler()` function will check which debug address register has been triggered and if the DR0 register has been triggered than ECX value is changed to "Hooked Message". 

```cpp
LONG WINAPI exception_handler(EXCEPTION_POINTERS* const exception_info)
{
    if (exception_info->ExceptionRecord->ExceptionCode == EXCEPTION_SINGLE_STEP)
    {
        if (exception_info->ContextRecord->Dr6 & 0x1)
        {
            auto first_parameter = reinterpret_cast<std::string*>(exception_info->ContextRecord->Ecx);
            *first_parameter = "Hooked Message\n";

            exception_info->ContextRecord->EFlags |= 0x10000;
        }
     
        return EXCEPTION_CONTINUE_EXECUTION;
    }
   
    return EXCEPTION_CONTINUE_EXECUTION;
}
```

The `message` parameter of the `display_message()` function will be hooked so each time the function is called the message `"Hooked"` is displayed instead of the original message inside of the `message` parameter.

```cpp title="Display Message Function"
void __fastcall display_message(const std::string& message)
{
    printf("%s\n", message.c_str());
}
```

Now since all the functions purposes has been explained. Here's a overview of Hardware Breakpoint Hooking in use:

```cpp title="Hardware Breakpoint Hooking Example"
#include <iostream>
#include <Windows.h>
#include <thread>
#include <TlHelp32.h>
#include <ProcessSnapshot.h>

LONG WINAPI exception_handler(EXCEPTION_POINTERS* const exception_info)
{
    if (exception_info->ExceptionRecord->ExceptionCode == EXCEPTION_SINGLE_STEP)
    {
        if (exception_info->ContextRecord->Dr6 & 0x1)
        {
            auto first_parameter = reinterpret_cast<std::string*>(exception_info->ContextRecord->Ecx);
            *first_parameter = "Hooked\n";

            exception_info->ContextRecord->EFlags |= 0x10000;
        }
     
        return EXCEPTION_CONTINUE_EXECUTION;
    }
   
    return EXCEPTION_CONTINUE_EXECUTION;
}

DWORD get_main_thead_id(const HANDLE process_handle) {

    std::shared_ptr<HPSS> snapshot(new HPSS{}, [&](HPSS* snapshotPtr) {
        PssFreeSnapshot(process_handle, *snapshotPtr);
        });

    if (PssCaptureSnapshot(process_handle, PSS_CAPTURE_THREADS, 0, snapshot.get()) != ERROR_SUCCESS)
    {
        printf("PssCaptureSnapshot failed...");
    }

    std::shared_ptr<HPSSWALK> walker(new HPSSWALK{}, [&](HPSSWALK* walkerPtr) { PssWalkMarkerFree(*walkerPtr); });

    if (PssWalkMarkerCreate(nullptr, walker.get()) != ERROR_SUCCESS)
    {
        printf("PssWalkMarkerCreate failed...");
    }

    DWORD main_thread_id{};
    FILETIME lowest_create_time{ MAXDWORD, MAXDWORD };

    PSS_THREAD_ENTRY thread{};

    while (PssWalkSnapshot(*snapshot, PSS_WALK_THREADS,
        *walker, &thread, sizeof(thread)) == ERROR_SUCCESS) {
        if (CompareFileTime(&lowest_create_time, &thread.CreateTime) == 1) {
            lowest_create_time = thread.CreateTime;
            main_thread_id = thread.ThreadId;
        }
    }

    return main_thread_id;
}

bool set_debug_breakpoint(const HANDLE& main_thread_handle, const void* const target_addr)
{
    CONTEXT context{
        .ContextFlags = CONTEXT_DEBUG_REGISTERS,
        .Dr0 = (DWORD)target_addr,
        .Dr7 = (1 << 0)
    };

    if (!SetThreadContext(main_thread_handle, &context)) {
        printf("SetThreadContext failed...\n");
        return false;
    }

    ResumeThread(main_thread_handle);

    return true;
}

void __fastcall display_message(const std::string& message)
{
    printf("%s\n", message.c_str());
}

int main()
{
    DWORD main_thead_id = get_main_thead_id(GetCurrentProcess());
    HANDLE main_thread_handle = OpenThread(THREAD_SET_CONTEXT | THREAD_SUSPEND_RESUME, false, main_thead_id);
    AddVectoredExceptionHandler(true, exception_handler);
    set_debug_breakpoint(main_thread_handle, display_message);
    CloseHandle(main_thread_handle);
    display_message("Hello World");
    return 0;
}
```

What is happening inside of the code? As explained previously, each time the `display_messsage()` function is called the message `"Hooked"` will be displayed instead of the message inside of the parameter. Hardware Breakpoint Hooking is a difficult topic to understand therefore I highly recommend playing around with it.
