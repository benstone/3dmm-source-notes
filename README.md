# 3DMM Source Notes

This is a collection of notes about the [Microsoft 3D Movie Maker source code](https://github.com/microsoft/Microsoft-3D-Movie-Maker).

**NOTE:** This is not official documentation, and will almost certainly have some inaccuracies. If you find something that isn't correct, please create an issue or a pull request to fix it. Contributions are welcome!

## Source repos

* Official repo: https://github.com/microsoft/Microsoft-3D-Movie-Maker
* Forks:
  * [3DMM Forever](https://github.com/foone/3DMMForever) by [Foone Turing](https://twitter.com/foone)
  * [Clemenswasser's fork](https://github.com/clemenswasser/Microsoft-3D-Movie-Maker)
  * [Frank Weindel's fork](https://github.com/frank-weindel/3DMMForever)
  * [My fork](https://github.com/benstone/Microsoft-3D-Movie-Maker)
  * [Other forks](https://github.com/microsoft/Microsoft-3D-Movie-Maker/network)

## Source tree layout

* SRC: The source code for Socrates/3DMM
  * STUDIO: Main application executable, scripts / assets for the studio
  * ENGINE: Library that handles movie creation and playback
  * BUILDING, SHARED: Scripts and assets for navigating around the building
  * HELP: Context-sensitive help, Talent Book pages
  * HELPAUD: Tooltip narrations
* KAUAI: The source code for the Kauai application framework
  * TOOLS: Development tools for Kauai
* BREN: Library that wraps BRender
* INC: Headers referenced from C++ source and Chomp script files
* SETUP: Scripts for generating the installer and the final CD layout. This directory also includes some content from the Multimedia Catalog.
* TOOLS: Source for some tools used for generating 3D content and texture maps
* CD12: Another copy of the source tree. I'm not sure why this has been duplicated.
* CD2: A collection of PowerPoint slides from an internal conference circa 1996 which seems to be completely unrelated to 3DMM.
* CD3: The CD layout of the RTM release of 3DMM (excluding binaries).
* CD9: Compiled 3D content and pre-rendered videos

## Documentation

The source repo contains some documentation:

* [Kauai documentation](https://github.com/microsoft/Microsoft-3D-Movie-Maker/tree/main/kauai/DOC)
* Test release documentation: [one from April 1995](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/TRD.TXT) and [another from a later date](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/NEWTRD.TXT)
* [Building room entry states](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/SRC/BUILDING/ENTSTATE.TXT)

## Build Environment

### Compiler

3DMM was likely compiled with Microsoft Visual C++ 2.1.

* The source code builds with MSVC 2.1 without any changes. MSVC 2.0 will compile the source but not link it due to missing CRT functions. MSVC 2.2 has newer headers that cause compile errors.
* `kauai\src\MDEV2PRI.H` includes some references to using headers from MSVC 2.1:
  * eg. line 196: `// Define these so we can use old (msvc 2.1) header files`
* MSVC 2.1 and 2.2 have the same linker version (2.55) which matches the linker version in the release executable headers.

Kauai was designed to also run on the Macintosh. It is not yet confirmed which compiler was used for Macintosh builds.

### Environment Variables

System environment variables:

* MSVCNT_ROOT: Set to the directory where MSVC is installed
* PATH: The MSVC `bin` directory needs to be on the path
* INCLUDE: Set to MSVC `include` directory
* LIB: Set to MSVC `lib` directory

Build environment variables:

* SOC_ROOT: Path to root of source repo
* KAUAI_ROOT: Path to Kauai: %SOC_ROOT%\kauai
* ARCH: Operating system to build for
  * WIN: Windows
  * MAC: Macintosh 68k
* INCLUDE: Set to the MSVC include directories, plus:
  * %SOC_ROOT%\INC
  * %SOC_ROOT%\BREN\INC
  * %SOC_ROOT%\SRC
  * %KAUAI_ROOT%\SRC
* UNICODE: If set to non-empty, build Unicode instead of ANSI
  * NOTE: The Unicode build is broken
* TYPE: Sets build type
  * DAY: Debug, incremental (daily?)
  * HOME: Debug, not incremental
  * SHIP: Release
  * DBSHIP: Release, with linker debug output
* CHIP: Set to use optimized assembly language implementations of some functions in Kauai
  * IN_80386: Intel 80386
  * MC_68020: Motorola 68020 (for Macintosh)
  * If not set, uses the C implementation

Many of these variables are defined in `kauai\MAKEFILE.DEF`.

### Build artifacts

The repo contains some build artifacts of what were possibly the RTM build. The `OBJ\WINS` directory contains a linker map file that has the same timestamp as the final release of `3DMOVIE.EXE`. This file can be loaded into Ghidra/IDA to add symbols to the release executable. The directory also includes preprocessed chunky script files (as .I files).

Object files and executable files have been removed from the build.

### External libraries

The build expects the BRender and AudioMan libraries to be present in `elib` and `kauai\elib` respectively. These libraries can be obtained from [3DMMForever](https://github.com/foone/3DMMForever). Place them in subdirectories that match the build type (eg. `WINS` for the ANSI release).

## Build Tools

These tools are built as part of Kauai. The binaries will be in `kauai\obj\<build-type>`.

### Chomp

Chomp is a compiler used to generate chunky files. The Kauai makefile rules run the C preprocessor over the CHT files before calling Chomp.

The source tree includes a document describing the [syntax used by Chomp](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/DOC/CHOMP.DOC).

The CHT files use a lot of macros to reduce code duplication. Many of these macros are defined in [KIDGS.H](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/INC/KIDGS.H).

### Chelp

Chelp is an editor for help topics. Help topics are also used for tooltips, dialog boxes, Talent Book pages, etc.

### Mkmbmp

Mkmbmp converts a bitmap file (*.bmp) to a MBMP image, and optionally compresses it with Kauai's compression codec. The source tree contains many PBM files which are referenced by the CHT scripts. These PBM files are MBMP images which were likely generated by mkmbmp.

### Kpack

Kpack will compress or decompress a file using Kauai's compression codec.

### KCDC_386 and KCD2_386

These tools [generate an Intel x86 assembly version](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/KCDC_386.C) of the [custom compression codec](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CODKAUAI.CPP). This is used if CHIP is set to IN_80386.

## Debugging

### Initial Setup

If you don't have 3DMM installed, you will need to create the following directory layout:

* (base directory eg. c:\mskids)
    * 3DMOVIE.EXE
    * 3DMOVIE (directory)
        * place all CHK/3TH/3CN/AVI files here
    * Users (directory)
        * Melanie (directory)

You will also need to create a registry value to set the product name. Go to `HKLM\SOFTWARE\Microsoft\Microsoft Kids\3D Movie Maker\Products` and create a string value named `1` with data `3D Movie Maker/3DMovie`.

### Building a debug version

Build a debug version of 3DMM by setting the environment variable `TYPE=DAY` and re-running nmake. This will build debug binaries to `OBJ\WIND`.

The debug build will also generate debug chunky files. The scripts in debug chunky files will print log messages to a debugger if attached. When a debug build starts it will look for debug versions of chunky files by appending "D" to the filename (eg. `STUDIOD.CHK`) and load those if present.

Debug builds add an option to change the assertion level. This can be changed at runtime by pressing Ctrl-Alt-I and changing the cactAV value.

## File Formats

### Chunky File

See [kauai/SRC/CHUNK.CPP](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP) for a high-level overview of the chunky file format.

The Kauai tools directory includes Ched, a graphical chunky file editor, and Chomp, a chunky file compiler. Third-party tools that can read chunky files include [3DMM Pencil](http://frank.weindel.info/proj.pencil.html), [Chunk Extractor](https://3dmm.com/showthread.php?t=50392), and [Pymaginopolis](https://github.com/benstone/pymaginopolis).

### Chunk types

Serializable classes will generally inherit from the BACO (base cacheable object) class. The FWrite function serializes the object to a data block object (BLCK). To find the on-disk representation of a class, look for this function.

## Interesting finds

* The file format was [changed multiple times](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP#L80) during the development of 3DMM. The initial version was developed in October 1993. The last change was in September 1995, one month before RTM!
* The chunky file format has a field that indicates the type of program that wrote the file. Eg: 'CHMP' indicates that the file was compiled with Chomp vs. 'CHED' which indicates that it was saved using the chunky editor. This can be used to determine if the file was generated by a tool or manually edited.
* Debug builds have an option to dump every frame of the movie to a bitmap file. Open a movie in the studio and press Ctrl-F10 to dump all of the frames.
* Adding `#define SHOW_FPS` to [STDIODEF.H](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/INC/STDIODEF.H#L16) will show the framerate while playing movies in the Studio.
* The [setup build script](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/SETUP/3DMOVIE.DDF) generates the final CD layout and places it on a server called `\\SHEILA`.
* The setup directory includes some code from Microsoft Explorapedia (codenamed Sendak), including what appears to be [a setup Easter egg](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/SETUP/CUSTDLL/CREDITS.CPP).

## Glossary

* ACME: Setup framework used by many Microsoft products in the 1990s
* [APP](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/INC/UTEST.H): Application class
* [APPB](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/APPB.H): Kauai base class for application 
* AudioMan: Audio library used in multiple Microsoft Home products
* [BACO](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CRF.H): Base cacheable object (ie. a serializable object)
* BRender: 3D rendering library developed by Argonaut Technologies
* [BWLD](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/INC/BWLD.H): BRender World class
* [CEX](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CMD.CPP#L12): Command execution dispatcher: Kauai class that manages dispatching messages to command handler objects (subclasses of CMH)
* Ched: Chunky editor: GUI tool for editing chunky files
* Chelp: GUI tool for editing help content inside chunky files
* [CHID](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP): Child chunk ID
* CID: Command ID
* Chomp: Chunky compiler: compiles .CHT files and creates Chunky files
* [Chunky](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP): File format used for resources, scripts and movie files
* [CKI](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP): Chunk identifier: combination of a ctg and cno
* [CMH](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CMD.CPP#L19): Command handler: a class that can receive commands
* [CNO](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP): Chunk number
* [CODM](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CODEC.CPP): Codec manager
* [CTG](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CHUNK.CPP): Chunk tag / type
* DKIT: SoftImage SDK
* ELIB: Directory name used for external static libraries
* [ERS](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/UTILERRO.H): Error stack class used for error reporting
* [FNI](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/FNI.H): File name class
* [GG](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/DOC/GROUPS.TXT): General group
* [GL](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/DOC/GROUPS.TXT): General list
* [GNV](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/GFX.H): Graphics Environment
* [GOB](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/GOB.H): Graphical object
* [GOK](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/KIDSPACE.H): Graphical object in Kidspace
* [GOKD](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/KIDWORLD.H): GOK Descriptor
* [GORP](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/KIDSPACE.H): Graphical object representation. Has subclasses for bitmaps (GORB), fills (GORF), tiled bitmaps (GORT), and video (GORV).
* [GPT](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/GFX.H): Graphics port. Implemented in [GFXWIN.CPP](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/GFXWIN.CPP) for Windows and [GFXMAC.CPP](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/GFXMAC.CPP) for the Macintosh.
* [GST](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/DOC/GROUPS.TXT): String table
* [HBAL](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/KIDHELP.h): Help Balloon
* [Hungarian Notation](http://www.byteshift.de/msg/hungarian-notation-doug-klunder): Coding style used in many Microsoft projects in the 1990s
* Kauai: Application framework used by 3D Movie Maker and Creative Writer 2
* [KCDC](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CODKAUAI.CPP): Kauai Codec compression algorithm
* [KCD2](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/CODKAUAI.CPP): Variant of Kauai Codec that supports multi-byte runs of uncompressed data
* Kidspace: Kauai scripting system
* [MDPS](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/MIDIDEV2.H): MIDI player class
* MBMP: Masked Bitmap
* Mkmbmp: Tool that converts a BMP file into a MBMP, and optionally compresses it with KCDC/KCD2
* Playdo: Unknown project related to Socrates (do you know what this is? let me know!)
* RTM: Release to manufacturing
* [SDAM](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/SNDAM.CPP): Sound device class that wraps the AudioMan library
* Sitobren: Tool to convert 3D models from SoftImage to BRender
* [SNDM](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/SNDM.H): Sound Manager
* [SNDV](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/SNDM.H): Sound Device - base class for AudioMan wrapper class and MIDI player class
* Socrates: Codename for 3D Movie Maker
* [STN](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/kauai/SRC/UTILSTR.H): String class
* [Tag Manager](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/INC/TAGMAN.H): Class that manages caching content from the CD
* Tdfmake: 3D font authoring tool
* [TGOB](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/INC/TGOB.H): Text rendering GOB
* TRD: Test Release Document
* [Utest](https://github.com/microsoft/Microsoft-3D-Movie-Maker/blob/main/SRC/STUDIO/UTEST.CPP): Internal name of the main program executable
* WOKS: World of Kidspace. Contains GOKs, help balloons and script interpreters.
