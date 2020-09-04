# Fully documented source code for Elite on the BBC Micro

This repository contains the original source code for Elite on the BBC Micro, with every single line documented and (for the most part) explained.

The annotated source can be assembled on modern computers to produce a working game disc that can be loaded into a BBC Micro or an emulator.

Hopefully this repository will be useful for those who want to learn more about Elite and what makes it tick. It is provided on an educational and non-profit basis, with the aim of helping people appreciate one of the most iconic games of the 8-bit era.

## Contents

* [Quick start](#quick-start)

* [Acknowledgements](#acknowledgements)

* [Introduction](#introduction)

  * [Ian Bell's original sources](#ian-bells-original-sources)
  * [Paul Brink's annotated disc disassembly](#paul-brinks-annotated-disc-disassembly)
  * [Kieran Connell's elite-beebasm](#kieran-connells-elite-beebasm)
  * [At last, a fully documented version](#at-last-a-fully-documented-version)

* [Versions of Elite](#versions-of-elite)
  
* [Building Elite from the source](#building-elite-from-the-source)

  * [Requirements](#requirements)
  * [Build targets](#build-targets)
  * [Windows](#windows)
  * [Mac and Linux](#mac-and-linux)
  * [Verifying the output](#verifying-the-output)

* [Source files and the build pipeline](#source-files-and-the-build-pipeline)

  * [1. Compile the main game with elite-source.asm](#1-compile-the-main-game-with-elite-sourceasm)
  * [2. Concatenate the game code and compile the header with elite-bcfs.asm](#2-concatenate-the-game-code-and-compile-the-header-with-elite-bcfsasm)
  * [3. Compile the loader with elite-loader.asm](#3-compile-the-loader-with-elite-loaderasm)
  * [4. Calculate checksums and add encryption with elite-checksum.py](#4-calculate-checksums-and-add-encryption-with-elite-checksumpy)
  * [5. Assemble a bootable disc image with elite-disc.asm](#5-assemble-a-bootable-disc-image-with-elite-discasm)
  * [Log files](#log-files)

* [Differences between the various source files](#differences-between-the-various-source-files)

  * [ELITEC](#elitec)
  * [ELTB](#eltb)

* [Next steps](#next-steps)

## Quick start

If you want to jump straight in, here's a _tl;dr_ for you.

* The most interesting files are these ones:

  * The main game's source code is in the [elite-source.asm](elite-source.asm) file - this is the motherlode and probably contains all the stuff you're interested in

  * The game's loader is in the [elite-loader.asm](elite-loader.asm) file - this is mainly concerned with setup and copy protection

* It's probably worth skimming through the notes on terminology at the start of the [elite-loader.asm](elite-loader.asm) file, as this explains a number of terms used in the commentary, without which it might be a bit tricky to follow at times (in particular, you should understand the terminology I use for multi-byte numbers)

* The source code is peppered with a number of "deep dives", each of which goes into an aspect of the game in more detail. You find deep dives in the source files by simply searching for `Deep dive:`

* There are loads of routines in Elite - literally hundreds. I will be adding more information on these soon, but for now you can find them in the source files by searching for `Subroutine name:`

* The entry point for the [main game code](elite-source.asm) is routine `TT170`, which you can find by searching for `Subroutine name:  TT170` (note the two spaces after the colon). If you want to follow the program flow all the way from the title screen around the main game loop, then you can find a deep dive in the `TT170` routine that has you covered

* The source code is designed to be read at an 80-column width and with a monospaced font, just like in the good old days

If you want to build the source on a modern computer, or if you want to know more about this project, keep reading. Otherwise, I hope you enjoy exploring the inner-workings of BBC Elite as much as I have.

## Acknowledgements

The original 1984 source code is copyright &copy; Ian Bell and David Braben, and the code on this site is identical to the version released by the authors on [Ian Bell's personal website](http://www.iancgbell.clara.net/elite/) (it's just been reformatted to be more readable).

The commentary is copyright &copy; Mark Moxon. Any misunderstandings or mistakes in the documentation are entirely my fault.

Huge thanks are due to the original authors for not only creating such an important piece of my childhood, but also for releasing the source code for us to play with; to Paul Brink for his annotated disassembly; and to Kieran Connell for his BeebAsm version, which I forked as the original basis for this repository. You can find out more about all of these in the [introduction](#Introduction) that follows.

## Introduction
This project is based on the original source files for Elite on the BBC Micro, which can be found on [Ian Bell's personal website](http://www.iancgbell.clara.net/elite/). The game code in this repository is totally unchanged from the original source, apart from being reformatted to be easier to read. I've left all the original label names intact, as this site is all about digital archaeology and appreciating the authors' original handiwork.

The following archives from Ian Bell's site form the basis for this project:

* [Cassette sources as a disc image](http://www.elitehomepage.org/archive/a/a4080602.zip)
* [Cassette sources as text files](http://www.elitehomepage.org/archive/a/a4080610.zip)

Here's a bit more on how this project came to be.

### Ian Bell's original sources

When I first saw that the sources to Elite had been released by the authors, I couldn't believe it. I'd always wanted to understand how this astonishing technical feat had been achieved, ever since I'd sat wide-mouthed as a 14-year-old when I first launched from the space station and saw the planet Lave hanging in space, right in front of my eyes. Which, of course, was shortly before dying for the first time, but that didn't matter. It was love at first sight.

So I excitedly opened one of the source files at random... and was greeted by page after page of this kind of thing:

```
 9310LDAXX15+5:.LL147 LDX#Y*2-1:ORAXX12+1:BNELL107:CPXXX12
 9315BCCLL107:LDX#0:.LL107 STXXX13:LDAXX15+1:ORAXX15+3:BNELL83
 9320LDA#Y*2-1:CMPXX15+2:BCCLL83
 9325LDAXX13:BNELL108:.LL146 LDAXX15+2
 9330STAXX15+1:LDAXX15+4:STAXX15+2:LDAXX12:STAXX15+3:CLC:RTS
 9335.LL109 SEC:RTS:.LL108 LSRXX13:.LL83
 9340LDAXX13:BPLLL115
 9345LDAXX15+1:ANDXX15+5:BMILL109:LDAXX15+3:ANDXX12+1:BMILL109
 9350LDXXX15+1:DEX:TXA:LDXXX15+5:DEX:STXXX12+2:ORAXX12+2
 9355BPLLL109:LDAXX15+2:CMP#Y*2:LDAXX15+3:SBC#0:STAXX12+2
 9360LDAXX12:CMP#Y*2:LDAXX12+1:SBC#0:ORAXX12+2:BPLLL109
 9365.LL115 TYA:PHA:LDAXX15+4:SEC:SBCXX15:STAXX12+2:LDAXX15+5
 9370SBCXX15+1:STAXX12+3:LDAXX12:SEC:SBCXX15+2:STAXX12+4
 9375LDAXX12+1:SBCXX15+3:STAXX12+5:EORXX12+3:STAS
```

I suppose I should have expected it, but the original source files are _incredibly_ terse. Because the game was compiled on a BBC Micro, the source code had to be squashed into a number of extremely cramped BASIC files, with all the spaces removed and almost no comments to speak of. The source files are not particularly human-friendly; they aren't supposed to be.

Not only that, but parts of the game started life on an Acorn Atom, where labels in assembly language are restricted to two letters plus digits, so the source is full of memorable names like `XX16`, `QQ17` and `LL9`. I mean, look at this bit:

```
8501.LL42 \DO nodeX-Ycoords
8506\TrnspMat:LDYXX16+2:LDXXX16+3:LDAXX16+6:STAXX16+2:LDAXX16+7:STAXX16+3:STYXX16+6:STXXX16+7
8508LDYXX16+4:LDXXX16+5:LDAXX16+12:STAXX16+4:LDAXX16+13
8509STAXX16+5:STYXX16+12:STXXX16+13
8510LDYXX16+10:LDXXX16+11:LDAXX16+14:STAXX16+10:LDAXX16+15
8511STAXX16+11:STYXX16+14:STXXX16+15
```

All those `XXX`s are enough to make your eyes boggle, but at least this excerpt has some comments, so do they help? `TrnspMat` - is that "transponder materials"? Or "transport maths"? I guess it's something to do with `DO nodeX-Ycoords`, which clearly involves nodes and coordinates, but it's not exactly readable. (I now know that this is part of the `LL42` routine that transposes the rotation matrix, but knowing that doesn't make it any easier to follow; it possibly makes it even scarier.)

This terseness is not remotely surprising given the space constraints of compiling code on a 32K micro, but I was still flummoxed. The fact that any kind of source code had been released at all was a kind of Holy Grail experience for me, but it ended up generating more questions than answers.

So I put it to one side and figured I'd probably never understand how this game worked.

### Paul Brink's annotated disc disassembly

The next breakthrough was the commentary by Paul Brink, whose annotated disassembly of the disc version of BBC Elite appeared on Ian Bell's site in 2014:

* [BBC disc source (docked) annotated by Paul Brink](http://www.elitehomepage.org/archive/a/d4090010.txt)
* [BBC disc source (flight) annotated by Paul Brink](http://www.elitehomepage.org/archive/a/d4090012.txt)

This was a _big_ improvement over the original source files, and like many others, I eagerly grabbed them and settled down with a cup of tea for some interesting reading. Unfortunately, I still couldn't really work out what was going on; it was like stumbling across a trail of breadcrumbs in the forest, but after heavy monsoonal rain. Every now and then something would seem to make some vague kind of sense, but then I'd come across this kind of thing:

```
\XX16 got INWK 9..21..26 up at LL15  . The ROTMAT has 18 bytes, for 3x3 matrix
\XX16_lsb[   0  2  4        highest XX16 done below is 5, then X
\            6  8 10        taken up by 6, Y taken up by 2.
\	    12 14 16=0 ?]
```

This refers to the same code as above, and is one of the more verbose explanations in the commentary. It's definitely a step up from `DO nodeX-Ycoords` and `TrnspMat`, but what are `XX16` and `INWK`? And `ROTMAT` - that's a rotation matrix, right? OK, so there _are_ matrices in there somewhere, which is no surprise given the 3D nature of the game. But it's still really hard to work out what's going on, and the code that this comment explains doesn't really make things any clearer than before:

```
	.LL42	\ ->  &4B04 \ DO nodeX-Ycoords their comment  \  TrnspMat 
A4 0B                   LDY &0B		\ XX16+2	      \ Transpose Matrix
A6 0C                   LDX &0C		\ XX16+3
A5 0F                   LDA &0F		\ XX16+6
85 0B                   STA &0B		\ XX16+2
A5 10                   LDA &10		\ XX16+7
```

We're still left with `XX16+2` and its friends, so this is essentially the source code, laid out differently, with cryptic hints scattered throughout, hints that seemed to be aimed at someone who already understands the basics, which I certainly didn't as I sat there, just as confused as ever.

By this time my tea had gone cold, so once again I put my dreams on hold and forgot about trying to unlock the secrets of Elite.

### Kieran Connell's elite-beebasm

In 2020, lockdown boredom led me to stumble across a [2018 post on the Stardot forums](https://stardot.org.uk/forums/viewtopic.php?t=15375) by Kieran Connell of the [Bitshifters Collective](https://bitshifters.github.io/). These guys do some incredibly clever things with BBC computers, and that's exactly what Kieran had done - he'd created [elite-beebasm](https://github.com/kieranhj/elite-beebasm), a port of the original BBC Elite source code from the super-terse BASIC files into the [BeebAsm assembler](https://github.com/stardot/beebasm).

Not only had he managed to drag the source code into some kind of human-compatible shape, but he'd also managed to pull apart the encryption process that hides Elite's code from prying eyes. He'd then created an equivalent system in Python, enabling modern computers to build an exact replica of the released version of Elite from the original source. This meant I could not only build a local version of Elite, but I could tweak the code to help work out what it did, which I figured would be a really useful way of working out how Elite weaves its magic.

That said, the source code still looked worryingly familiar:

```
.LL42
\DO nodeX-Ycoords
\TrnspMat
 LDY XX16+2
 LDX XX16+3
 LDA XX16+6
 STA XX16+2
 LDA XX16+7
 STA XX16+3
 STY XX16+6
 STX XX16+7
```

But at least I now had a buildable codebase I could work with, and that was real progress.

### At last, a fully documented version

Kieran's version gave me the leg-up that I needed to crack the problem. I started by copying Paul Brink's comments into Kieran's version, hoping that this would give me some clues to analysing the code, and some small, early glimmers of understanding gave me enough confidence to start poking my way through the bits of the game that had always fascinated me.

I started with the text token system, then worked out the split-screen mode, and then moved on to the universe generation... and by then I was completely hooked. Every little step forward, I felt like I was unpicking a bit more of the story of two young developers creating a modern-day masterpiece; if you squint carefully, you can almost sense where the whole starts to become greater than the sum of the parts. Elite is the coding equivalent of _A Day in the Life_, a mash-up between the Acorn world's very own Lennon and McCartney, with results that are just as seminal in their field. They say you should never meet your heroes, but grokking their source code... well, that's another matter altogether.

This repository is the result. The aim is that anyone with a basic knowledge of 6502 assembly language and simple trigonometry will be able to read through the source code and not only understand what's going on, but will also be able appreciate the beauty and elegance of this exceptional piece of 1980s programming.

It has been a privilege to unravel the intricacies of Elite. I hope you enjoy the ride.

## Versions of Elite

There are quite a few versions of Elite for the BBC range of computers. These are the main variants (the links will take you to playable web-based versions of the game):

* The [BBC Micro tape version](http://bbcmicro.co.uk/game.php?id=2484), which is the version I analyse on this site
* The [BBC Micro disc version](http://bbcmicro.co.uk/game.php?id=366), which is generally regarded as the canonical version
* An [enhanced version for the BBC Micro with a 6502 second processor](http://bbcmicro.co.uk/game.php?id=3471), with a four-colour space screen and no loading from disc
* An [enhanced version for the BBC Master](http://bbcmicro.co.uk/game.php?id=2088) that's almost identical to the second processor version
* The "executive version", which has a different font, an extended intro sequence and a maxed-out default commander (this version was not officially released but is available from [Ian Bell's site](http://www.iancgbell.clara.net/elite/bbc/))

I chose the tape version for this commentary for three reasons.

* First, that's the version that Kieran Connell converted to BeebAsm, which I forked to kickstart this project, so it made sense to stand on the shoulders of giants (as that's pretty much the whole theme of this project).

* Second, the tape version is the one I fell in love with back in 1984, and in which I reached the heady rank of Elite for the first time. I eventually upgraded to a disc drive, traded in my tape for the disc version and reached Elite all over again, but for me, the tape version is _the_ original game.

* Third, the tape version is the most impressive from a programming perspective. Sure, the disc version has loads more ships, a couple of missions, mining and military lasers and a proper docking computer, but the tape version takes the core of the game and squeezes it into a 32K BBC Micro, leaving very little free space. The disc version effectively loads a brand new program every time you launch or dock, but the tape version is 100% self-contained, and from a technical viewpoint, that's just incredible. How can such a sophisticated game squeeze into 32K? By being incredibly clever and incredibly efficient, and that's why the tape version is the most interesting one to pick apart. 
After all, the best things come in small packages...

I hope to document the extra features in the disc and second processor versions in a future project.

## Building Elite from the source

### Requirements

You will need the following to build Elite from source:

* BeebAsm, which can be downloaded from the [BeebAsm repository](https://github.com/stardot/beebasm). Mac and Linux users will have to build their own executable with `make code`, while Windows users can just download the `beebasm.exe` file.
* Python. Both versions 2.7 and 3.x should work.
* Mac and Linux users may need to install `make` if it isn't already present (for Windows users, `make.exe` is included in this repository).

For details of how the build process works, see the [Source files and the build pipeline](#Source-files-and-the-build-pipeline) section below. For now, let's look at how to build Elite from the source.

### Build targets

There are two main build targets available. They are:

* `build` - An unencrypted version
* `encrypt` - An encrypted version that exactly matches the released version of the game

The unencrypted version should be more useful for anyone who wants to make modifications to the game code. It includes a default commander with lots of cash and equipment, which makes it easier to test the game. As this target produces unencrypted files, the binaries produced will be quite different to the binaries on the original source disc, which are encrypted.

The encrypted version produces the released version of Elite, along with the standard default commander.

(Note that there is a third build target, `extract`, which is explained in the section below on [differences between the various source files](#Differences-between-the-various-source-files]).)

Builds are supported for both Windows and Mac/Linux systems. In all cases the build process is defined in the `Makefile` provided.

Note that the build ends with a warning that there is no `SAVE` command in the source file. You can ignore this, as the source file contains a `PUTFILE` command instead, but BeebAsm still reports this as a warning.

### Windows

For Windows users, there is a batch file called `make.bat` to which you can pass one of the three build targets above. Before this will work, you should edit the batch file and change the values of the `BEEBASM` and `PYTHON` variables to point to the locations of your `beebasm.exe` and `python.exe` executables.

All being well, doing one of the following:

```
make.bat build
```

```
make.bat encrypt
```

will produce a file called `elite.ssd`, which you can then load into an emulator, or into a real BBC Micro using a device like a Gotek.

### Mac and Linux

The build process uses a standard GNU `Makefile`, so you just need to install `make` if your system doesn't already have it. If BeebAsm or Python are not on your path, then you can either fix this, or you can edit the `Makefile` and change the `BEEBASM` and `PYTHON` variables in the first two lines to point to their locations.

All being well, doing one of the following:

```
make build
```

```
make encrypt
```

will produce a file called `elite.ssd`, which you can then load into an emulator, or into a real BBC Micro using a device like a Gotek.

### Verifying the output

The build process also supports a verification target that prints out checksums of all the generated files, along with the checksums of the files extracted from the original sources.

You can run this verification step on its own, or you can run it once a build has finished. To run it on its own, use the following command on Windows:

```
make.bat verify
```

or on Mac/Linux:

```
make verify
```

To run a build and then verify the results, you can add two targets, like this on Windows:

```
make.bat encrypt verify
```

or this on Mac/Linux:

```
make encrypt verify
```

The Python script `crc32.py` does the actual verification, and shows the checksums and file sizes of both sets of files, alongside each other, and with a Match column that flags any discrepancies. If you are building an unencrypted set of files then there will be lots of differences, while the encrypted files should mostly match (see the Differences section below for more on this).

The binaries in the `extracted` folder were taken straight from the [cassette sources disc image](http://www.elitehomepage.org/archive/a/a4080602.zip) (though see the [notes on `ELTB`](#eltb) below), while those in the `output` folder are produced by the build process. For example, if you don't make any changes to the code and build the project with `make encrypt verify`, then this is the output of the verification process:

```
[--extracted--]  [---output----]
Checksum   Size  Checksum   Size  Match  Filename
-----------------------------------------------------------
a88ca82b   5426  a88ca82b   5426   Yes   ELITE.bin
0f1ad255   2228  0f1ad255   2228   Yes   ELTA.bin
e725760a   2600  e725760a   2600   Yes   ELTB.bin
97e338e8   2735  97e338e8   2735   Yes   ELTC.bin
322b174c   2882  322b174c   2882   Yes   ELTD.bin
29f7b8cb   2663  29f7b8cb   2663   Yes   ELTE.bin
8a4cecc2   2721  8a4cecc2   2721   Yes   ELTF.bin
7a6a5d1a   2340  7a6a5d1a   2340   Yes   ELTG.bin
01a00dce  20712  01a00dce  20712   Yes   ELTcode.bin
99529ca8    256  99529ca8    256   Yes   PYTHON.bin
49ee043c   2502  49ee043c   2502   Yes   SHIPS.bin
c4547e5e   1023  c4547e5e   1023   Yes   WORDS9.bin
*             *  f40816ec   5426    *    ELITE.unprot.bin
*             *  1e4466ec  20712    *    ELTcode.unprot.bin
*             *  00d5bb7a     40    *    ELThead.bin
```

All the compiled binaries match the extracts, so we know we are producing the same final game as the release version.

## Source files and the build pipeline

The build process described above uses a five-stage pipeline. This pipeline is based on the original build process from the source disc, but it uses BeebAsm and Python instead of BBC BASIC.

The end product is an SSD disc image file that can be loaded by a BBC Micro with DFS, or an emulator like JSBeeb or BeebEm. The code produced is identical to the released version of the game (see the section on [verifying the output](#Verifying-the-output) for more details).

Each stage of the build pipeline uses one of the source files, so let's look at what's involved.

### 1. Compile the main game with `elite-source.asm`

BeebAsm loads `elite-source.asm` and creates the following files:

* `output/ELTA.bin`
* `output/ELTB.bin`
* `output/ELTC.bin`
* `output/ELTD.bin`
* `output/ELTE.bin`
* `output/ELTF.bin`
* `output/ELTG.bin`
* `output/PYTHON.bin`
* `output/SHIPS.bin`
* `output/WORDS9.bin`

`elite-source.asm` contains the main source code for Elite. It is based on the original BASIC source files, converted to BeebAsm assembler syntax. In the original build, this is what happens:

* `ELITEA` produces the `ELTA` binary
* `ELITEB` produces the `ELTB` binary
* `ELITEC` produces the `ELTC` binary
* `ELITED` produces the `ELTD` binary
* `ELITEE` produces the `ELTE` binary
* `ELITEF` produces the `ELTF` binary
* `ELITEG` produces the `ELTG` binary
* `DIALSHP` contains the `PYTHON` binary
* `SHPPRTE` produces the `SHIPS` binary
* `GENTOK` produces the `WORDS9` binary

So the BeebAsm process mirrors the original compilation steps pretty closely.

### 2. Concatenate the game code and compile the header with `elite-bcfs.asm`

BeebAsm then loads `elite-bcfs.asm`, which reads the following files:

* `output/ELTA.bin`
* `output/ELTB.bin`
* `output/ELTC.bin`
* `output/ELTD.bin`
* `output/ELTE.bin`
* `output/ELTF.bin`
* `output/ELTG.bin`
* `output/SHIPS.bin`

and creates the following:

* `output/ELTcode.unprot.bin`
* `output/ELThead.bin`

`elite-bcfs.asm` is the BeebAsm version of the BASIC source file `S.BCFS`, which is responsible for creating the "Big Code File" - i.e. concatenating the `ELTA` to `ELTG` binaries plus the `SHIPS` data into a single executable called `ELTcode`.

There is also a simple checksum test added to the start of the `ELTcode` file, but at this stage the compiled code is not encrypted, which is why it has `unprot` in the name. The original BASIC files contain encryption code that can't be replicated in BeebAsm, so we do this using Python in step 4 below.

### 3. Compile the loader with `elite-loader.asm`

Next, BeebAsm loads `elite-loader.asm`, which reads the following files:

* `images/DIALS.bin`
* `images/P.ELITE.bin`
* `images/P.A-SOFT.bin`
* `images/P.(C)ASFT.bin`
* `output/WORDS9.bin`
* `output/PYTHON.bin`

and creates the following:

* `output/ELITE.unprot.bin`

This is the BeebAsm version of the BASIC source file `ELITES`, which creates the executable Elite loader `ELITE`. This is responsible for displaying the title screen and planet, loading the dashboard image, setting up interrupt routines, configuring a number of operating system settings, relocating code to lower memory (below `PAGE`), and finally loading and running the main game.

The loader incorporates four image binaries from the `images` folder that, together with the code to draw the Saturn backdrop, make up the loading screen. It also incorporates the `WORDS9` and `PYTHON` data files that contains the game's text and the Python ship blueprint.

There are also a number of checksum and protection routines that EOR the code and data with other parts of memory in an attempt to obfuscate and protect the game from tampering. This can't be done in BeebAsm, so we do this using Python in the next step.

### 4. Calculate checksums and add encryption with `elite-checksum.py`

Next, the pipeline runs the Python script `elite-checksum.py`, which reads the following files:

* `output/ELTA.bin`
* `output/ELTB.bin`
* `output/ELTC.bin`
* `output/ELTD.bin`
* `output/ELTE.bin`
* `output/ELTF.bin`
* `output/ELTG.bin`
* `output/ELThead.bin`
* `output/SHIPS.bin`
* `output/ELITE.unprot.bin`

and creates the following:

* `output/ELTcode.bin`
* `output/ELITE.bin`

There are a number of checksum and simple EOR encryption routines that form part of the Elite build process. These were trivial to interleave with the assembly process in the original BASIC source files, but they've been converted into Python so they can run on modern machines (as not too many modern computers support BBC BASIC out of the box). Kieran Connell is the genius behind all this Python magic, so many thanks to him for cracking the code.

The script has two parts. The first part generates an encrypted version of the `ELTcode` binary, based on the code in the original `S.BCFS` BASIC source program:

* Concatenate all the compiled binaries
* Compute the checksum for the commander data
* Poke the checksum value into the binary
* Compute the checksum for all the game code except the header
* Poke the checksum value into the binary
* Encrypt all the game code except for the header using a cycling EOR value (0-255)
* Compute the final checksum for the game code
* Output the encrypted `ELTcode` binary

The second part implements the checksum and encryption functions from the `ELITES` BASIC source program to generate an encrypted `ELITE` binary:

* Reverse the bytes for a block of code that is placed on the stack
* Compute the checksum for `MAINSUM`
* Poke the checksum value into the binary
* Compute the checksum for `CHECKbyt`
* Poke the checksum value into the binary
* Encrypt a block of code by EOR'ing with the code to be placed on the stack
* Encrypt all the code destined for lower RAM by EOR'ing with the loader boot code
* Encrypt binary data (dashboard etc.) by EOR'ing with the loader boot code
* Output the encrypted `ELITE` binary

At the end of all this we have two encrypted binaries, one for the loader and another for the main game.

### 5. Assemble a bootable disc image with `elite-disc.asm`

Finally, BeebAsm loads `elite-disc.asm`, which reads the following files:

* `output/ELTcode.bin`
* `output/ELITE.bin`

and creates the following:

* `elite.ssd`

This script builds the final disc image. It copies the assembled `ELITE` and `ELTcode` binary files from the `output` folder to the disc image, and is passed as an argument to BeebAsm by the `Makefile` when it creates the disc image. The BeebAsm command is configured to add a `!Boot` file that `*RUN`s the `ELITE` binary, so the result is a bootable BBC Micro disc image that runs the tape version of Elite.

The disc image is called `elite.ssd`, and you can load it into an emulator, or into a real BBC Micro using a device like a Gotek.

### Log files

During compilation, details of every step are output in a file called `compile.txt`. If you have problems, it might come in handy, and it's a great reference if you need to know the addresses of labels and variables for debugging (or just snooping around).

## Differences between the various source files

### ELITEC

It turns out that the [cassette sources as text files](http://www.elitehomepage.org/archive/a/a4080610.zip) do not contain identical code to the binaries in the [cassette sources disc image](http://www.elitehomepage.org/archive/a/a4080602.zip). Specifically, there are some instructions in the `ELTC` binary that are different to the instructions in the `ELITEC.TXT` source file.

You can see these differences documented in the `WARP` routine in the `elite-source.asm` file. To find this, search the file for `Subroutine name:  WARP` and follow the comments for mentions of `ELITEC.TXT`.

The instructions included in `elite-source.asm` are those that match the binary files rather than `ELITEC.TXT`, to ensure that the build process produces binaries that match the released version of the game.

### ELTB

It also turns out there are two versions of the `ELITEB` BASIC source program on the [cassette sources disc image](http://www.elitehomepage.org/archive/a/a4080602.zip), one called `$.ELITEB` and another called `O.ELITEB`. These two versions of `ELITEB` differ by just one byte in the default commander data. This byte controls whether or not the commander has a rear pulse laser. In `O.ELITEB` this byte is generated by:

```
EQUB (POW + 128) AND Q%
```

while in `$.ELITEB`, this byte is generated by:

```
EQUB POW
```

The BASIC variable `Q%` is a Boolean flag that, if `TRUE`, will create a default commander with lots of cash and equipment, which is useful for testing. You can see this in action if you build an unencrypted binary with `make build`, as the unencrypted build sets `Q%` to `TRUE` for this build target.

The BASIC variable `POW` has a value of 15, which is the power of a pulse laser. `POW + 128`, meanwhile, is the power of a beam laser.

Given the above, we can see that `O.ELITEB` correctly produces a default commander with no a rear laser if `Q%` is `FALSE`, but adds a rear beam laser if `Q%` is `TRUE`. This matches the released game, whose executable can be found as `ELTcode` on the same disc. The version of `ELITEB` in the [cassette sources as text files](http://www.elitehomepage.org/archive/a/a4080610.zip) matches this version, `O.ELITEB`.

In contrast, `$.ELITEB` will always produce a default commander with a rear pulse laser, irrespective of the setting of `Q%`, so it doesn't match the released version.

The `ELTB` binary file in the `extracted` folder of this repository is the release version, so we can easily tell whether any changes we've made to the code deviate from the release version. However, the `ELTB` binary file on the sources disc matches the version produced by `$.ELITEB`, rather than the released version produced by `O.ELITEB` - in other words, `ELTB` on the source disc is not the release version.

The implication is that the `ELTB` binary file on the [cassette sources disc image](http://www.elitehomepage.org/archive/a/a4080602.zip) was produced by `$.ELITEB`, while the `ELTcode` file (the released game) used `O.ELITEB`. Perhaps the released game was compiled, and then someone backed up the `ELITEB` source to `O.ELITEB`, edited the `$.ELITEB` to have a rear pulse laser, and then generated a new `ELTB` binary file. Who knows? Unfortunately, files on DFS discs don't have timestamps, so it's hard to tell.

To support this discrepancy, there is an extra build target for building the `ELTB` binary as found on the sources disc, and as produced by `$.ELITEB`. You can build this version, which has the rear pulse laser, with:

  `make extract`

The `ELTcode` executable produced by this build target is different to the released version, because the default commander has the extra rear pulse laser. You can use the verify target to confirm this. Doing `make encrypt verify` shows that all the generated files match the extracted ones, while `make extract verify` shows that the all the generated files match the extracted ones except for `ELTB` and `ELTcode`.

## Next steps

* The commentary needs tidying up and clarifying in places - as it stands, this whole thing is basically a first draft that needs a fair amount of editing. There are one or two areas where the code is documented in terms of explaining what the code does, but I'm still trying to get my head around exactly how it works, so those areas still need addressing.

* I'm going to write more deep dive articles, as well as expanding the ones that are there. There is so much more to say about this masterpiece, from explaining the program flow to analysing how much of the code is devoted to each type of functionality.

* I'm in the process of creating a website that will take the documented source files from GitHub and generate a code-friendly website. This should make the source code easier to navigate, as GitHub can only display the main source as a raw file (it's too big). Having linked routine names and indexes into the code will make a big difference.

* I'm also hoping to analyse the disc and second processor versions, so I can document the code that differs from the tape version. That's a longer-term goal, though - first, I need to get the tape version polished up.

---

Right on, Commanders!

_Mark Moxon_ | August 2020