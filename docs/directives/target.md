## `target` directive

`target` is a block directive which defines a high level product of compilation process. This is typically a final "TAP" file, containing bootstrap program written in basic, plus all compiled machine code. The directive must be used in the global scope (e.g. cannot be declared inside another directive)

The directive has following format:
```
target TargetName
   [.meta Key [Value]]
   Bootstrap DSL
end
```

Your source codes may define multiple targets, each with unique name. In this case, the first defined target is implicitly used, or you can select one with using `.meta default` attribute, or by compiler's `--target` command line option. Multiple targets are typically useful for situations, when you want to build and test just a fragment of your final program. You can specify as much additional targets as you want.

### `.meta` attributes:

- `format tape`, defines format of final package, produced in the target
   - `tape` is default format and product is "TAP" file
   - *Note that you can override this attribute by using `--targetFormat` compiler's command line parameter*

- `program NameOfProgram` - changes file name for produced bootstrap BASIC program. 
   - If omitted then `TargetName` as program name is used
 
- `reboot pure48k | basic | basic+` - changes expected mode after machine's reboot:
   - `pure48k` - Speccy must be rebooted to pure-48k mode, with 128k paging disabled
   - `basic` - Speccy must be rebooted into 48k BASIC, but with 128k paging enabled
   - `basic+` - 128k BASIC loader is used
   - *Note that compiler has no way to force how TAP file is loaded and therefore the attribute is just a hint for possible future debugger, or for developers who read your source codes :)*

- `default` - makes this target as default in case that multiple targets are defined

- `generator options` - you can specify a comma separated list of options for BASIC generator:
    - `pretty` - if used, then a more human readable BASIC loader will be generated. Normally, the whole loader is generated to a single BASIC line. In pretty mode, each DLS statement will occupy its own line.
    - `shadowNumbers` - if used, then all numbers will have `0` as a textual representation. Check [bastapir](https://github.com/hvge/bastapir) documentation for details. 

### Bootstrap DSL

Bootstrap DSL is a [domain specific language](https://en.wikipedia.org/wiki/Domain-specific_language), which defines how's your program, or its parts, loaded into the memory and executed. Following commands are supported:

- `colors BORDER, PAPER, INK` - changes screen colors
- `clear ADDRESS` - changes RAMTOP value for BASIC. The `ADDRESS` may be reference to any object or label defined in global scope.
- `load OBJECT [, ADDRESS]` - loads `OBJECT` into memory. If `ADDRESS` parameter is not specified, then the object's begin address is used. 
   - You can interpret this as `LOAD "" CODE` vs `LOAD "" CODE address` in regular basic.
- `call ADDRESS` - executes machine code at specified ADDRESS. The `ADDRESS` may be reference to any object or label defined in global scope.
   - This is translated to `RANDOMIZE USR address` BASIC command 
- `basic` - switches DSL into BASIC syntax. The actual BASIC program lays between `begin` and `end` keywords, where `end` also ends parent `target` directive. See next chapter for details.

This simple set of commands defines both, how TAP file is constructed and how routines are executed. For example, order of `load` commands also defines order of binary files in final TAP file. 
   

### Inline BASIC

In your Bootstrap DSL you can switch to ZX Spectrum BASIC syntax and define your own target's bootstrap. To do this, use `basic` command:

```
basic
    [.var ImportedSymbol [, OtherSymbol]]
    [.var AnotherSymbol = value]
begin
    ... your basic program in 'bastapir' compatible format ...
end
```

Only one `basic` command can be used in target specification and must be declared before everything else. If `basic` is used, then all other commands except `load` are forbidden. The actual BASIC source code lays between `begin` and `end` keywords. The program syntax is compatible with [bastapir](https://github.com/hvge/bastapir) project.

The inline BASIC program can use symbols from assembler. To do this, you need to declare what symbol has to be imported into BASIC. After that, each symbol can be accessed in BASIC as `@SymbolName`. Only numeric symbols are supported.

### Example

```
target Demo
    .meta program   LowerState
    .meta output    tape
    .meta reboot    basic
    .meta default
    .meta generator pretty
    
    ; border, paper, ink
    colors  0, 0, 0 

    ; define RAMTOP
    clear   min(SplashScreen, Part1, Part2, Part3) - 1
    
    ; load & execute loading screen
    load    SplashScreen
    call    SplashScreen
    
    ; Load parts
    load    Part1
    load    Part1Data
    call    Part1
    load    Part2
    call    Part2
    load    Part3
    call    Part3
end

target Test_Splash
    colors  0, 0, 0 
    clear   SplashScreen - 1
    load    SplashScreen
    call    SplashScreen
end
```


A similar example, but with using inline BASIC program:

```
target Demo
    .meta program   LowerState
    .meta output    tape
    .meta reboot    basic
    .meta default
    
    ; Inline BASIC loader
    basic 
        .var SplashScreen
        .var Part1, Part2, Part3
        .var RAMTOP = min(SplashScreen, Part1, Part2, Part3) - 1
    begin
    @autostart:
        paper 0: ink 0: border 0: clear @RAMTOP
        load "" code: randomize usr @SplashScreen
        load "" code: load "" code: randomize usr @Part1
        load "" code: randomize usr @Part2
        load "" code: randomize usr @Part3
    end
    
    ; You still have to define order of files in final tape
    load    SplashScreen
    load    Part1 
    load    Part1Data
    load    Part2
    load    Part3
end

target Test_Splash
    basic
        .var SplashScreen
    begin
    @autostart: 
        # Note that @SplashScreen-1 is actually a statement in BASIC.
        # It'll be expanded to something like 25000-1
        10 paper 0: ink 0: border 0; clear @SplashScreen-1
        20 load "" code: randomize usr @SplashScreen
    end
end
```

As you can see, the Bootstrap DSL approach is more convenient, so we recommend to use Inline BASIC only when you need an additional customization (like some PRINTs), or if autogenerated BASIC is too big.

