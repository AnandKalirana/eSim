# eSim 2.5 on Ubuntu 25.04 / 26.04: issues and fixes

Tested on Ubuntu 26.04.1 LTS (VirtualBox VM, GCC 15, Python 3.14, LLVM 17-22 available). Full write-up with logs and diffs is in the submitted report.

## Fixed and tested (in this branch)

| # | Problem | Cause | Fix |
|---|---------|-------|-----|
| 1 | install-eSim.sh exits with "Unsupported Ubuntu version" | Version dispatcher only lists up to 24.04 | `install-eSim.sh`: `"24.04")` became `"24.04" \| "25.04" \| "26.04")` |
| 2 | KiCad install fails with a 404 from the KiCad PPA | The script picks the KiCad 6.0 PPA for every version except 24.04, and that PPA has no release for newer Ubuntu | `install-eSim-24.04.sh`, `installKicad`: on 25.04/26.04 install KiCad (9.0.8) from the Ubuntu repositories and return |
| 3 | NGHDL step stops with "Unsupported Ubuntu version" | `nghdl/install-nghdl.sh` has the same version list | `install-eSim-24.04.sh`: `sed` adds 25.04 and 26.04 before running `./install-nghdl.sh --install` |
| 4 | NGHDL package install fails: `libcanberra-gtk-module` has no installation candidate | The package was removed from Ubuntu 26.04 | `install-eSim-24.04.sh`: `sed` removes it from `install-nghdl-24.04.sh` |

## Found, documented, proposed (not yet in this branch)

| # | Problem | Cause | Proposed fix |
|---|---------|-------|--------------|
| 5 | GHDL 4.1.0 configure stops with "Unhandled version llvm 21.1.8" | GHDL 4.1.0 does not recognise LLVM 21 | Use LLVM 18: replace `llvm llvm-dev` with `llvm-18 llvm-18-dev` and `--with-llvm-config=/usr/bin/llvm-config-18` in `install-nghdl-24.04.sh` (confirmed: GHDL then builds) |
| 6 | NGHDL simulator (Ngspice) build fails: "bool cannot be defined via typedef" | GCC 15 defaults to C23, where `bool` is a keyword | Add `export CFLAGS=-std=gnu17` before the Ngspice `../configure` call |
| 7 | Installer reports success although NGHDL failed | The NGHDL exit status is never checked (`trap "" ERR` around the call) | Check `$?` after `./install-nghdl.sh --install` and abort with a message |
| 8 | NGHDL build stops on "replace ... overriding mode 0644?" | Build files left by an earlier run owned by root | `sudo chown -R` the nghdl folders before re-running, and do not run the installer with sudo |

## Not done
The complete eSim install was not finished (Ngspice, Makerchip, SKY130 steps were not reached). The KiCad library copy still targets the 6.0 config folder and probably needs updating for KiCad 9.
