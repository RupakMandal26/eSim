# eSim-2.5 Upgradation & Porting to Ubuntu 25.04 (Plucky Puffin)

**Submitted by:** Rupak Mandal (`rupakcmandal@gmail.com`)  
**Academic Background:** B.Tech in Electronics & Communication Engineering (2025–2029 Cohort), CVM University  
**Task:** FOSSEE IIT Bombay — eSim Semester Long Internship (Autumn 2026) Screening Task 4 (eSim Upgradation)  
**Target Environment:** Ubuntu 25.04 (Plucky Puffin, x86_64, Kernel 6.14, GCC 14.2.0) | KiCad 8.0 | GHDL 4.1.0 | LLVM-18 | Ngspice  
**Forked Repository:** `https://github.com/RupakMandal26/eSim/tree/installers`

---

## Summary of Issues Diagnosed & Resolved (11 Items Covered)

### B-01 / Issue 1: Root Installer Version Regex Failure & Missing Ubuntu 25.04 Target (`install-eSim.sh`)
- **Issue:** `lsb_release -d | grep -oP '\d+\.\d+\.\d+'` expects a three-part version string and returns empty on Ubuntu `25.04`, causing `Unsupported Ubuntu version: 25.04 ()`, while `run_version_script()` lacks a `"25.04")` branch.
- **Fix:** Updated the regex to `grep -oP '\d+\.\d+(\.\d+)?'` in `get_ubuntu_version()` and added a `"25.04")` branch routing to `install-eSim-scripts/install-eSim-25.04.sh`.

### B-02 / Issue 2: Obsolete KiCad 6.0 Launchpad PPA on Ubuntu 25.04 (`install-eSim-scripts/install-eSim-25.04.sh`)
- **Issue:** `ppa:kicad/kicad-6.0-releases` lacks a `plucky` Release file and fails `apt-get update` with HTTP 404.
- **Fix:** Bypassed the obsolete PPA on Ubuntu 25.04 and installed Ubuntu 25.04's native KiCad 8.0 packages (`kicad`, `kicad-footprints`, `kicad-libraries`, `kicad-symbols`, `kicad-templates`).

### B-03 / Issue 3: Nested NGHDL Dispatcher Version Detection Failure (`nghdl/install-nghdl.sh`)
- **Issue:** After `nghdl.zip` extracts, `nghdl/install-nghdl.sh` fails with the same three-part version regex and missing `"25.04")` case branch.
- **Fix:** Updated `FULL_VERSION` to `grep -oP '\d+\.\d+(\.\d+)?'` and added a `"25.04")` branch dispatching to `nghdl/install-nghdl-scripts/install-nghdl-25.04.sh`.

### B-04 / Issue 4: Removed GTK2 Package `libcanberra-gtk-module` (`nghdl/install-nghdl-scripts/install-nghdl-25.04.sh`)
- **Issue:** Ubuntu 25.04 removed the legacy GTK2 `libcanberra-gtk-module` package (`Package 'libcanberra-gtk-module' has no installation candidate`).
- **Fix:** Replaced `libcanberra-gtk-module` with `libcanberra-gtk3-module`.

### B-05 / Issue 5: GHDL 4.1.0 Compiler Failure with Default LLVM 20.1.2 (`nghdl/install-nghdl-scripts/install-nghdl-25.04.sh`)
- **Issue:** Default `llvm`/`clang` on Ubuntu 25.04 installs LLVM 20.1.2, causing GHDL 4.1.0 `./configure` to fail with `Unhandled version llvm 20.1.2`.
- **Fix:** Removed unversioned LLVM 20 (`llvm llvm-dev`), installed `llvm-18`, `llvm-18-dev`, and `clang-18`, and configured GHDL with `--with-llvm-config=/usr/bin/llvm-config-18 CC=clang-18 CXX=clang++-18`.

### B-06 / Issue 6: Parallel Build Out-of-Memory (OOM) Crashes (`nghdl/install-nghdl-scripts/install-nghdl-25.04.sh`)
- **Issue:** Unbounded `make -j$(nproc)` triggers OOM (`SIGKILL`) kills during GHDL 4.1.0, Verilator, and Ngspice builds on 4 GB VMs.
- **Fix:** Provisioned a 4 GiB swap file (`/swapfile2`) and bounded parallel compilation to `make -j2` across build stages (lines 107, 136, 181).

### B-07 / Issue 7: KiCad 8.0 Symbol Library Table Migration (`install-eSim-scripts/install-eSim-25.04.sh`)
- **Issue:** `copyKicadLibrary()` copies `sym-lib-table` to `~/.config/kicad/6.0` instead of `~/.config/kicad/8.0`.
- **Fix:** Migrated `sym-lib-table` to `~/.config/kicad/8.0/sym-lib-table` and updated `${KICAD6_SYMBOL_DIR}` to `${KICAD8_SYMBOL_DIR}` across symbol library entries.

### B-08 / Issue 8: Legacy Example Netlist Model Path Resolution (`Examples/BJT_CE_config`)
- **Issue:** Legacy example netlist failed with `Error on line 8: ... esim_npn could not find a valid modelname`.
- **Fix:** Linked `library/deviceModelLibrary/Templates/NPN.lib` via KiCad-to-Ngspice (`KiNg`) and regenerated `BJT_CE_config.cir.out`.

### B-09 / Issue 9: SPICE Model Library UTF-8 / ASCII Enforcement
- **Issue:** Non-UTF-8 characters in legacy `.lib`/`.sub` files cause Ngspice syntax errors.
- **Fix:** Sanitized non-ASCII bytes across SPICE `.lib`/`.sub` files.

### B-10 / Issue 10: Ngspice Binary `.raw` Parsing Crash (`src/ngspiceSimulation/NgspiceWidget.py`)
- **Issue:** Line 138 runs `ngspice -p "{raw_file}"`, causing Ngspice to parse binary `.raw` output as SPICE syntax (`Error: UTF-8 syntax error in line 19`).
- **Fix:** Patched line 138 to launch `ngspice -i` so both interactive Ngspice console usage and PythonPlotting (`v_u_ic1#branch`) work cleanly.

### B-11 / Issue 11: End-to-End Mixed-Signal VHDL Compilation & KiCad 8.0 Symbol Generation (NGHDL)
- **Verification:** Compiled `and_gate.vhdl` via NGHDL + GHDL 4.1.0/LLVM-18 into `ghdl.cm` and verified `(symbol "and_gate" ...)` inside `/usr/share/kicad/symbols/eSim_Nghdl.kicad_sym`.

---

## Repository & Patch Artifacts
- `Ubuntu/install-eSim.sh` & `Ubuntu/install-eSim-scripts/install-eSim-25.04.sh`
- `ubuntu-25.04-upgrade/patched-files/install-nghdl.sh`
- `ubuntu-25.04-upgrade/patched-files/install-nghdl-25.04.sh`
- `ubuntu-25.04-upgrade/patched-files/NgspiceWidget.py`
- `ubuntu-25.04-upgrade/*.diff` (`esim_25_04_patch.diff`, `nghdl_25_04_patch.diff`, `ngspice_widget_25_04_patch.diff`)
