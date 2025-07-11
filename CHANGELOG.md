# splat Release Notes

### 0.34.1
* Deprecate `--stdout-only` cli flag. Changed output such that errors and warnings go to stderr and all other output goes to stdout.

### 0.34.0
* Add new global option `is_unsupported_platform`. If enabled, disable checks on platform option.
* Add new global option `allow_segment_overrides`. If enabled, allows to take precedence over the splat builtin platform segments via splat extension.

### 0.33.2
* Change `make_full_disasm_for_code` to output other sections for the TU in addition to .text.

### 0.33.1
* Fix `hasm` segments overwriting files already on disk

### 0.33.0

* BREAKING: Move splat config loading and parsing into "splat.util.conf" to modularize this part of the code and allow projects to interface with segment loading more gracefully.
  * `initialize_config` is now `splat.util.conf.load`. It takes `Path`s instead of `str`s for YAML paths. It will no longer `exit` if config merging fails, but will throw a `TypeError` which can be handled as desired by callers.
* Fix splat on Windows not using forward slashes on generated paths.
* Setup CI to be run on Windows and MacOS too.
* Fix output of incbin segments for SN64 projects
* New yaml option: `make_full_disasm_for_code`
  * Emit a full `.s` file for each `c`/`cpp` segment besides the generated `nonmatchings` individual functions.
  * Can be used to generate "target" or "expected" objects for asm diffing.
  * Also available as a cli parameter: `--make-full-disasm-for-code`
* `spimdisasm` 1.33.0 or above is now required.

### 0.32.3

* Fix "unrecognized YAML option" error if disassemble_all is provided via CLI and as a YAML option.
* Slightly speed up `bss` disassembly for projects over thousands of subsegments in the same top-level segment.

### 0.32.2

* Fix jumptable labels not being properly formatted with the given `symbol_name_format`.
* `spimdisasm` 1.32.0 or above is now required.

### 0.32.1

* Include subsegment information when aggregating split statistics.

### 0.32.0

* Tag `PSYQ` as a compiler that uses `j` instructions as branches.
  * Fixes incorrect detection of non existing functions under the mentioned compiler.
* Try to tell the users what segments are causing the overlapping issue.
* The overlap warning has been promoted to an error.

### 0.31.0

* Fix subsegments of auto `data` segments not being split.

### 0.30.1

* Fix `create_config` not handling unsafe path characters.

### 0.30.0

* New `asmtu` segment type.
  * Allows writing the disassembly of every section for a given object into the same assembly file.
  * Useful for dealing with symbols with local visibility.
* Cleanup some redundant code regarding duplicated `asm` segments.
* The global option `add_set_gp_64` now defaults to `False` on psx and ps2 platforms.
* Add compiler options supported by spimdisasm.
  * New compilers: `KMC` (n64), `EGCS` (iQue), `PSYQ` (PS1) and `MWCCPS2` (PS2)
  * It is highly recommended to use the specific compiler that the game uses (i.e. `KMC`) than just a general option like `GCC` because splat won't be able to adjust as best as it can.
* `spimdisasm` 1.31.0 or above is now required.
* Python 3.9 or above is now required.

### 0.29.0

* Fix bss/sbss asm files not being generated.
* Remove `...` as a valid rom start address for segments.

### 0.28.2

* New global option and per-segment option: `suggestion_rodata_section_start`
  * Allows to toggle the rodata section start suggestion that is emitted by inspecting the data section
* Avoid running spimdisasm on auto segments.
  * Should improve runtime performance and avoid giving redundant filesplit suggestions.
* A zero-sized bin segment is now considered a hard error.

### 0.28.1

* Fix a crash when listing zero-sized segments on the yaml.
  * It is discouraged to have zero sized segments on the yaml, but they may be needed on games that have a "segment address" table of any kind and shows the segments having a zero size.
  * This allows to have segment symbols for those kind of segments without having to go through linker script hacks.

### 0.28.0

* Minor version release for previous release's breaking change that should have had its own minor release (oopsh, yanked 0.27.4)
* BREKAING: Change the default value for `ld_generate_symbol_per_data_segment`. It defaults to `False` now.
* Improve `create_config` to avoid choking on SN64 games.
  * The results may not be completely accurate but should give a decent enough starting yaml.

### 0.27.3

* Fix a possible infinite recursion due to a segment being its own sibling.

### 0.27.2

* Added new global option `emit_subalign`. If disabled, subalign directives will not be emitted in the generated linker script. Enabled by default (no change in default behavior).

### 0.27.1

* Add new symbol attributes:
  * `function_owner`: Allows to force a rodata symbol to be migrated to the given function, skipping over the rodata migration heuristic.
  * `can_reference`: Allows toggling if the symbol is allowed to reference other symbols.
  * `can_be_referenced`: Allows toggling if the symbol is allowed to be referenced by other symbols.
* `spimdisasm` 1.29.0 or above is now required.

### 0.27.0

* BREAKING: Renamed `auto_all_sections` to `auto_link_sections` and documented its behavior.
* BREAKING: Removed redundant `N64Segment`, `PSXSegment`, `PSPSegment` stub classes. Any references to these should be instead to the base `Segment`
* Promoted `linker_offset` segment type to common, so it's now usable by all platforms.
* Added documentation for the remaining undocumented segment types and did some general doc tidying.
* Splat will now error when the last segment is `pad`, as this will not work as expected.
* Attempting to retrieve the `subalign` property of a non-top-level segment will now return an error.

### 0.26.2

* Fixed not being able to disable the `subalign` directive for a given segment.
* Removed bugged alignment check on image segments.
* splat will now error out if any of the following attributes is specified in a non top-level segment.
  * `subalign`
  * `ld_fill_value`.
* `spimdisasm` 1.28.1 or above is now required.

### 0.26.1

* Fix pairing text to other sections on cases where text is not be the first section.
* Fix some code that assumes that `.rodata` will always be present on the `sections_order` list.
* Fix `.text`/`.rdata` section pairing (hopefully).
* Emit an error if we try to migrate the rodata symbols to functions if the rodata section is not prefixed with a dot (ie `- [0x1234, rodata, some_file]` instead of `- [0x1234, .rodata, some_file]`)
  * Not prefixing the type with a dot would produce splat to both disassemble the rodata section to its own assembly file and to migrate the symbols to the corresponding functions, generating link-time errors and many headaches.
* Rewrite the `ld_legacy_generation` docs for clarity.

### 0.26.0

* Fixed the `subalign` segment property logic to be more straightforward
* Updated required versions of rabbitizier, n64img, and crunch64
* BREAKING: Removed `include_macro_inc` option, as it was never required and often detrimental

