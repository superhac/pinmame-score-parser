# pinmame-score-parser

`pinmame-score-parser` is a small Python project for reading PinMAME NVRAM and extracting high scores, leaderboards, and other score-related records for supported ROMs.

The parser is driven by ROM-specific metadata in [`resources/roms.json`](/home/superhac/test/nvram_parser/resources/roms.json), which maps ROM names to the byte offsets and decoding rules needed to turn raw NVRAM bytes into readable results.

## What It Does

- Parses supported `.nv` files from PinMAME.
- Supports multiple score-decoding strategies, including BCD, digit-based, high-nibble, and mixed leaderboard layouts.
- Handles ROM aliases so compatible ROM revisions can share decoder definitions.
- Can also read certain `.ini`-based score files.
- Returns either a single score value or a structured list of parsed leaderboard entries.

## Project Layout

- [`score_parser.py`](/home/superhac/test/nvram_parser/score_parser.py): core parsing logic and formatter helpers
- [`resources/roms.json`](/home/superhac/test/nvram_parser/resources/roms.json): ROM decoder definitions and score metadata
- [`README.md`](/home/superhac/test/nvram_parser/README.md): project documentation

## Supported Data

The current ROM metadata file contains definitions for 247 ROM entries covering several decoder styles:

- `single_bcd_score`
- `single_bcd_score_x10`
- `single_digit_score`
- `single_digit_score_x10`
- `single_high_nibble_score`
- `single_high_nibble_score_x10`
- `leaderboard_bcd`
- `mixed_leaderboard`

Because the parser is metadata-driven, adding support for a new ROM is usually a matter of adding the correct offsets and decoder rules to [`resources/roms.json`](/home/superhac/test/nvram_parser/resources/roms.json).

## Usage

### Run the built-in example script

The `__main__` block in [`score_parser.py`](/home/superhac/test/nvram_parser/score_parser.py) contains a few hard-coded sample NVRAM paths. Running it will print a human-readable version of the parsed scores followed by JSON output:

```bash
python3 score_parser.py
```

### Use it from Python

```python
from score_parser import read_rom, format_result, result_to_jsonable

rom_name = "ww_lh6"
filename = "/path/to/ww_lh6.nv"

result = read_rom(rom_name, filename)

for line in format_result(rom_name, result):
    print(line)

print(result_to_jsonable(rom_name, result, filename))
```

## How It Works

1. The parser resolves the requested ROM name, including known aliases.
2. It loads that ROM's decoder configuration from [`resources/roms.json`](/home/superhac/test/nvram_parser/resources/roms.json).
3. It reads bytes from the configured offsets in the NVRAM file.
4. It decodes initials, scores, and special entry formats into structured Python data.
5. It can then format the result as readable text or JSON-friendly objects.

## Credits

This project builds on work from the virtual pinball community, and I want to explicitly acknowledge the people and projects that helped make this possible:

- `tomlogic` for [`pinmame-nvram-maps`](https://github.com/tomlogic/pinmame-nvram-maps), which is a valuable reference for PinMAME NVRAM structure research.
- Dna Disturber, creator of [PINemHi](https://www.pinemhi.com/), for the long-running work around pinball high-score tracking and preservation.

## Notes

- The current script does not yet expose a general command-line interface; the easiest way to use it today is by importing `read_rom(...)` or adjusting the example paths in [`score_parser.py`](/home/superhac/test/nvram_parser/score_parser.py).
- Some ROMs use aliases so multiple revisions can point to a shared definition.
