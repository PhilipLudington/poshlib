# POSH According to Wikipedia
Posh is a software framework used in cross-platform software development. It was created by Brian Hook. It is BSD licensed and as of 17 March 2014 at version 1.3.002.  - https://en.wikipedia.org/wiki/Poshlib

# OverView

Overview: Rationale, Goals, and Philosophy

As a general rule, there is no great mystery when it comes to writing portable software. There are a certain number of obvious gotchas a programmer encounters, and techniques that a programmer learns, to make sure his or her software compiles and runs on a wide range of hardware, compilers and operating systems.

Unfortunately, a lot of the work that programmers undertake when writing cross-platform software is redundant, tedious, error prone and, well, boring. The most fundamental, basic tasks -- inferring your current configuration environment, byte order/endian safety routines, import/export function signatures for Windows DLLs, correctly sized data types -- are performed over and over by thousands of programmers around the world.
The Catalyst

Every major cross-platform, open source project provides these facilities, usually in the form of a "config.h" or "types.h" header and some associated source files, but extracting just the configuration elements from these projects can prove cumbersome and, in the case of licensing, impractical.

I ran into this problem when I was trying to extract some of my own code to make it open source. That sub-library was dependent on a lot of bits of unrelated code, mostly having to do with the issues I described earlier. So I did the obvious thing -- I extracted those pieces separately, and made a "libconfig.h", and all was well.

Until I wanted to open source another sublibrary.

Now, was I going to basically copy and rename a bunch of stuff in "libconfig.h", or was I going to figure out a way of sharing libconfig.h between two completely unrelated projects? I opted for the latter, and it was about that time that I realized that no one else has tried to make a cross-platform, project agnostic "libconfig.h".

And thus the Portable Open Source Harness was born.
The Goals

I set out to make a single header file that could compile almost anywhere and, in the process, tell me about the host platform and target system. My goals were simple:

    an open, unrestrictive License
    easy to integrate
    easy to use
    no external dependencies on anything, including the C run-time library 

Once those goals were defined, I then had to determine what specific features I wanted POSH to have. The original idea was to create a set of sized types; define macros that indicated target operating system, CPU and endianess; and proper handling of import/export identifiers when building Windows DLLs.

After discussion with some friends, I decided to add support for in-memory serialization/deserialization, verification and architecture string reporting. These elements required the addition of a single source file. Things were already getting more complicated, but two source files is livable complexity.

But in the end, POSH still basically does two things: compile time configuration management and optional run-time routines for endianess conversion, serialization and verification.
Configuration Management

POSH's configuration management consists of detecting the target architecture and operating system, along with the current compiler, and appropriately defining a set of manifest constants that can be "queried" during compilation. POSH only tries to infer basic information such as data type sizes; endianess; pointer size; and the availability of 64-bit integers. More complex information, such as the existence of certain packages, APIs or operating system features, is not supported.

Another key element of POSH's configuration management is the automation of the various magic keywords and linkage specifications necessary to create and use a Windows DLL (specifically, I'm talking about __declspec(dllimport) and __declspec(dllexport)).

The final element of POSH's configuration management is defining a set of correctly sized types (including 64-bit integers) that an application can count on.
Optional Utility Functions

In addition to configuration management, POSH provides a set of optional functions and macros that are relevant and useful to nearly all applications. These routines include retrieving a string representation of the current platform; endianess conversion; and macros for compile time assertions.
What POSH is Not

One problem with many open source projects is overambitiousness -- a simple concept takes on a life of its own and becomes too big and unwieldy for its own good. For that reason, I've gone out of my way to define what POSH is not and what it will hopefully never be.

POSH is not a general cross-platform framework such as SDL, Qt, wxWindows or GTK. It is focused on providing an extremely simple set of features that are useful regardless of the user's domain of interest.

POSH is not an emulation library. It does not attempt to emulate missing features on a given platform, instead choosing to communicate to the programmer (via the build system) as much about the target and host platforms as possible. For example, POSH provides a common access mechanism to the compiler's underlying 64-bit integer type, but if the compiler does not support this feature, POSH simply punts, as opposed to emulating such support.

Finally, POSH is not religious about how extreme it will be when trying to achieve portability. At some point, you have to just say "screw it" when a platform fights you too much. If a particular architecture or compiler is so idiosyncratic that it breaks the rest of POSH, I will not compromise the cleanliness or functionality of POSH just so we can say we run on yet one more platform.

