# dwarf-synthesis

A tool for automatic synthesis of DWARF.

The purpose of this tool is to take any given binary program or library,
examine its assembly code and, based solely on that, generate the corresponding
`.eh_frame` DWARF data.

## Dependencies

This tool relies on

* `opam` set up with an OCaml 4.05 switch (see below),
* [BAP](https://github.com/BinaryAnalysisPlatform/bap) version 1.5 as of today,
  which is available through OPAM;
* `objcopy`, often packaged as `binutils`
* `libelf`
* `libdwarf`
* `libdwarfw`, packaged as submodule

### Installing dependencies

You should be able to easily install `objcopy` (`binutils`), `libelf`,
`libdwarf` and `opam` via your package manager.

Once [`opam` is set up](https://opam.ocaml.org/doc/Install.html), you will have
to use an `OCaml 4.05` opam switch. If you are not familiar with opam, we
recommand using a fresh switch:

```bash
# If you never used opam before on this session:
opam init
eval $(opam config env)

opam switch create dwarf-synthesis ocaml-base-compiler.4.05.0  # With opam 2
# OR
opam switch dwarf-synthesis --alias-of 4.05.0                  # With opam 1.x

# Always:
eval $(opam config env)
opam install bap
```

## Compiling

Simply run `make` to compile all the necessary tools, including compiling and
installing the BAP plugin `dwarfsynth`.

## Running with a wrapper script

To generate an `.eh_frame` section for some binary `foo.bin` and write the
output as `foo.eh.bin`, you can run

```
./synthesize_dwarf foo.bin foo.eh.bin
```

You can also omit the second parameter to simply overwrite `foo.bin`.

## Running by hand

If you want, for some reason, to run by hand the multiple components, you can
follow this procedure (by using more appropriate file names, and, possibly, a
temporary directory -- see `mktemp -d`).

### Running the BAP plugin

`bap prog_to_analyze.bin -p dwarfsynth --dwarfsynth-output tmp.marshal`

### Running `ml_dwarf_write`

You can get a help text with `./ml_dwarf_write.bin`. Otherwise, you can run

```
./ml_dwarf_write.bin tmp.marshal prog_to_analyze.bin eh_frame_section`
```

### Stitching the section into a binary

```
objcopy --add-section .eh_frame=eh_frame_section prog_to_analyze.bin prog_to_analyze.eh.bin
```

## Commonly used commands

### List a binary sections

`objdump -h blah.bin`

### Strip a binary of its `eh_frame`

`objcopy --remove-section '.eh_frame' --remove-section '.eh_frame_hdr' blah.bin`
