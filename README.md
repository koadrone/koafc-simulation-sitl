# KOAFC Simulation - SITL flight controller source

This repository holds the source of the simulated flight controller that ships with the KOAFC Simulation (and the KOAFC Plane
Simulation), and nothing else. The programs are:

| Shipped program | Says on start (the short commit is this tag's) | Branch / tag here | Upstream ArduPilot base | KOAFC changes |
|---|---|---|---|---|
| `ardupilot\build\sitl\bin\arducopter.exe` | ArduCopter V4.6.3 (koafc-sitl-4.6.3's commit) | `4.6.3` / `koafc-sitl-4.6.3` | `12a6efea39425b084234551355470341f71f79d4` (branch ArduPilot-4.6, one commit after tag Copter-4.6.3) | 3 files, +18 / -1 |
| `ardupilot\build\sitl\bin\arducopter-4.7.1.exe` | ArduCopter V4.7.1 (koafc-sitl-4.7.1's commit) | `4.7.1` / `koafc-sitl-4.7.1` | `dbe792162d06cab66c3475fd5556bf7a120f119e` (tag Copter-4.7.1) | 3 files, +18 / -1 |
| `ardupilot\build\sitl\bin\arduplane-4.7.1.exe` (KOAFC Plane Simulation) | ArduPlane V4.7.1 (koafc-sitl-4.7.1's commit) | `4.7.1` / `koafc-sitl-4.7.1` (the same tag, built with `./waf plane`) | `dbe792162d06cab66c3475fd5556bf7a120f119e` (tag Copter-4.7.1; one ArduPilot tree for every vehicle) | the same 3 files |

Each branch has these commits, in order:

1. The upstream ArduPilot tree at the base commit. It includes the submodules the SITL build uses, as plain files; the
   commit message lists each submodule's commit. Not included, because the SITL build does not use them: ChibiOS,
   CrashDebug, gbenchmark, gsoap, gtest (and, for 4.6.3, littlefs and the DDS client).
2. KOAFC's changes.
3. This file.

The rest of the KOAFC Simulation (the simulator itself, its launcher, maps and assets) is not ArduPilot and is not here.

## KOAFC's changes

The same three changes in both versions (and so in the plane build) (not limited to SITL in the code, but only the SITL build is shipped):
- `libraries/AP_GPS/AP_GPS.cpp`: 230400 baud is tried first when detecting the GPS.
- `libraries/AP_NavEKF3/AP_NavEKF3_Control.cpp`: the gyro bias counts as learned after 2.5 s.
- `libraries/AP_NavEKF3/AP_NavEKF3_VehicleStatus.cpp`: no 10 s GPS stability wait before the EKF aligns, and the GPS
  quality checks (speed / vertical accuracy, innovations) are passed, so the simulated GPS is never called a glitch.

`git show koafc-sitl-4.6.3` and `git show koafc-sitl-4.7.1` show the exact changes.

## Building (Windows, Cygwin, as the shipped programs were built)

The shipped programs were built on Windows 10 under Cygwin 64-bit with ArduPilot's own waf (2.0.27).

1. Install Cygwin 64-bit and set it up for ArduPilot as ArduPilot's guide says
   (https://ardupilot.org/dev/docs/building-setup-windows-cygwin.html). In short, the packages are gcc-g++, make,
   git, python3, python3-devel, python3-pip, python3-future, python3-lxml, libxml2-devel and libxslt-devel, then
   `pip3 install empy==3.3.4 pexpect dronecan`.
2. In the Cygwin terminal:

   ```sh
   git clone <this repository> koafc-sitl && cd koafc-sitl
   git checkout koafc-sitl-4.6.3        # or koafc-sitl-4.7.1
   python3.9 ./waf configure --board sitl --no-submodule-update
   python3.9 ./waf copter               # or, on koafc-sitl-4.7.1, ./waf plane for arduplane
   ```

3. The program is at `build/sitl/bin/arducopter.exe`. The simulator uses the 4.7.1 build under the name
   `arducopter-4.7.1.exe`. `./waf plane` gives `build/sitl/bin/arduplane.exe`, which the
   KOAFC Plane Simulation ships as `arduplane-4.7.1.exe`.
4. It needs Cygwin's runtime beside it, copied unmodified from Cygwin's `bin` folder: cygwin1.dll,
   cyggcc_s-seh-1.dll, cygstdc++-6.dll, cygiconv-2.dll and cygintl-8.dll.

The submodules are already in the tree as plain files, so the build must not try to update them:
`--no-submodule-update` says so, and neither `git submodule update` nor a network connection is needed. The shipped
programs were built with Cygwin's python3.9 (with empy 3.3.4, pexpect and future installed for it).

## Licence

ArduPilot is free software under the GNU General Public License version 3 or later; see `COPYING.txt`. KOAFC's
changes are under the same licence. Copyright (c) ArduPilot Development Team and contributors; the changes, KOAFC.
