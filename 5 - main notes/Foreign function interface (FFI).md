---
created: 2026/08/13  -  12:23
updated: 2026-11-09 21:06
type: note
status: seed
topic:
tags:
  - software-engineering
aliases: []
---

A **foreign function interface** (**FFI**) is a mechanism by which a program written in one [programming language](https://en.wikipedia.org/wiki/Programming_language "Programming language") can call [interoperable](https://en.wikipedia.org/wiki/Language_interoperability "Language interoperability") routines or make use of services written or compiled in another one. An FFI is often used in contexts where calls are made into a binary [dynamic-link library](https://en.wikipedia.org/wiki/Dynamic-link_library "Dynamic-link library").

The primary function of a foreign function interface is to mate the semantics and [calling conventions](https://en.wikipedia.org/wiki/Calling_convention "Calling convention") of one programming language (the _host_ language, or the language which defines the FFI), with the semantics and conventions of another (the _guest_ language). This process must also take into consideration the [runtime environments](https://en.wikipedia.org/wiki/Runtime_environment "Runtime environment") and [application binary interfaces](https://en.wikipedia.org/wiki/Application_binary_interface "Application binary interface") of both. This can be done in several ways:

- Requiring that guest-language functions which are to be host-language callable be specified or implemented in a particular way, often using a compatibility [library](https://en.wikipedia.org/wiki/Library_\(computing\) "Library (computing)") of some sort.
- Use of a tool to automatically _wrap_ guest-language functions with appropriate [glue code](https://en.wikipedia.org/wiki/Glue_code "Glue code"), which performs any necessary translation.
- Use of a [wrapper library](https://en.wikipedia.org/wiki/Wrapper_library "Wrapper library")
- Restricting the set of host language abilities which can be used cross-language. For example, C++ functions called from C may not (in general) include reference parameters or throw exceptions.
C++ offers compatibility with C trivially, however as C++ uses [name mangling](https://en.wikipedia.org/wiki/Name_mangling "Name mangling"), symbols that are to be exported in [shared libraries](https://en.wikipedia.org/wiki/Shared_library "Shared library") must be wrapped in [extern "C"](https://en.wikipedia.org/wiki/Extern_%22C%22 "Extern \"C\"") blocks which prevent name mangling. `extern "C"` functions may call C++ code internally, but may only return C types and may not throw [exceptions](https://en.wikipedia.org/wiki/Exception_handling_\(programming\) "Exception handling (programming)").

FFIs may be complicated by the following considerations:

- If one language supports [garbage collection](https://en.wikipedia.org/wiki/Garbage_collection_\(computer_science\) "Garbage collection (computer science)") (GC) and the other does not; care must be taken that the non-GC language code does nothing to cause GC in the other to fail. In JNI, for example, C code which "holds on to" object references that it receives from Java must communicate this information successfully to the [Java virtual machine](https://en.wikipedia.org/wiki/Java_virtual_machine "Java virtual machine") or [Java Runtime Environment](https://en.wikipedia.org/wiki/Java_Runtime_Environment "Java Runtime Environment") (JRE), otherwise, Java may delete objects before C finishes with them. (The C code must also explicitly release its link to any such object once C has no further need of that object.)
- Complicated or non-trivial objects or datatypes may be difficult to map from one environment to another.
- It may not be possible for both languages to maintain references to the same instance of a mutable object, due to the mapping issue above.
- One or both languages may be running on a [virtual machine](https://en.wikipedia.org/wiki/Virtual_machine "Virtual machine") (VM); moreover, if both are, these are often different VMs.
- Cross-language [inheritance](https://en.wikipedia.org/wiki/Inheritance_\(object-oriented_programming\) "Inheritance (object-oriented programming)") and other differences, such as between [type systems](https://en.wikipedia.org/wiki/Type_system "Type system") or between [object composition](https://en.wikipedia.org/wiki/Object_composition "Object composition") models, may be especially difficult.

Many FFIs can be generated automatically: for example, [SWIG](https://en.wikipedia.org/wiki/SWIG "SWIG"). However, in the case of an [extension language](https://en.wikipedia.org/wiki/Extension_language "Extension language") a semantic inversion of the relationship of guest and host can occur, when a smaller body of extension language is the guest invoking services in the larger body of host language, such as writing a small plugin[[8]](https://en.wikipedia.org/wiki/Foreign_function_interface#cite_note-8) for GIMP.[[9]](https://en.wikipedia.org/wiki/Foreign_function_interface#cite_note-9)

Some FFIs are restricted to free standing [functions](https://en.wikipedia.org/wiki/Function_\(computer_programming\) "Function (computer programming)"), while others also allow calls of functions embedded in an object or class (often called [method calls](https://en.wikipedia.org/wiki/Method_call "Method call")); some even permit migration of complex datatypes or objects across the language boundary.

In most cases, an FFI is defined by a [higher-level language](https://en.wikipedia.org/wiki/High-level_programming_language "High-level programming language"), so that it may employ services defined and implemented in a [lower-level language](https://en.wikipedia.org/wiki/Low-level_programming_language "Low-level programming language"), typically a [system programming language](https://en.wikipedia.org/wiki/System_programming_language "System programming language") like [C](https://en.wikipedia.org/wiki/C_\(programming_language\) "C (programming language)") or [C++](https://en.wikipedia.org/wiki/C++ "C++"). This is typically done to either access [operating system](https://en.wikipedia.org/wiki/Operating_system "Operating system") (OS) services in the language in which the OS API is defined, or for performance goals.

Many FFIs also provide the means for the called language to invoke services in the host language also.

The term foreign function interface is generally not used to describe multi-lingual runtimes such as the Microsoft [Common Language Runtime](https://en.wikipedia.org/wiki/Common_Language_Runtime "Common Language Runtime"), where a common _substrate_ is provided which enables any CLR-compliant language to use services defined in any other. (However, in this case the CLR does include an FFI, [P/Invoke](https://en.wikipedia.org/wiki/P/Invoke "P/Invoke"), to call outside the runtime.) In addition, many distributed computing architectures such as the [Java remote method invocation](https://en.wikipedia.org/wiki/Java_remote_method_invocation "Java remote method invocation") (RMI), RPC, [CORBA](https://en.wikipedia.org/wiki/CORBA "CORBA"), [SOAP](https://en.wikipedia.org/wiki/SOAP "SOAP") and [D-Bus](https://en.wikipedia.org/wiki/D-Bus "D-Bus") permit different services to be written in different languages; such architectures are generally not considered FFIs.
# References
