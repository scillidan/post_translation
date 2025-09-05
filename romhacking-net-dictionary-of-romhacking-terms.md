# [ROMhacking.net Dictionary of ROMhacking Terms](https://www.romhacking.net/dictionary)

## Overview

This is a dictionary we here at ROMhacking.net have put together just for you. It's not just any dictionary. It's a special dictionary specific to ROM hacking and includes all the ROM hacking terms we think you might need to know to really understand what people are talking about around here as well as understand the many documents we have here.

Don't be afraid of all the technical jargon, abbreviations, and language thrown around the community. Read and learn! Knowledge is power!

## List of Terms

### ASM Hacking

One with knowledge of the target system’s assembly language reprograms small portions of the target game’s code. This is done for reasons such as: resizing a window, adding or removing compression on text or graphics, or moving data to another location in the ROM where editing may be easier to perform.

### Assembly / ASM

Refers to game code written using only instructions directly convertable to binary code that the CPU of the target system can understand. This is a so-called “low-level” language that operates very close to hardware. Unlike high-level languages like “C” or “Basic”, ASM code cannot be easily ported to different hardware using another compiler.

### Binary

The numbering system used by all digital electronics hardware. It is also known as base 2 in mathmatical number system terms. There are only two digits available “0″ and “1″. For example the number “6″ in decimal is “110″ in binary. Digital electronics can only distinguish between “current” or “no current” (on/off) and so this numbering system is used. This is both for reliability and simplicity.

### Bit

The smallest possible information unit for storing data. It can only contain “0″ or “1″.
Cavespeak