# QuickTutorial

Quick Tutorial

POSH was originally designed to make cross-platform library development easier, so the focus on its design and implementation has been dealing with libraries. That said, it's even easier to use POSH in an application than a library, since applications don't have to worry about the inanity of being built as a Windows DLL or not.

One reason that I haven't really put much thought or effort into POSH as a cross-platform application tool is that most developers of cross-platform apps already leverage third party libraries. Very few non-trivial cross-platform applications are completely dependency free. Because of this, many applications already use POSH-like facilities provided by their libraries. For example, LibSDL, Qt and GTK+ all provide features similar to POSH's, however they are aimed squarely at the application, not library, developer.
Step 1: Add POSH to Your Project

Adding POSH to your project is trivial: stick source:posh.h and source:posh.c somewhere you can get to them, #include the former and, if you're using POSH's utility functions, compile and link to the latter. That's it. If you do things right, an application that uses your library won't have to know you're using POSH at all.
Use POSH Data Types

To leverage POSH's cross-platform exact-sized types, you need to actually use them. No biggie there.

POSH provides type definitions for 8-bit, 16-bit, 32-bit and (if available) 64-bit signed and unsigned integer types. These are in the form posh_u16_t, posh_s32_t, etc. POSH also provides a byte (unsigned 8-bit) type, posh_byte_t.

If you use these types, you are guaranteed to get native types of the exact given size, not "at least" the given size. This is to ensure that serialization and deserialization work, since you need to be able to count on a constant sizeof(T) across platforms.

The posh data types are fairly verbose. For this reason (and to avoid user confusion), you may want to create your own type definitions and simply alias them to the POSH ones, e.g.:

typedef posh_u16_t my_u16;

Step 3: Use POSH Function Decoration/Signature Macros

Note: This is only pertinent to library developers
Step 3a: POSH_PUBLIC_API

Any functions and data exported by your library should have their type or return type wrapped with POSH_PUBLIC_API():

POSH_PUBLIC_API(void) MyLib_Function( void );
POSH_PUBLIC_API(int)  MyLib_integer;

POSH_PUBLIC_API() ensures that the appropriate DLL import/export directives are used if you library is built or used as a Windows DLL.
Step 3b: POSH_CDECL, POSH_STDCALL, and POSH_FASTCALL

Different compilers specify function calling conventions differently. POSH has wrapped these into the macros POSH_CDECL, POSH_STDCALL, and POSH_FASTCALL, which may be used in place of __cdecl, __stdcall, and __fastcall, respectively.
Step 4: Configure POSH'S Preprocessor Symbols

Of course, all this magic requires some effort on the part of the library author, but thankfully not that much. In fact, on most systems you don't have to configure anything if you don't need to disable floating point support and if you aren't aren't building a Windows DLL.

The only three symbols a POSH user is responsible for defining are POSH_BUILDING_LIB, POSH_DLL and POSH_NO_FLOAT.
Step 4a: Defining POSH_BUILDING_LIB

Note: Only relevant to library developers

When building a library should should define the preprocessor symbol POSH_BUILDING_LIB before including source:posh.h. Do this in your source files, not in your public header files'' You do not want this defined inadvertently when a user is trying to link to your library, since they may cause linkage failures on Windows if your library is a DLL.

For example, if your library is called MyLib?, make sure all your source (not header) files define this before including source:posh.h, for example:

//MYLIB.C
#define POSH_BUILDING_LIB
#include "mylib.h" //which in turn includes posh.h

Alternatively, if you distribute a project or Makefile you can ensure that the appropriate compiler option (e.g. -DPOSH_BUILDING_LIB=1) is set correctly instead of modifying your source code.
Step 4b: Defining POSH_DLL

Note: Only relevant to library developers on the Windows platform.

POSH checks the POSH_DLL symbo to determine if the __declspec(dllexport) or __declspec(dllimport) directive should be part of the POSH_PUBLIC_API() macro. This is a moot issue on operating systems other than Windows, but under windows this is very important if you're building a DLL.

The typical way to handle this, especially if you want to build optionally as a statically linked or dynamically linked library, is to have your own preprocessor symbol that the user can define to enable/disable building as a DLL.

