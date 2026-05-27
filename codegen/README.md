# PrintConfig Codegen -- Implementation Status

## Pipeline Overview

```
src/proto/generated/*.proto  (22 files, grouped by Tab.cpp page)
        |
        v
protoc  (compile)
        |
        v
config.desc  (binary descriptor set)
        |
        v
config_codegen.py  (codegen)
        |
        +-- codegen/generated/PrintConfigDef_generated.cpp
        +-- codegen/generated/Preset_options_generated.cpp
        +-- codegen/generated/Invalidation_generated.cpp
        +-- codegen/generated/OptionKeys_generated.cpp
        +-- codegen/generated/TabLayout_generated.cpp
```

## Tools

| Tool | Purpose |
|--|--|
| `tools/config_codegen.py` | Reads compiled proto descriptor, generates 4 C++ source files |
| `tools/validate_codegen.py` | Compares generated output against original `PrintConfig.cpp` for fidelity |
| `tools/run_codegen.py` | Convenience script: runs the full pipeline (compile -> generate -> validate) |
| `cmake/modules/ConfigCodegen.cmake` | CMake integration with `codegen_config` and `validate_config` targets |

## Generated Outputs

| File | Replaces | Status |
|--|--|--|
| `codegen/generated/PrintConfigDef_generated.cpp` | `init_fff_params()` body (~6000 lines) | Generated, not yet wired in |
| `codegen/generated/Preset_options_generated.cpp` | `s_Preset_*_options` string vectors in `Preset.cpp` | Generated, not yet wired in |
| `codegen/generated/Invalidation_generated.cpp` | `opt_key == "..."` chains in `Print.cpp` | Generated, not yet wired in |
| `codegen/generated/OptionKeys_generated.cpp` | Extruder/filament key lists | Generated, not yet wired in |
| `codegen/generated/TabLayout_generated.cpp` | `add_options_page` / `new_optgroup` / `append_single_option_line` calls in `Tab.cpp` | Generated, not yet wired in |

## Coverage (as of last run)

- **622 settings** total (612 parsed + 12 machine axis expanded from loop, 2 duplicates removed)
- **474/622 settings** mapped to Tab.cpp pages; remaining grouped by PrintConfig category
- **22 .proto files** grouped by Tab.cpp page (e.g. `print_quality.proto`, `filament.proto`)
- **451 option lines** across 16 pages in `TabLayout_generated.cpp`
- **610/610 shared settings** validated against original -- **0 errors**
- **12 extra settings** from machine axis expansion (original uses a runtime `AxisDefault` loop)

## Known Limitations

- `filament_type` enum values are populated at runtime via `MaterialType::all()` -- cannot be statically parsed
- `min_bead_width` shows a false-positive enum_map mismatch in validation
- Machine axis settings (`machine_max_speed_x/y/z/e`, etc.) are manually expanded from the original `AxisDefault` loop

## How to Run

```bash
# Full pipeline (compile + generate + validate)
python tools/run_codegen.py

# Validate only
python tools/run_codegen.py --validate-only

# CMake targets (after including ConfigCodegen.cmake)
cmake --build . --target codegen_config
cmake --build . --target validate_config
```

## File Layout

```
src/proto/
+-- config_metadata.proto              # Custom field option extensions
+-- generated/
    +-- print_quality.proto            # 99 settings  (Print / Quality page)
    +-- strength.proto                 # 46 settings  (Print / Strength page)
    +-- print_speed.proto              # 48 settings  (Print / Speed page)
    +-- print_support.proto            # 48 settings  (Print / Support page)
    +-- others.proto                   # 48 settings  (Print / Others page)
    +-- print_multimaterial.proto      # 41 settings  (Print / Multimaterial page)
    +-- filament.proto                 # 43 settings  (Filament / Filament page)
    +-- cooling.proto                  # 22 settings  (Filament / Cooling page)
    +-- filament_multimaterial.proto   # 21 settings  (Filament / Multimaterial page)
    +-- advanced.proto                 # 3 settings   (Filament / Advanced page)
    +-- dependencies.proto             # 2 settings   (Filament / Dependencies page)
    +-- filament_notes.proto           # 1 setting    (Filament / Notes page)
    +-- basic_information.proto        # 29 settings  (Printer / Basic information page)
    +-- machine_g_code.proto           # 12 settings  (Printer / Machine G-code page)
    +-- motion_ability.proto           # 10 settings  (Printer / Motion ability page)
    +-- printer_notes.proto            # 1 setting    (Printer / Notes page)
    +-- machine_limits.proto           # 19 settings  (no Tab.cpp page, Machine limits category)
    +-- uncategorized.proto            # 116 settings (no Tab.cpp page, no category)
    +-- quality.proto                  # 6 settings   (fallback)
    +-- support.proto                  # 5 settings   (fallback)
    +-- speed.proto                    # 1 setting    (fallback)
    +-- extruders.proto                # 1 setting    (fallback)
    +-- parsed_settings.json           # JSON IR for inspection

tools/
+-- config_codegen.py                  # Proto descriptor -> C++ codegen
+-- config_metadata_pb2.py             # Generated Python bindings for extensions
+-- validate_codegen.py                # Generated vs original validation
+-- run_codegen.py                     # Pipeline script

codegen/generated/
+-- PrintConfigDef_generated.cpp       # init_fff_params() body
+-- Preset_options_generated.cpp       # s_Preset_*_options
+-- Invalidation_generated.cpp         # Invalidation map
+-- OptionKeys_generated.cpp           # Extruder/filament key lists
+-- TabLayout_generated.cpp            # Tab UI layout (TabPrint/Filament/Printer_build_layout)

cmake/modules/
+-- ConfigCodegen.cmake                # CMake integration
```