When the Japanese font has been replaced by Latin characters the game displays the Japanese strings with Latin characters. This looks like trash as “Dg(k ,. PoÂ§_3-*”. That gibberish is called cavespeak by hackers.

### Dual/Multi Tile Encoding (DTE)

An encoding/compression scheme where one hexadecimal number is used to represent two or more letters. Adding or removing this requires ASM modification, though it is usually regarded as one of the easier ASM hacks.

### Emulator

A program that mimics a target hardware’s behavior to execute software of the target hardware on different hardware. A SNES Emulator for example lets you run games that were designed for the specific SNES hardware on your PC. They are important for ROM hacking, because they often deliver special debugging features, which make hacking easier. It is often also difficult/expensive to bring the altered ROM images back on a cartridge/Disc the actual Hardware can use.

### Font

The font is the collection of the characters a game can display as text. In English games this includes a-z, A-Z, 0-9 as well as any special characters like $,% and punctuation. These characters are stored as graphical data in the ROM. For Japanish-English translations the Japanese characters often must be replaced by Latin ones. Some games already include a Latin font though.

### Hex Editor

A program used to edit the binary data of a ROM image.

### Hexadecimal Numbering

In low-level programming languages like Assembler every number is stored in binary format because the hardware works with these. Since endless columns of zeros and ones are extremley difficult to work with, the data is usually displayed in hexadecimal format. Unlike decimal numbering (the system we use in daily math) the system has 16 digits from 0-F. The number “10″ for example is “A” in hex. To distinguish the systems from each other hexadecimal numbers are usually followed by “h” (e.g.: 3Bh) and decimal numbers by d (e.g.: 25d). In Programming Languages hexadecimal numbers are most times expressed with a “$” or “0x” in front of them (e.g.: $2C 0×2C).

### ISO / Disc Image

When you extract data from a optical medium or a hard drive, it is not called ROM but “Disc Image”. The short name “ISO” is common because the format the data is stored is called ISO9660. Unlike ROMs, ISOs do not only contain only the binary data but also information like directory structure or even several data tracks like separate audio.
Patch

A File that contains information on how to change data of a ROM image. Hacks are distributed as patches because the distribution of pre-altered ROM images is illegal due to several reasons. There are a few formats for patches like IPS, PPF and NINJA, which can be applied with appropriate tools, however ROM hackers also often write small executables that patch the game on their own.

### Pointer

Address value that references or “points” to other data. They are found in the file image and memory image of a program. Can be absolute or relative. Absolute means a file or memory address is specified; relative means the specified value is an offset from a base address.
Relative Search

A method of search for finding the hex codes that make up the characters in a game. The process involves entering a search string, usually a word, that the program will read & then decipher the differences of values between each character. For example, the search string “copyright” may be entered. The program will see that the value of the letter “c” is 12 less than the letter “o,” & that “o” is 1 less than “p.” The program will then find a hex string that matches the differences in value. The hex will look something like “A8B4B5BEB7AEACADB9,” where “A8″ equals “C,” “B4″ equals “O,” & “B5″ equals “P.” Note that relative searching for an alphabet will only work when the hex string is arranged in alphabetical order.

### ROM

Short for Read Only Memory. This is the chip inside a game cartridge in which the physical data of the game is stored. It cannot be altered by the hardware, thus called read only. It contains all data of the game including the actual game code that is executed, as well as graphics and music. A ROM hacker modifies this code to produce different effects such as having the text in English instead of Japanese. In ROM hacking the term is usually used as a short form of “ROM image”.

### ROM Hacking

Modifying the data in a ROM image to achieve such purposes as playing the game in a different language than intended, creating new levels for old games, or maybe playing with a different skill level than intended.

### ROM Image

A binary copy of the physical data that is stored in a ROM chip. There are a lot of different formats for each system in use. These images are altered and later patched by ROM hackers. To use the hacks on real hardware one must transfer it back onto a actual cartridge. You need special hardware to do this (usually called “Copiers” or “Flasher”)

### Script

All the raw text inside a ROM. This includes all story dialogue, monster names and skill names. Any text you see in the game is part of the script.

### Script Dumping

If a ROM hacker has located the text inside a ROM, it needs to be extracted to be translated. Replacing every string of a game individually by hand is extremley tedious if not impossible. Another problem is that this work can usually only be done by the hacker himself as it requires technical knowledge a translator does not have. Because of that the hacker writes a program that extracts all the text into a specially formatted text file. There are generic tools to do this as well but they usually don’t work on every game. This process is called “Script Dumping”. With this text file it is comfortable to translate a ROM and the translator does not need to know the technical details of the game.

### Script Editor/Formatter

A person, usually someone directly involved in the translation (hacker, translator, etc), who edits the translated scripts so that they will be displayed correctly in-game. This includes adding line-breaks where necessary and sometimes even rewriting portions of the script for either clarity or due to space limitations (or for making use of extra ROM space, allowing for a more “colorful” script). More advanced hackers handle the formatting portion automatically by either an in-game hack or a custom external script formatting utility.

### Script Insertion

After the Script has been dumped and translated the text must be reinserted into the ROM. This is usually more difficult than the extraction because the size of the script has most likely changed.

### Tracer

An emulator that can be set to report a record of all code executed during a specific timeframe (usually reporting to a window, or to a large text file). This will allow one with assembly skills to look for things like compressed text/graphics, screen layout information (such as window coordinates and sizes), and more.

### Translation Hacking

Hacking a game with the intent to translate it into a different language.

### Translator

A person who is fluent in the native language of the game being translated and hacked. This person receives the scripts (usually Japanese) from the hacker to translate. Some times his/her job may include formatting and proofreading the script.

### VWF (Variable Width Font):

Japanese Kanji are always square and of the same size. Latin characters though are of different sizes (e.g.: “w” and “i”). This produces 2 problems: First it just looks bad to have words displayed like “wi zar d.”. The other problem is that you waste a lot of space. Only a limited amount of letters can be displayed per line and in sentences with lots of “i”s and “l”s you’ll run out of space even though you could display a lot more. To get around these problems ROM hackers implement a so-called variable width font. They alter the text drawing code so small characters take only the space they really need. This is a complex hack that requires profound assembly skills.
