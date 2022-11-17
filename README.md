# vtspice

vtspice is a bash script that runs SPICE simulations on a remote server. If the server is not powerful enough, it is difficult to use EDA while the simulation is running. This script allows you to separate the local server for EDA and the remote server for running SPICE simulations. Furthermore, the simulation will not be interrupted if the network connection is lost (this is important!).

## Dependency

-   Remote And Local Server: `ssh`, `scp` (Make sure SSH connection is available from local to remote)
-   Remote Server Only: `screen`, `hspice` (You can use other SPICE like `ngspice`. See "Change SPICE simulator" section at the bottom of the README.md)

## Before Using

-   This script transfers local files to a remote server via scp command and runs the simulation on the remote server with screen command. If a passphrase is set for the SSH private key, you will need to enter the passphrase 4 times. If this is inconvenient, it is recommended to create a special user on remote server who can only execute SPICE commands and use public key authentication without a password.
-   Even if the terminal is closed during simulation, the simulation continues on the virtual terminal on the remote server. `ssh` to the remote server and run `screen -r`. If you want to interrupt the simulation, type <kbd>Ctrl</kbd>+<kbd>C</kbd>.

## Setup

-   Check the commands in "Dependency" section in README.md is available.
-   Create directory fot vtspice on remote server (ex. `/home/remoteuser/vtspice`).
-   Edit "VARIABLES" section in vtspice script, and set below items.
    -   `REMOTEUSR`, Username who run simulation on remote server
    -   `REMOTEIP`, IP adress of remote server
    -   `REMOTEVTSDIR`, Path of directory fot vtspice on remote server (Cannot end of path with "/")
    -   `LOCALUSR`, Username who run vtspice script on local server
    -   `LOCALIP`, IP adress of local server (127.0.0.1 or localhost cannot be used)
    -   `SPICE`, The type of SPICE simulator, like hspice or ngspice (Options defined in aliases are stripped)
    -   `OPTIONS`, Options given to SPICE commands
-   Place the script in some directory on local server (ex. `/home/localuser/.local/vtspice`), and set environment variable PATH.

## How To Run

Place SPICE files (ex. inverter.sp, netlist, mos.mdl) to local directory.

**inverter.sp**

```
.title inverter
.include "./netlist"
.include "./mos.mdl"
.end
```

Then, open a terminal in the directory, and run the following command.

```
$ vtspice inverter.sp netlist mos.mdl
```

The files specified in the options will then be transferred to the remote server via SCP command.
Any type or number of files is acceptable.

```
$ vtspice inverter.sp stimulus.sp meas.sp netlist.spf  mos.mdl mos.skw diode.mdl
```

It should be noted that these commands are interpreted as follows.
Therefore, be careful which file you use for the **first argument**; the order after the second is unimportant.

```
$ hspice inverter.sp
```

If you can place library file or device model on the remote server, fewer arguments are required.

```
$ vtspice inverter.sp netlist
```

In this case, you may need to edit the path of `.include` or `.lib` statement in SPICE files.

```
# BEFORE
.include "/home/localuser/design/rules/mos.mdl"
```
```
# AFTER
.include "/home/remoteuser/design/rules/mos.mdl"
.include "~/design/rules/mos.mdl"
```

## Change SPICE simulator

If you want to use other SPICE simulator like `ngspice`, Edit "VARIABLES" section in vtspice script.

```
# BEFORE
## The type of SPICE simulator (Options defined in aliases are stripped)
SPICE="hspice"

## Options given to SPICE commands
OPTIONS="-mt 8 -hpp"
```
```
# AFTER
## The type of SPICE simulator (Options defined in aliases are stripped)
SPICE="ngspice"

## Options given to SPICE commands
OPTIONS=""
```

Some machine configurations may have SPICE commands aliased.

```
$ which hspice
hspice:    aliased to /path/to/spice/hspice64 -mt 4 -hpp -i
```

Therefore, vtspice strips the options by default.
Any necessary options should again be entered in the `OPTIONS` of the "VARIABLES" section.