### 0.25.3

* Fix incorrect calculation for `$gp` value in create_config.py for PSX when immediate value is negative.

### 0.25.2

* Two new yaml options: `use_gp_rel_macro_nonmatching` and `use_gp_rel_macro`
  * Allows to toggle the use of `%gp_rel`s.
  * Not using `%gp_rel`s may be desirable for projects that use old assemblers that do not support said relocation parameter.
  * The assembler is free to choose the kind of relocation to use if no explicit relocation parameter is used for a given instruction, it may even expand the instruction into multiple instructions. To avoid this, it is the user's responsability to provide the symbol's information (like the symbol's size) to the assembler so it can pick the correct relocation.

### 0.25.1

* Added two new segment types: `gcc_except_table` and `eh_frame`.
  * Used by GCC to handle C++ exceptions.
* New yaml option: `asm_ehtable_label_macro`
  * Allows to specify the macro used by ehlabels, the ones generated by `gcc_except_table` references.

### 0.25.0

* BREAKING: Changed the default value of `use_legacy_include_asm` to false

### 0.24.7

* splat no longer creates unnecessary directories for asm

### 0.24.6

* Handle PS-X EXE header that includes .text/.data vram address

### 0.24.5

* New `use_src_path` option for incbins segments.
  * Allows to make the generated assembly files relative to the `src_path` directory instead of the default `data_path`.
* New yaml option: `global_vram_start` and `global_vram_end`.
  * Allow specifying that the global memory range may be larger than what was automatically detected.
  * Useful for projects where splat is used in multiple individual files, meaning the expected global segment may not be properly detected because each instance of splat can't see the info from other files (like in PSX and PSP projects).

### 0.24.4

* New yaml option: `matchings_path`
  * Determines the path to the asm matchings directory
  * This is used alongside `disassemble_all` to organize matching functions from nonmatching functions

### 0.24.3

* New yaml option: `ld_align_segment_start`
  * Allows specifying an alignment for the start of all the segments.
  * The alignment can be overriden or disabled per segment too.
* Add `visibility` attribute to symbols.
  * Allows to specify if a symbol should be declared as `local`, `weak`, etc in the disassembly.
* `spimdisasm` 1.26.0 or above is now required

### 0.24.2

* Fixed create_config to replace "/" in detected binary names with "_"

### 0.24.1

* Tweak the disassembler's configuration for PSP platform to improve assembly analysis.
  * Should improve function start/end detection.

### 0.24.0

* Added PSP as a new platform.
* `spimdisasm` 1.25.0 or above is now required
* `rabbitizer` 1.10.0 or above is now required

### 0.23.2

* Fixed a bug where auto segments insertion may not respect the proper ordering if there are linker_offset segments present.

### 0.23.1

* New `EEGCC` compiler option.
  * Provide specific adjustments for the GCC compiler used for the PS2 platform.
* `spimdisasm` 1.24.2 or above is now required.

### 0.23.0

* splat now checks if symbol names can be valid filepaths and produce an error if not.
  * This is checked because functions are written to their own files and the symbol name is used as the filepath.
  * There are two checks in place:
    * The resulting filename should not exceed 255 bytes, since most OSes impose that restriction.
    * It should not contain any of the following characters: `"<", ">", ":", '"', "/", "\\", "|", "?", "*"`
  * It is possible to specify a different filename and retain the symbol name by using the `filename` attribute for each symbol on the `symbol_addrs` file.
    * Make sure that the new specified `filename` does fit the listed requirements.
* Change `sbss` to properly work as a noload section.
  * To make it not behave as noload then turn off `ld_bss_is_noload`.
* `ld_bss_is_noload` is now `False` by default for `psx` projects.
* Allow to properly override `get_linker_section` and `get_section_flags` in `asm` and `hasm` files.
* Fix disassembling segments that only have bss.
* `ld_section_labels` was removed since it was a redundant list to `section_order`.
* Give a proper error message for missing sections on `section_order` during linker script generation.
* Allow configuring the `spimdisasm` section on extension segments (that inherit from `asm`, `data`, `rodata` or `bss` segments) before running the analisys on it.
  * This is done by overriding the `configure_disassembler_section` method.
* New ps2-specific segments:
  * `lit4` and `lit8`: "Literal" sections that only contain `float`s and `double`s respectively.
  * `ctor`: Data pointing to C++ global data initialization functions.
  * `vtables`: C++ vtables
* `spimdisasm` 1.23.0 or above is now required.

### 0.22.3

* Fix linker script generation not respecting other noload segments (like `.sbss`) when using `bss_contains_common`.

### 0.22.2

* Allow for `image_type_in_extension` to be overridden by subclasses of N64SegImg

### 0.22.1

* Fixed a bug where palettes with `global_id`s were reporing errors too eagerly

### 0.22.0: Palette Revamp

* The N64 ci/palette system has been rewritten to be more versatile and support a larger variety of configurations.
  * ci segments now have a "palettes:" argument, which can be a list of palettes or a single palette to be linked to the ci for extraction. The implicit value of `palettes:` is a one-element list containing the name of the ci, meaning palettes whose names match a ci will automatically be linked to the ci. Each palette linked to a ci will result in a separate png.
  * the `raster_name` field on palettes and the `palette` field on rasters no longer exist. Instead, rasters point to palettes via the `palettes:` property of the ci segment (or the final argument after width and height, if using list format).
  * palette segments can provide a `global_id` field, which serves as a globally searchable palette id. This can be used for cross-segment ci/palette linking.
  * added option `image_type_in_extension`, which puts the type of an image in the file extension. For example, with the setting enabled, an image named `texture` would export with filename `texture.ci4.png`.
* `spimdisasm` 1.21.0 or above is now required.

### 0.21.12

* Fixed issue that prevented symbols from being added to undefined_funcs_auto
* Add rodata file split suggestions for PSX.
  * This works by inspecting the expected alignment of jumptables. If a jumptable is not 8-aligned file-wise means there's a file split.

### 0.21.11

* Show an error on `create_config` if the format is not supported.
* Allow lib segment to be a dictionary with `object` and `section` options
* Allow lib segment to use `vram` option

### 0.21.10

* Allow the global `subalign` option to take `null` values

### 0.21.9

* Removed gc code in favor of decomp-toolkit, which is now linked-to from this project.

### 0.21.8

* `gfx` and `vtx` segments now support an optional `length` parameter, allowing splat to know their size

### 0.21.7

* New attribute for symbols: `allow_duplicated`
  * Allows to lift the duplicated symbol restriction for the specified symbols, allowing to have specific symbols that are not checked for shared vrams or names but keeping the check for everything else.

### 0.21.6

* Fix `bss_contains_common` option not being passed to "auto all" inserted sections.
* New yaml option: `ld_bss_contains_common`
  * Sets the default option for the `bss_contains_common` attribute of all segments.

### 0.21.5

* Fixed issue with Python 3.8 compatibility (oops)

### 0.21.4

* New yaml option: `hasm_in_src_path`
  * Tells splat to consider `hasm` files to be relative to `src_path` instead of `asm_path`.
* Remove some dead code.
* `auto_all_sections` is even more "auto" now, creating automatic entries for all .text segments.
  * Before, this feature did not function for a given type if you had one or more of that type manually specified elsewhere. Now, the automatic entries are still populated, even if you have manual ones as well.

### 0.21.3

* Updated version graphically

### 0.21.2

* Fix bugs involving segments not having proper end rom positions if followed by segments with "auto" rom addresses; splat will now skip over these properly

### 0.21.1

* Fix duplicated symbol resolution in symbol_addrs.txt file.
  * To allow symbols with the same name or vram address, splat was expecting both `rom` and `segment` to be specified. This check was changes so either one of them is required to disambiguate the symbol.
* Python dependencies per target architecture have been regrouped.
  * splat now requires an architecture to be specified during installation through pip.
  * This allows to avoid installing dependencies for an architecture that won't be used.
  * The new syntax to be used with pip is `python3 -m pip install -U splat64[arch1, arch2, etc]`, for example, `splat64[mips]`
  * Currently the only architecture supported is `mips`.
* The `main` function has been modularized and cleaned up.
  * When using splat as a library, allows the user to call each individual step of split without having to pass by its full monolithic `main` function.

### 0.21.0

* BREAKING: Extension segments will need adjustments to continue to work.
  * Due to splat working as a library now, absolute imports on extension segments no longer work.
  * Here's an example on how to fix this. Before version 0.21.0 you would have something like this

    ```py
    from util import log, options

    from segtypes.common.segment import Segment
    from segtypes.common.data import CommonSegData
    ```

    There are two ways to fix this, depending on how the user uses splat:
    * Installing splat as a Python package:

      If the user decides to use splat as a package instead of as a subrepo/submodule then fixing this issue becomes very easy, just prefix the imports with `splat.`:

      ```py
      from splat.util import log, options

      from splat.segtypes.common.segment import Segment
      from splat.segtypes.common.data import CommonSegData
      ```

    * Using splat as a submodule/subrepo:

      This option is a bit more complex since it requires relative imports, as if the extension segment were part of splat.

      splat will load extension segment as if it were in `segtypes/{PLATFORM}/{EXTENSION}.py`, so imports should be relative to that folder.

      Assuming the extension is for an `n64` project, the fixed version would look like this:

      ```py
      from ...util import log, options

      from ..common.segment import Segment
      from ..common.data import CommonSegData
      ```

* splat has been librarified!
  * splat can now be installed as a Python package and used as a library.
  * The normal way of invoking `./split.py` still works as usual.
* Installing the splat package allows to use it as a cli tool besides using it as a library.
  * Check `python3 -m splat --help` (or simply `splat --help`) to the options.
  * `splat split` has the same functionality as the plain `./split.py` script.
  * `splat create_config` has the same functionality as the plain `./create_config.py` script.
  * `splat capy`.
* DO NOT use splat as both an installed Python package and a submodule/subrepo.
  * This may be very problematic if the version of both splats go out of sync.
  * This warning is mainly for users that want use their own extension segments or use splat as a library.

### 0.20.1

* splat now uses [crunch64](https://github.com/decompals/crunch64) as a dependency for handling decompression of various formats, starting with Yay0 and MIO0
* Changed compression_type (and thus file extension) for the Mio0 segment to "MIO0" from "Mio0", accurately reflecting its true name
* Removed utility cli Yay0decompress.py and Mio0decompress.py scripts; see crunch64-cli for a much more performant (de)compression CLI tool

### 0.20.0

* Add a pad segment that advances the linker script instead of dumping a binary / generating an assembly file.
* Move the logic of writing the entry to the linker script from `LinkerWriter` to `LinkerEntry`
  * This allows to have custom behavior for an entry without needing to hardcode extra checks on `LinkerWriter`.
  * Extension segments can make a subclass of `LinkerEntry` and override its methods to have custom linker script behavior.
* New yaml option: `ld_generate_symbol_per_data_segment`
  * If enabled, the generated linker script will have a linker symbol for each data file.
  * Defaults to `True`.

### 0.19.7

* Ensure the directory exists when extracting a palette segment.
* Ensure the directory exists when writing the undefined funcs/syms files.
* Make `.splat` hidden folder to be relative to `base_path`

### 0.19.6

* The `*_END` linker symbol of every section for each segment is now aligned to the configured alignment by default.
* New yaml option: `ld_align_section_vram_end`
  * Allows to toggle aligning the `*_END` linker symbol of each section.
  * Defaults to `True`.

### 0.19.5

* The `*_VRAM_END` linker symbol for each segment is now aligned to the configured alignment by default.
* New yaml option: `ld_align_segment_vram_end`
  * Allows to toggle aligning the `*_VRAM_END` linker symbol.
  * Defaults to `True`.

### 0.19.4

* Fix `ld_fill_value` not accepting `null` as a valid value on the yaml

### 0.19.3

* New yaml option: `ld_bss_is_noload`
  * Allows to control if `bss` sections (and derivatived sections) will be put on a `NOLOAD` segment on the generated linker script or not.
  * Applies to all `bss` (`sbss`, `common`, `scommon`, etc) sections.
  * Defaults to `True`, meaning `bss` sections will be put on `NOLOAD` segments.

### 0.19.2

* `named_regs_for_c_funcs` (default True): Can be disabled to make c functions' disassembled functions contain numeric registers.

### 0.19.1

* Fixed disassembly of certain ps2 instructions to properly re-assemble in a compatible and matching way.

### 0.19.0: vram_classes

* New top-level yaml feature: `vram_classes`. This allows you to make common definitions for vram locations that can be applied to multiple segments. Please see the [documentation](docs/VramClasses.md) for more details!
  * Renamed `ld_use_follows` to `ld_use_symbolic_vram_addresses` to more accurately describe what it's doing
  * Renamed `vram_of_symbol` segment option to `vram_symbol` to provide consistency between the segment-level option and the vram class field.
  * Removed `appears_after_overlays_addr` symbol_addrs option in favor of specifying this behavior with `vram_classes`
* Removed `dead` symbol_addrs option
* A warning is now emitted when the `sha1` top-level yaml option is not provided. Adding this is highly recommended, as it prevents errors using splat in which the wrong binary is provided.

### 0.18.3

* splat now will emit a `FILL(0)` statement on each segment of a linker script by default, to customize this behavior use the `ld_fill_value` yaml option or the per-segment `ld_fill_value` option.
* New yaml option: `ld_fill_value`
  * Allows to specify the value of the `FILL` statement generated on every segment of the linker script.
  * It must be either an integer, which will be used as the parameter for the `FILL` statement, or `null`, which tells splat to not emit `FILL` statements.
  * This behavior can be customized per segment too.
* New per segment option: `ld_fill_value`
  * Allows to specify the value of the `FILL` statement generated for this specific top-level segment of the linker script, ignoring the global configuration.
  * If not set, then the global configuration is used.

### 0.18.2

* Fix rodata migration for `.rdata` sections (and other rodata sections that don't use the name `.rodata`)
* `spimdisasm` 1.18.0 or above is now required.

### 0.18.1

* New yaml options: `check_consecutive_segment_types`
  * Allows to turn off checking for segment types not being in a consecutive order
* New option for segments: `linker_section_order` and `linker_section`
  * `linker_section_order`: Allows overriding the section order used for linker script generation. Useful when a section of a file is not between the other sections of the same type in the ROM, for example a file having its data section between other files's rodata.
  * `linker_section`: Allows to override the `.section` directive that will be used when generating the disassembly of the corresponding section, without needing to write an extension segment. This also affects the section name that will be used during link time. Useful for sections with special names, like an executable section named `.start`

### 0.18.0

* `symbol_addrs` parsing checks:
  * Enforce lines contain a single `;`
  * Enforce no duplicates (same vram, same rom)

### 0.17.3

* Move wiki to the `docs` folder
* Added the ability to specify `find_file_boundaries` on a per segment basis
* Fix `cpp` segment not symbolizing rodata symbols properly

### 0.17.2

* Added more support for PS2 elf files

### 0.17.1

* New yaml options: `ld_sections_allowlist` and `ld_sections_denylist`
  * `ld_sections_allowlist`: A list of sections to preserve during link time. It can be useful to preserve debugging sections.
  * `ld_sections_denylist`: A list of sections to discard during link time. It can be useful to avoid using the wildcard discard. Note that this option does not turn off `ld_discard_section`.

### 0.17.0

* BREAKING: Linker script generation now imposes the specified `section_order`, which may not completely reflect the yaml order.
  * In case this new linker script generation can't be properly adapted to a repo, the old generation can be reenabled by using the `ld_legacy_generation` flag as a temporary solution. Keep in mind this option may be removed in the future.
* New yaml options related to linker script generation: `ld_partial_linking`, `ld_partial_scripts_path`, `ld_partial_build_segments_path`, `elf_path`, `ld_dependencies`
  * `ld_partial_linking`: Changes how the linker script is generated, allowing partially linking each segment. This allows for faster linking times when making changes to files at the cost of a slower build time from a clean build and loosing filepaths in the mapfile. This is also known as "incremental linking". This option requires both `ld_partial_scripts_path` and `ld_partial_build_segments_path`.
  * `ld_partial_scripts_path`: Folder were each intermediary linker script will be written to.
  * `ld_partial_build_segments_path`: Folder where the built partially linked segments will be placed by the build system.
  * `elf_path`: Path to the final elf target.
  * `ld_dependencies`: Generate a dependency file for every linker script generated, including the main linker script and the ones for partial linking. Dependency files will have the same path and name as the corresponding linker script, but changing the extension to `.d`. Requires `elf_path` to be set.
* New misc yaml options: `asm_function_alt_macro` and `ique_symbols`
  * `asm_function_alt_macro`: Allows to use a different label on symbols that are in the middle of functions (that are not branch targets of any kind) than the one used for the label for functions, allowing for alternative function entrypoints.
  * `ique_symbols` Automatically fills libultra symbols that are exclusive for iQue. This option is ignored if platform is not N64.
* New "incbin" segments: `textbin`, `databin` and `rodatabin`
  * Allows to specify binary blobs to be linked in a specific section instead of the data default.
  * If a `textbin` section has a corresponding `databin` and/or `rodatabin` section with the same name then those will be included in the same generated assembly file.
  * If a known symbol matches the vram of a incbin section then it will be emitted properly, allowing for better integration with the rest of splat's symbol system.
* `spimdisasm` 1.17.0 or above is now required.

### 0.16.10

* Produce an error if subsegments do not have an ascending vram order.
  * This can happen because bss subsegments need their vram to be specified explicitly.

### 0.16.9

* Add command line argument `--disassemble-all`, which has the same effect as the `disassemble_all` yaml option so will disamble already matched functions as well as migrated data.
  * Note: the command line argument takes precedence over the yaml, so will take effect even if the yaml option is set to false.

### 0.16.8

* Avoid ignoring the `align` defined in a segment for `code` segments

### 0.16.7

* Use `pylibyaml` to speed-up yaml parsing

### 0.16.6

* Add option `ld_rom_start`.
  * Allows offsetting rom address linker symbols by some arbitrary value.
    * Useful for SN64 games which often have rom addresses offset by 0xB0000000.
  * Defaults to 0.

### 0.16.5

* Add option `segment_symbols_style`.
  * Allows changing the style of the generated segment symbols in the linker script.
  * Possible values:
    * `splat`: The current style for segment symbols.
    * `makerom`: Style that aims to be compatible with makerom generated symbols.
  * Defaults to `splat`.

### 0.16.4

* Add `get_section_flags` method to the `Segment` class.
  * Useful for providing linker section flags when creating a custom section when making splat extensions.
  * This may be necessary for some custom section types, because sections unrecognized by the linker will not link its data properly.
  * More info about section flags: <https://sourceware.org/binutils/docs/as/Section.html#ELF-Version>

### 0.16.3

* Add `--stdout-only` flag. Redirects the progress bar output to `stdout` instead of `stderr`.
* Add a check to prevent relocs with duplicated rom addresses.
* Check empty functions only have 2 instructions before autodecompiling them.

### 0.16.2

* Add option `disassemble_all`. If enabled then already matched functions and migrated data will be disassembled to files anyways.

### 0.16.1

* Various changes so that series of image and palette subsegments can have `auto` rom addresses (as long as the first can find its rom address from the parent segment or its own definition)

### 0.16.0

* Add option `detect_redundant_function_end`. It tries to detect redundant and unreferenced functions ends and merge them together.
  * This option is ignored if the compiler is not set to IDO.
  * This type of codegen is only affected by flags `-g`, `-g1` and `-g2`.
  * This option can also be overriden per file.
* Disable `include_macro_inc` by default for IDO projects.
* Disable `asm_emit_size_directive` by default for SN64 projects.
* `spimdisasm` 1.16.0 or above is now required.

### 0.15.4

* Try to assign a segment to an user-declared symbol if the user declared the rom address.
  * Helps to disambiguate symbols for same-address overlays.

### 0.15.3

* Disabled `asm_emit_size_directive` by default for IDO projects.

### 0.15.2

* Various cleanup and fixes to support more liberal use of `auto` for rom addresses

### 0.15.1

* Made some modifications such that linker object paths should be simpler in some circumstances

### 0.15.0

* New options:
  * `data_string_encoding` can be set at the global level (or `str_encoding` at the segment level) to specify the encoding using when guessing and disassembling strings the the data section. In spimdisasm this value defaults to ASCII.
  * `rodata_string_guesser_level` changes the behaviour of the rodata string guesser. A higher value means more agressive guessing, while 0 and negative means no guessing at all. Even if the guesser feature is disabled, symbols manually marked as strings in the symbol_addrs.txt file will still be disassembled as strings. In spimdisasm this value defaults to 1.
    * level 0: Completely disable the guessing feature.
    * level 1: The most conservative guessing level. Imposes the following restrictions:
      * Do not try to guess if the user provided a type for the symbol.
      * Do no try to guess if type information for the symbol can be inferred by other means.
      * A string symbol must be referenced only once.
      * Strings must not be empty.
    * level 2: A string no longer needs to be referenced only once to be considered a possible string. This can happen because of a deduplication optimization.
    * level 3: Empty strings are allowed.
    * level 4: Symbols with autodetected type information but no user type information can still be guessed as strings.
  * `data_string_guesser_level` is similar to `rodata_string_guesser_level`, but for the data section instead. In spimdisasm this value defaults to 2.
  * `asm_emit_size_directive` toggles the size directived emitted by the disassembler. In spimdisasm this defaults to True.

### 0.14.1

* Fix bug, cod cleanup

### 0.14.0

* Add support for PSX's GTE instruction set

### 0.13.10

* New option `disasm_unknown` (False by default)
  * If enabled it tells the disassembler to try disassembling functions with unknown instructions instead of falling back to disassembling as raw data

### 0.13.9

* New segment option `linker_entry` (true by default).
  * If disabled, this segment will not produce entries in the linker script.

### 0.13.8

* New option `segment_end_before_align`.
  * If enabled, the end symbol for each segment will be placed before the alignment directive for the segment

### 0.13.7

* Severely sped-up linker entry writing by using a dict instead of a list. Symbol headers will no longer be in any specific order (which shouldn't matter, because they're headers).

### 0.13.6

* Changed CI image processing so that their data is fetched during the scan phase, supporting palettes that come before CI images.

### 0.13.5

* An error will be produced if a symbol is declared with an unknown type in the symbol_addrs file.
  * The current list of known symbols is `'func', 'label', 'jtbl', 'jtbl_label', 's8', 'u8', 's16', 'u16', 's32', 'u32', 's64', 'u64', 'f32', 'f64', 'Vec3f', 'asciz', 'char*', 'char'`.
  * Custom types are allowed if they start with a capital letter.

### 0.13.4

* Renamed `follows_vram_symbol` segment option to `vram_of_symbol` to more accurately reflect what it's used for - to set the segment's vram based on a symbol.
* Refactored the `appears_after_overlays_addr` feature so that expressions are written at the latest possible moment in the linker script. This fixes errors and warnings regarding forward references to later symbols.

### 0.13.3

* Added a new symbol_addrs attribute `appears_after_overlays_addr:0x1234` which will modify the linker script such that the symbol's address is equal to the value of the end of the longest overlay starting with address 0x1234. It achieves this by writing a series of sym = MAX(sym, seg_vram_END) statements into the linker script. For some games, it's feasible to manually create such statements, but for games with hundreds of overlays at the same address, this is very tedious and prone to error. The new attribute allows you to have peace of mind that the symbol will end up after all of these overlays.

### 0.13.2

* Actually implemented `ld_use_follows`. Oopz

### 0.13.1

* Added `ld_wildcard_sections` option (disabled by default), which adds a wildcard to the linker script for section linking. This can be helpful for modern GCC, which creates additional rodata sections such as ".rodata.xyz".
* Added `ld_use_follows` option (enabled by default), which, if disabled, makes splat ignore follows_vram and follows_symbols. This helps for fixing matching builds while being able to add infrastructure to the yaml for non-matching builds by just re-enabling the option.

### 0.13.0

* Automatically generate `INCLUDE_RODATA`/`#pragma GLOBAL_ASM` directives for non migrated rodata symbols when creating new C files.
* Non migrated rodata symbols will now only be produced if the C file has a corresponding rodata file with the same name and the C file has a `INCLUDE_RODATA`/`#pragma GLOBAL_ASM` directive referencing the symbol, similar to how functions are disassembled.
  * Because of this, the `partial_migration` attribute has lost its purpose and has been removed.
* Rodata symbol files are now included in the autogenerated dependency files too.

### 0.12.14

* New option: `pair_rodata_to_text`.
  * If enabled, splat will try to find to which text segment an unpaired rodata segment belongs, and it will hint it to the user.

### 0.12.13

* bss segments can now omit the rom offset.

### 0.12.12

* Try to detect and warn to the user if a gap between two migrated rodata symbols is detected and suggest possible solutions to the user.

### 0.12.11

* New disassembly option in the yaml: `allow_data_addends`.
  * Allows enabling/disabling using addends on all `.data` symbols.
* Three new options for symbols: `name_end`, `allow_addend` and `dont_allow_addend`.
  * `name_end`: allows to provide a closing name for any symbol. Useful for handwritten asm which usually have an "end" name.
  * `allow_addend` and `dont_allow_addend`: Allow overriding the global `allow_data_addends` option for allowing addends on data symbols.

### 0.12.10

* Allows passing user-created relocs to the disassembler via the `reloc_addrs.txt` file, allowing to improve the automatic disassembly.
* Multiple reloc_addrs files can be specified in the yaml with the `reloc_addrs_path` option.

### 0.12.9

* Added `format_sym_name()` to the vtx segment so it, too, can be extended

### 0.12.8

* The gfx and vtx segments now have a `data_only` option, which, if enabled, will emit only the plain data for the type and omit the enclosing symbol definition. This mode is useful when you want to manually declare the symbol and then #include the extracted data within the declaration.
* The gfx segment has a method, `format_sym_name()`, which will allow custom overriding of the output of symbol names by extending the `gfx` segment. For example, this can be used to transform context-specific symbol names like mac_01_vtx into N(vtx), where N() is a macro that applies the current "namespace" to the symbol. Paper Mario plans to use this, so we can extract an asset once and then #include it in multiple places, while giving each inclusion unique symbol names for each component.

### 0.12.7

* Allow setting a different macro for jumptable labels with `asm_jtbl_label_macro`
  * The currently recommended one is `jlabel` instead of `glabel`
* Two new options for symbols: `force_migration` and `force_not_migration`
  * Useful for weird cases where the disassembler decided a rodata symbol must (or must not) be migrated when it really shouldn't (or should)
* Fix `str_encoding` defaulting to `False` instead of `None`
* Output empty rules in generated dependency files to avoid issues when the function file does not exist anymore (i.e. when it gets matched)
* Allow changing the `include_macro_inc` option in the yaml

### 0.12.6

* Adds two new N64-specific segments:
  * IPL3: Allows setting its correct VRAM address without messing the global segment detection
  * RSP: Allows disassembling using the RSP instruction set instead of the default one
* PS2 was added as a new platform option.
  * When this is selected the R5900 instruction set will be used when disassembling instead of the default one.

### 0.12.5

* Update minimal spimdisasm version to 1.7.1.
* Fix spimdisasm>=1.7.0 non being able to see symbols which only are referenced by other data symbols.
* A check was added to prevent segments marked with `exclusive_ram_id` have a vram address range which overlaps with segments not marked with said tag. If this happens it will be warned to the user.

### 0.12.4

* Fixed a bug involving the order of attributes in symbol_addrs preventing proper range searching during calls to `get_symbol`

### 0.12.3: Initial Gamecube Support
Initial support for Gamecube disk images has been set up! Disassembly is not currently supported, and a more comprehensive explanation of Gamecube support will come once that is finished.

* The Symbol class is now hashable
* Added the ability for segments to specify a file path (`path`) to receive that file's contents as their split input
* The `generated_s_preamble` option now will be applied to data files created by spimdisasm
* Rewrote symbol range check code to be more efficient
* Fixed bug that allowed empty top-level segments of type `code`.
* Fixed progress bars to properly update their descriptions
* Fixed bug pertaining to symbols getting assigned to segments they shouldn't if their segment is given in symbol_addrs (`segment:`)

### 0.12.2
* Fixed bug where `given_dir` was possibly not a `Path`

### 0.12.1
* The constructor for `Segment` takes far fewer arguments now, which will affect (and hopefully simplify) any custom segments that are implemented.

* The new option `string_encoding` can be set at the global or segment level and will influence the encoding for strings in rodata during disassembly. The default encoding used is EUC-JP, as it was previously.

## 0.12.0: Performance Boost

In this release, we bring many performance improvements, making splat dramatically faster. We have observed speedups of 10-20x, though your results may vary.

* Linker script `_romPos` alignment statements now take a form that is friendlier to different assemblers.

* Fixed the default value of `use_legacy_include_asm` to be what it was before 0.11.2

### 0.11.2
* The way options are parsed and accessed has been completely refactored. The following option names have changed:

`linker_symbol_header_path` -> `ld_symbol_header_path`

`asm_endlabels` -> `asm_end_label`

Additionally, any custom segments or code that needs to read options will have to accommodate the new API for doing so. Options are now fields of an object named `opts` within the existing `options` namespace. Because the options are fields, `get_` is no longer necessary. To give an example:

Before: `options.get_asm_path()`

After: `options.opts.asm_path`

The clean_up_path function in linker_entry.py now uses a cache, offering a small performance improvement during the linker script writing phase.

### 0.11.1
* The linker script now includes a `_SIZE` symbol for each segment.
* The new `create_asm_dependencies`, if enabled, will cause splat to create `.asmproc.d` files that can inform a build system which asm files a c file depends upon. If your build system is configured correctly, this can allow triggering a rebuild of a C file when its included asm files are modified.
* Splat no longer depends directly on pypng and now instead uses [n64img](https://github.com/decompals/n64img). Currently, all image behavior uses the exact same code. Eventually, n64img will be implemented in C and support rebuilding images as well.

## 0.11.0: Spimdisasm Returns

Spimdisasm now handles data (data, rodata, bss) disassembly in splat! This includes a few changes in behavior:

* Rodata will be migrated to c files' asm function files when a .rodata subsegment is used that corresponds with an identically-named c file. Some symbols may not be automatically migrated to functions when it is not clear if they belong to the function itself (an example of which being const arrays). In this case, the `partial_migration` option can be enabled for the given .rodata subsegment and splat will create .s files for these unmigrated rodata symbols. These files can then be included in your c files, or you can go ahead and migrate these symbols to c and disable the `partial_migration` feature.

* BSS can now be disassembled as well, and the size of a code segment's bss section can be specified with the `bss_size` option. This option will tell splat how large the bss section is in bytes so BSS can properly be handled during disassembly. For bss subsegments, the rom address will of course not change, but the vram address should still be specified. This currently can only be done in the dict form of segment representation, rather than the list form.

Thanks again to [AngheloAlf](https://github.com/AngheloAlf) for adding this functionality and continuing to improve splat's disassembler.

## 0.10.0: The Linker Script Update

Linker scripts splat produces are now capable of being shift-friendly. Rom addresses will automatically shift, and ram addresses will still be hard-coded unless the new segment option `follows_vram` is specified. The value of this option should be the name of a segment (a) that this segment (b) should follow in memory. If a grows or shrinks, b's start address will also do so to accommodate it.

The `enable_ld_alignment_hack` option and corresponding behavior has been removed. This proved to add too much complexity to the linker script generation code and was becoming quite a burden to keep dealing with. Apologies for any inconvenience this may cause. But trust me: in the long run, it's good you won't be depending on that madness.

### 0.9.5
* Changes have been made to the linker script such that it is more shiftable. Rather than setting the rom position to hard-coded addresses, it increments the position by the size of the previous segment. Some projects may experience some alignment-related issues after this change. If specified, the new segment option `align: n` will add an `ALIGN(n)` directive for that section's linker segment.

### 0.9.4
* A new linker script section is now automatically created when the .bss section begins, using NOLOAD as opposed to the previous hacky rom rewinding we were previously doing. Additionally, `ld_section_labels` now includes `.rodata` by default.

### 0.9.3
* Added `add_set_gp_64` option (true by default), which allows controlling whether to add ".set gp=64" to asm/hasm files

### 0.9.2
* Added "palette" argument to ci4/ci8 segments so that segments' palettes can be manually specified

### 0.9.1
* Fixed a bug in which local labels and jump table labels could replace raw words in data blobs during data disassembly

## 0.9.0: The Big Update
### Introducing [spimdisasm](https://github.com/Decompollaborate/spimdisasm)!
* Thanks to [AngheloAlf](https://github.com/AngheloAlf), we now have a much better MIPS disassembler in splat! spimdisasm has much better hi/lo matching, much lower ram usage, and plenty of other goodies.

We plan to roll this out in phases. Currently, it only handles actual code disassembly. Later on, we will probably migrate our current data assembly code to use spimdisasm as well.

**NOTICE**: This integration has been tested on a variety of games and configurations. However, with any giant change to the platform like this, there are bound to be things we didn't catch. Please be patient with us as we handle these remaining issues. Though from what we've seen already, the slight bugs one may come across are totally worth the much improved disassembly.

### gfx segment type
* A new `gfx` segment type is available, which creates a c file containing a disassembled display list according to the segment's start and end offsets. Thanks to [Glank](https://github.com/glankk) and [Tharo](https://github.com/thar0/) for their work on [libgfxd](https://github.com/glankk/libgfxd) and [pygfxd](https://github.com/thar0/pygfxd/), respectively, for helping make this a possibility in splat.

### API breaking changes
* Some `Segment()` arguments have changed, which may cause extensions to break. Please see the `__init__` function for `Segment` for more details.

### symbol_addrs.txt changes
* symbol_addrs now supports the `segment:` attribute, which allows specifying the symbol's top-level segment. This can be helpful for symbol resolution when overlays use overlapping vram ranges. See `exclusive_ram_id` below for more information.

### Global options changes

The new `symbol_name_format` option allows specification of how symbols will be named. This can be set as a global option and also changed per-segment. `symbol_name_format_no_rom` is used when the symbol does not have a rom address (BSS).

 The following substitutions are allowed:

`$ROM` - the rom address of the symbol, hex-formatted and padded to 6 characters (ABCF10, 000030, 123456) (note: only for `symbol_name_format`, usage in `symbol_name_format_no_rom` will cause an error)

`$VRAM` - the vram address of the symbol, hex-formatted and padded to 8 characters (00030010, 00020015, ABCDEF10)

`$SEG` - the name of the top-level segment in which the symbol resides

The default values for these options are as follows

`symbol_name_format` : `$VRAM`

`symbol_name_format_no_rom` : `$VRAM_$SEG`

The appropriate prefix string will still automatically be applied depending on the type of the symbol: `D_` for data, `jtbl_` for jump tables, and `func_` for functions. This functionality may be customizable in the future.

----
The `auto_all_sections` option now should be a list of section names (`[".data", ".rodata", ".bss"]` by default) indicating the sections that should be linked from .o files built from source files (.c or asm/hasm .s files), when no subsegment explicitly indicates linking this type of section.

For example, if any subsegment of a code segment is of segment type `data` or `.data`, the `.data` section from all `c`/`asm`/`hasm` subsegments will not be linked unless explicitly indicated with a relevant `.data` subsegment.

Previously, this option was a bool, and it enabled this feature for all sections specified in `section_order`. Now, the desired sections must be specified manually. The default value for this option retains previous behavior.

----
The new `mips_abi_float_regs` option allows for changing the format of float registers for MIPS disassembly. The default value does not change any prior behavior, but `o32` is heavily encouraged and may become the default option in the future. For more information, see this [great writeup](https://gist.github.com/EllipticEllipsis/27eef11205c7a59d8ea85632bc49224d).

----
The new `gfx_ucode` option allows for specifying the target for the graphics macro format, which is used in the gfx segment type. The default is `f3dex2`.


### Segment options changes

The new `exclusive_ram_id` segment option allows specifying an identifer that will prevent the segment from seeing any symbols from other segments with the same identifer. This is useful when multiple segments are mapped to the same vram address at runtime and should never be able to refer to each other's symbols. Setting all of these segments to have the same value for this option will prevent their symbols from clashing / meshing unexpectedly.

----

The `overlay` setting on segments has been removed. Please see `symbol_name_format` above for info on how to influence the names of symbols, which can be applied at the segment level as well as the global level.

----
## 0.8.0: Arbitrary Section Order
* You can now use the option `section_order` to define the binary section order for your target binary. By default, this is `[".text", ".data", ".rodata", ".bss"]`. See options.py for more details
* Documented all options in options.py
* Support for SN64 games (thanks Wiseguy!)
* More consistent handling of paths (thanks Mkst!)
* Various other cleanup and fixes across the board

### 0.7.10: WIP PSX support
* WIP PSX support has been added, thanks to @mkst! (https://github.com/ethteck/splat/pull/99)
  * Many segments have moved to a "common" package
  * Endianness of the input binary is now a configurable option
* Linker hack restored but is now optional and off by default

### 0.7.9
* Finally removed the dumb linker section alignment hack
* Added version number to output on execution

### 0.7.8

* Fixed a bug relating to a linker section alignment hack (thanks Wiseguy!)
* Fixed a bug in linker_entry.py's clean_up_path that should make this function more versatile (thanks Wiseguy!)

### 0.7.7

* Disassembly now reads the `size` property of a function in symbol_addrs.txt to disassemble `size / 4` number of instructions. Feel free to specify the size of your functions in your symbol_addrs file if splat's disassembly is chopping a function too short or making a function too long.

### 0.7.6

* Fixed a bug involving detection of defined functions in c files for GLOBAL_ASM-using projects
* Added options to disable the creation of undefined_funcs/syms_auto.txt files
* Added a Vtx segment type for creating c files containg model vertex data in the n64 libultra Vtx format
* Added a `cpp` segment type which is identical to `c` but looks for a file with the extension ".cpp" instead of ".c".

### 0.7.5: all_ types and auto_all_sections

If you have a group segment with multiple c files and want splat to automatically create linker entries at a given position for each code object (c, asm, hasm) in the segment, you can use an `all_` type for that section. For example, you can add `[auto, all_bss]` as the last subsegment in a segment. This will direct splat to create a linker entry for each code object in the segment. This saves a lot of time when it comes to manually adding .bss subsegments for bss support, for example. The same thing can be done for data and rodata sections, but note this should probably be done later into a project when all data / rodata is migrated to c files, as the `all_` types lose the rom positioning information that's necessary for splat to do proper disassembly.

The `auto_all_sections` option, when set to true, will automatically add `all_` types into every group. This is only done for a section in a group if no other manual declarations for that section exist. For example, if you have 30 c files in a group and a .data later on for one of them, `auto_all_sections` will not interfere with your `.data` subsegment. If you remove this, however, splat will use `auto_all_sections` to implicitly `.data` subsegments for all of your code objects behind the scenes. This feature is again particualrly helpful for bss support, as it will create bss linker entries for every file in your project (assuming you don't have any manual .bss subsegments), which eliminates the need to create dummy .bss subsegments just for the sake of configuring the linker script.

### 0.7.2

* Data disassembly changes:
  * String detection has been improved. Please send me false positives / negatives as you see them and I can try to improve it further!
  * Symbols in a data segment pointed to by other symbols will now properly be split out as their own symbols

### 0.7.1

* Image segment changes:
  * Added `flip_x` and `flip_y` boolean parameters to replace `flip`.
    * `flip` is deprecated and will produce a warning when used.
    * Fixed flipping of `ci4` and `ci8` images.
  * Fixed `extract: false` (and `start: auto`) behaviour.

## 0.7.0: The Path Update

* Significantly better performance, especially when using the cache feature (`--use-cache` CLI arg).
* BREAKING: Some cli args for splat have been renamed. Please consult the usage output (-h or no args) for more information.
  * `--new` has been renamed to `--use-cache`
  * `--modes` arg changes:
    * Image modes have been combined into the `img` mode
    * Code and ASM modes have been combined into the `code` mode
* BREAKING: The `name` attribute of a segment now should no longer be a subdirectory but rather a meaningful name for the segment which will be used as the name of the linker section. If your `name` was previously a directory, please change it into a `dir`.
* BREAKING: `subsections` has been renamed to `subsegments`
* New `dir` segment attribute specifies a subdirectory into which files will be saved. You can combine `dir` ("foo") with a subsegment name containing a subdirectory ("bar/out"), and the paths will be joined (foo/bar/out.c)
  * If the `dir` attribute is specified but the `name` isn't, the `name` becomes `dir` with directory separation slashes replaced with underscores (foo/bar/baz -> foo_bar_baz)
* BREAKING: Many configuration options have been renamed. `_dir` options have been changed to the suffix `_path`.
* BREAKING: Assets (non-code, like `bin` and images) are now placed in the directory `asset_path` (defaults to `assets`).
* Linker symbol header generation. Set the `linker_symbol_header_path` option to use.
  * `typedef u8[] Addr;` is recommended in your `common.h` header.
* You can now provide `auto` as the `start` attribute for a segment, e.g. `[auto, c, my_file]`. This causes the segment to not be extracted, but linked. This feature is intended for modding.
* Providing just a ROM address but no type or name for a segment is now valid anywhere in `segments` or `subsegments` rather than just at the end of the ROM. It specifies the end of the previous segment for types that need it (`palette`, `bin`, `Yay0`) and causes the linker to simply write padding until that address.
* The linker script file is left untouched if the contents have not changed since the previous split.
* You can now group together segments with `type: group` (similar to `code`). Note that any ASM or C segments must live under a `type: code` segment, not a basic `group`.

### 0.6.5: Bugfixes, rodata migration, and made options static

If you wrote a custom extension, options should be imported and statically referenced
`from util import options`

see options.py for more info on how to now get and set options

BREAKING: vram can only be specified on a segment if the segment is defined as a dict in the config

### 0.6.3: More refactoring
**Breaking Change**: The command line args to split.py have changed. Currently, only the config path is now a required argument to splat. The old `rom` and `outdir` parameters are now optional (`--rom`, `--outdir`). Now, you can add rom and out directory paths in the yaml.

The `out_dir` option specifies a directory relative to the config file. If your config file is in a subdirectory of the main repo, you can set `out_dir: ../`, for example.

The `target_path` option spcifies a path to the binary file to split, relative to the `out_dir`. If your `baserom.z64` is in the top-level of the repo, you can set `target_path: baserom.z64`, for example.

### 0.6.2: subsegments
I've begun a refactor of the code "files" code, which makes everything cleaner and easier to extend.
There's also a new option, `create_new_c_files`, which disables the creation of nonexistent c files. This behavior is on by default, but if you want to disable it for any reason, you now have the option to do so.

I am also working on adding bss support as well. It should almost be all set, aside from the changes needed in the linker script.

**Breaking change**: The `files` field in `code` segments should now be renamed to `subsegments`.

### 0.6.1: `assets_dir` option

This release adds a new `assets_dir` option in `splat.yaml`s that allows you to override the default `img`, `bin`, and other directories that segments output to.

Want to interdisperse split assets with your sourcecode? `assets_dir: src`!
Want to have all assets live in a single directory? `assets_dir: assets`!

## 0.6: The Symbol Update
Internally, there's a new Symbol class which stores information about a symbol and is stored in a couple places during disassembly. Many things should be improved, such as reconciling symbols within overlays, things being named functions vs data symbols, and more.

**Breaking change**: The format to symbol_addrs.txt has been updated. After specifying the name and address of a symbol (`symbol = addr;`), optional properties of symbols can be set via inline comment, space delimited, in any order. The properties are of the format `name:value`
  * `type:` supports `func` mostly right now but will support `label` and `data` later on. Internally, `jtbl` is used as well, for jump tables. Splat uses type information during disassembly to disambiguate symbols with the same addresses.
  * `rom:` is for the hex rom address of the symbol, beginning with `0x`. If available, this information is extremely valuable for use in disambiguating symbols.
  * `size:` specifies the size of the symbol, which splat will use to generate offsets during disassembly. Uses the same format as `rom:`

**function example**: `FuncNameHere = 0x80023423; // type:func rom:0x10023`

**data example**: `gSomeDataVar = 0x80024233; // type:data size:0x100`

## 0.5 The Rename Update
* n64splat name changed to splat
  * Some refactoring was done to support other platforms besides n64 in the future
    * New `platform` option, which defaults to `n64`
  * This will cause breaking changes in custom segments, so please refer to one of the changes in one of the n64 base segments for details
* Support for custom artifact paths
  * New `undefined_syms_auto_path` option
  * New `undefined_funcs_auto_path` option
  * New `cache_path` option
  * (All path-like options' names now end with `_path`)
