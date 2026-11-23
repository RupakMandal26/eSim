# eSim-2.5 Upgradation & Porting to Ubuntu 25.04 (Plucky Puffin)

**Submitted by:** Rupak Mandal  
**Task:** FOSSEE IIT Bombay — eSim Semester Long Internship (Autumn 2026) Screening Task 4  
**Target Environment:** Ubuntu 25.04 (Plucky Puffin, x86_64, GCC 14.2) | KiCad 8.0 | GHDL 4.1.0 | LLVM-18 | Ngspice

---

## Summary of Issues Diagnosed & Resolved (11 Bugs Fixed)

### 1. Root & NGHDL Installer Version Regex Failure (`install-eSim.sh` & `nghdl/install-nghdl.sh`)
- **Issue:** `lsb_release -d | grep -oP '\d+\.\d+\.\d+'` expects a three-part version string and returns empty on Ubuntu `25.04`, causing `Unsupported Ubuntu version: 25.04 ()`.
- **Fix:** Updated the regex to `grep -oP '\d+\.\d+(\.\d+)?'` in both dispatchers and added `"25.04")` case branches routing to `install-eSim-25.04.sh` and `install-nghdl-25.04.sh`.

### 2. Obsolete KiCad 6.0 Launchpad PPA (`install-eSim-scripts/install-eSim-25.04.sh`)
- **Issue:** `ppa:kicad/kicad-6.0-releases` lacks a `plucky` Release file and fails `apt-get update` with HTTP 404.
- **Fix:** Bypassed the obsolete PPA on Ubuntu 25.04 and installed Ubuntu 25.04's native KiCad 8.0 packages (`kicad`, `kicad-footprints`, `kicad-libraries`, `kicad-symbols`, `kicad-templates`).

### 3. Removed `libcanberra-gtk-module` Package (`nghdl/install-nghdl-scripts/install-nghdl-25.04.sh`)
- **Issue:** Ubuntu 25.04 dropped the legacy GTK2 `libcanberra-gtk-module` package.
- **Fix:** Replaced `libcanberra-gtk-module` with `libcanberra-gtk3-module`.

### 4. GHDL 4.1.0 Compiler Failure with LLVM 20.1.2 (`install-nghdl-scripts/install-nghdl-25.04.sh`)
- **Issue:** Default `llvm`/`clang` on Ubuntu 25.04 installs LLVM 20.1.2, causing GHDL 4.1.0 `./configure` to abort with `Unhandled version llvm 20.1.2`.
- **Fix:** Removed unversioned LLVM 20, installed `llvm-18`, `llvm-18-dev`, and `clang-18`, and configured GHDL with `--with-llvm-config=/usr/bin/llvm-config-18 CC=clang-18 CXX=clang++-18`.

### 5. Parallel Build Out-of-Memory (OOM) Crashes (`install-nghdl-scripts/install-nghdl-25.04.sh`)
- **Issue:** Unbounded `make -j$(nproc)` triggers OOM (`SIGKILL`) kills during GHDL 4.1.0, Verilator, and Ngspice compilation on 4 GB VMs.
- **Fix:** Provisioned swap space and bounded builds to `make -j2`.

### 6. KiCad 8.0 Symbol Library Table Migration (`install-eSim-scripts/install-eSim-25.04.sh`)
- **Issue:** `copyKicadLibrary()` copies `sym-lib-table` to `~/.config/kicad/6.0` instead of `~/.config/kicad/8.0`.
- **Fix:** Migrated `sym-lib-table` to `~/.config/kicad/8.0/sym-lib-table` and updated `${KICAD6_SYMBOL_DIR}` to `${KICAD8_SYMBOL_DIR}` across all 15 eSim symbol libraries.

### 7. Legacy Netlist Model Resolution & SPICE UTF-8 Enforcement
- **Issue:** Legacy `BJT_CE_config` netlists required KiCad-to-Ngspice regeneration (`Templates/NPN.lib`), and legacy SPICE `.lib` files contained non-UTF-8 characters rejected by modern Ngspice.
- **Fix:** Regenerated netlists via `KiNg` and sanitized non-ASCII bytes across `.lib`/`.sub` files.

### 8. Ngspice Interactive Console Fix (`src/ngspiceSimulation/NgspiceWidget.py`)
- **Issue:** Line 138 executes `ngspice -p "{raw_file}"`, causing modern Ngspice to parse binary `.raw` waveform output as UTF-8 SPICE syntax (`Error: UTF-8 syntax error in line 19`).
- **Fix:** Patched line 138 to launch `ngspice -i`, allowing both interactive Ngspice console usage and PythonPlotting (`v_u_ic1#branch`) to run cleanly.

### 9. End-to-End Mixed-Signal VHDL Verification (NGHDL)
- **Validation:** Compiled a custom 2-input VHDL AND gate (`and_gate.vhdl`) via NGHDL + GHDL 4.1.0/LLVM-18 into `ghdl.cm` and verified automatic symbol registration inside `/usr/share/kicad/symbols/eSim_Nghdl.kicad_sym`.

---

## Included Artifacts
- `Ubuntu/install-eSim.sh` & `Ubuntu/install-eSim-scripts/install-eSim-25.04.sh`
- `ubuntu-25.04-upgrade/patched-files/install-nghdl.sh`
- `ubuntu-25.04-upgrade/patched-files/install-nghdl-25.04.sh`
- `ubuntu-25.04-upgrade/patched-files/NgspiceWidget.py`
- `ubuntu-25.04-upgrade/*.diff` (Unified diff patches)
