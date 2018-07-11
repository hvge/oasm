## OASM

OASM (stands for "object assembler") is a next gen Z80 assembler, targetting ZX Spectrum demosceners.

### Project goals

- Make programming for ZX spectrum great again!
- Compatible with already estabilished assemblers
  - This goal is not about 100% compatibility, but more about migration your old routines to OASM. 
  - If you don't like my concept of objects, you'll still be able to write code as before, with a bare minimum object-related stuff. 
- Cover a whole demo development process 
  - Basically, your `asm` file should cover a whole process of demo creation, from writing routines, over data compression, up to packaging product into final TAP file.
- Motivate developers to build a huge library of shared, open source routines
  - Imagine a library containing all player routines for well known music trackers, simply accessible just by importing it's github repository
- Support for macOS, linux & windows platforms

#### Future goals

- Make API for external debuggers. In far far future, I would like to write a nice graphical debugger for macOS, but this requires a kind of debug symbols server process to be developed first.
- Make it multiplatform. The parser will not be Z80 centric, so it may be relatively easy to add support for another CPU architectures (like 6502).

### Current status

Right now I'm collecting ideas & trying to specify a final language syntax. You can check [documentation page](docs/readme.md) for details. Feel free to ask me about more details at durech.juraj@gmail.com. 