For example, if your library is called MyLib?, you might have your own symbol MYLIB_DLL. A user of your library would define this if they are building library as a DLL and/or using it as a DLL. Then in your own code you key off this symbol as such:

//MYLIB_H
#ifndef MYLIB_H
#define MYLIB_H

#if defined MYLIB_DLL
#  define POSH_DLL
#else
#  undef POSH_DLL
#endif

#include "posh.h"

#endif

Step 4c: Defining POSH_NO_FLOAT

POSH provides the ability to serialize/deserialize single and double-precision floating point values by default. However, it may be desirable or necessary to disable floating point support, for example on platforms that lack native floating point support or which do not have IEEE complaint floating point bit representations. Or you may just find that linking without floating support gives you a marginally smaller executable.

To disable floating point support in POSH, simply define the symbo POSH_NO_FLOAT, either at the top of source:posh.h or, preferably, in your makefile/project file (CFLAGS += -DPOSH_NO_FLOAT).
Step 5: Use Endianess Macros

Probably the single most common topic that comes up regarding cross-platform programming is that of endianess assumptions and conversion. For a complete discussion on processor endianess, um, search the Web, because I'm not going to get into the details here.

POSH provides a set of byte order conversion macros, such as POSH_LittleU16() and POSH_BigS32(), along with 64-bit (if available) and floating point versions, that convert a value in a specific endianess to host-endian format.

NOTE: If you use the endianess macros, you will have to link with source:posh.c for byte swapping functionality.
Step 6: Examine Configuration Macros

During compilation POSH looks at the local environment (via examining predefined symbols) and tries to figure out what's what. Once it figures things out, it defines numerous constants to give your code chance to react during the build phase.

The constants potentially defined include:

    POSH_BIG_ENDIAN
    POSH_LITTLE_ENDIAN
    POSH_64BIT_INTEGER
    POSH_64BIT_POINTER
    many other CPU and operating system specific symbols 

Use these constants in your code as appropriate:

#if defined POSH_BIG_ENDIAN
   DoBigEndianStuff();
#endif

#if !defined POSH_64BIT_INTEGER
#  error My library needs 64-bit integer support!
#endif

Note that POSH does not define any compiler identification macros, since unlike CPU and OS target macros, these are (hopefully) going to be consistent and unique. In addition, if you have code that is compiler specific, the expectation is that you already know how to detect that compiler.
Step 7: (Optional) Use Byte Swapping Functions

In support of the endianess macros, POSH provides a set of functions that byte swap 16, 32 and 64-bit (if available) values.

POSH does not have floating point byte swapping functions, since this could theoretically lead to floating-point exceptions on some systems.

The proper way to handle cross-platform floating point is to convert floating point values to integer form and byte swap that value before serialization. For deserialization, simply do the reverse -- read an integer form, byte swap, then load a floating point value from the converted integers. Doing a direct load to a floating point variable then swapping will potentially result in an invalid floating point variable either before or after the swap, depending on the conversion. For more information, see the floating point functions.
Step 8: (Optional) Use Serialization Functions

Directly related to the issue of cross-platform endianess is the ability to serialize and deserialize native data in a portable form. This is typically done by arbitrarily choosing a data file endianess then converting all data from host-to-data endianess at serialization time.

POSH provides a set of in-memory serialization/deserialization functions, along with 64-bit (if available) and floating point versions, that will automatically write native data types to memory and read them back in properly. For floating point values, you must convert to/from integer representation first.
Step 9: (Optional) Use POSH_COMPILE_TIME_ASSERT

POSH provides a cross-platform compile time assertion macro. You don't have to use it, but it's there if you want to. POSH itself uses it fairly liberally in posh.h to sanity check the environment. An example of its use might be something like:

/* ensure that 64-bit integers are actually 64-bits */
POSH_COMPILE_TIME_ASSERT(i64,sizeof(posh_i64_t)==8);

Conclusion

That's pretty much it. There really isn't much documentation for this stuff because it's all fairly straightforward. Browsing source:posh.h should provide you most of the answers for anything you'll run into.

# Source Code and Full Documentation
Code was downloaded from http://poshlib.hookatooka.com/poshlib/trac.cgi
userid: guest
password guest123

# License
This website also spells out the license:

Copyright (c) 2004-2006, Brian Hook All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

The names of this package'ss contributors contributors may not be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE. 
