# vtspice

vtspice is a bash script that runs SPICE simulations on a remote server.

## Dependency

-   Remote And Local Server: `ssh`, `scp`
-   Remote Server Only: `screen`, `hspice` (You can use other SPICE like `ngspice`. See "Change SPICE simulator" section at the bottom of the README.md)

## Before Using

-   This script transfers local files to a remote server via scp command and runs the simulation on the remote server with screen command. Depending on your machine configuration, you may need to enter a password for the ssh connection. If this is inconvenient, it is recommended to create a special user on remote server who can only execute SPICE commands and use public key authentication without a password.
-   Even if the terminal is closed during simulation, the simulation continues on the virtual terminal on the remote server. `ssh` to the remote server and run `screen -r`. If you want to interrupt the simulation, type <kbd>Ctrl</kbd>+<kbd>C</kbd>.

## Setup

-   Check the commands in "Dependency" section in README.md is available.
-   Create run-directory on remote server (ex. `/home/remoteuser/spice_rundir`).
-   Edit "VARIABLES" section in vtspice script, and set below items.
    -   `REMOTEUSR`, Username who run simulation on remote server
    -   `REMOTEIP`, IP adress of remote server
    -   `REMOTERUNDIR`, Path of simulation run-directory on remote server (Cannot end of path with "/")
    -   `LOCALUSR`, Username who run vtspice script on local server
    -   `LOCALIP`, IP adress of local server (127.0.0.1 or localhost cannot be used)
-   Place the script in some directory on local server (ex. `/home/localuser/.vtspice/vtspice`), and set environment variable PATH.

## How To Run

Place SPICE files to local directory (ex. inverter.sp, inverter.inp, netlist, mos.mdl, mos.skw).

**inverter.sp**

```
.title inverter
.include "./inverter.inp"
.include "./netlist"
.include "./mos.mdl"
.lib "./mos.skw" NT
.lib "./mos.skw" PT
.end
```

Then, open a terminal in the directory, and run the following command.

```
$ vtspice inverter.sp inverter.inp netlist mos.mdl mos.skw
```

This command is interpreted as follows. Therefore, be careful which file you use for the **first argument**; the order after the second is unimportant.

```
$ hspice inverter.sp
```

If you can place library file or device model on the remote server, fewer arguments are required.

```
$ vtspice inverter.sp inverter.inp netlist
```

In this case, you may need to edit the path of `.include` or `.lib` statement in SPICE files.

```
# BEFORE
.include "/home/localuser/design/rules/mos.mdl"

# AFTER
.include "/home/remoteuser/design/rules/mos.mdl"
.include "~/design/rules/mos.mdl"
```

## Change SPICE simulator

If you want to use other SPICE simulator like `ngspice`, Edit the following part of the script (around line 100).

```
# YOU CAN CHANGE THIS LINE TO OTHER SPICE LIKE NGSPICE (CHANGE OPTION TOO)
hspice ${HEADFILE} -mt ${MT} -hpp | tee ${CELLNAME}.spice.log
```

**The case of ngspice**

```
ngspice ${HEADFILE} | tee ${CELLNAME}.spice.log
```
