# vtspice

vtspice is a bash script that runs SPICE simulations on a remote server. If the server is not powerful enough, it is difficult to use EDA while the simulation is running. This script allows you to separate the local server for EDA and the remote server for running SPICE simulations. Furthermore, the simulation will not be interrupted if the network connection is lost (this is important!).

## Dependency

-   Remote And Local Server: `ssh`, `scp` (Make sure SSH connection is available from local to remote)
-   Remote Server Only: `screen`, `hspice` (You can use other SPICE like `ngspice`. See "Change SPICE simulator" section at the bottom of the README.md)

## Before Using

-   This script transfers local files to a remote server via scp command and runs the simulation on the remote server with screen command. If a passphrase is set for the SSH private key, you will need to enter the passphrase 4 times. If this is inconvenient, it is recommended to create a special user on remote server who can only execute SPICE commands and use public key authentication without a passphrase.
-   Even if the terminal is closed during simulation, the simulation continues on the virtual terminal on the remote server. `ssh` to the remote server and run `screen -r`. If you want to interrupt the simulation, type <kbd>Ctrl</kbd>+<kbd>C</kbd>.

## Setup

-   Check the commands in "Dependency" section in README.md is available.
-   Download the vtspice script from Releases or `git clone https://github.com/nxg-nxg/vtspice.git`
-   Create directory for vtspice on remote server (ex. `/home/remoteuser/vtspice`).
-   Edit "VARIABLES" section in vtspice script, and set below items.
    -   `REMOTEUSR`, Username who run simulation on remote server
    -   `REMOTEIP`, IP adress of remote server
    -   `REMOTEVTSDIR`, Path to directory for vtspice on remote server (Cannot end of path with "/")
    -   `LOCALUSR`, Username who run vtspice script on local server
    -   `LOCALIP`, IP adress of local server (127.0.0.1 or localhost cannot be used)
    -   `SPICE`, Path to the original file of the SPICE command (Use `which` command, like `which hspice`)
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

It should be noted that these commands are interpreted as follows.
Therefore, be careful which file you use for the **first argument**; the order after the second is unimportant.

```
$ /path/to/hspice inverter.sp
```

The files specified in the options will then be transferred to the remote server via SCP command.
Any type or number of files is acceptable.

```
$ vtspice inverter.sp netlist stimulus.sp meas_file mos.mdl mos.lib diode.mdl cap.mdl
```

If you can place library file or device model on the remote server, fewer arguments are required.

```
$ vtspice inverter.sp netlist stimulus.sp meas_file
```

In this case, you may need to edit the path of `.include` or `.lib` statement in your SPICE files.

```
# BEFORE
.include "/home/localuser/design/rules/mos.mdl"
```
```
# AFTER
.include "/home/remoteuser/design/rules/mos.mdl"
# OR
.include "~/design/rules/mos.mdl"
```

## Change SPICE simulator

If you want to use other SPICE simulator like `ngspice`, Edit "VARIABLES" section in vtspice script.

```
# BEFORE
## The type of SPICE simulator (Options defined in aliases are stripped)
SPICE="/path/to/hspice"

## Options given to SPICE commands
OPTIONS="-mt 8 -hpp"
```
```
# AFTER
## The type of SPICE simulator (Options defined in aliases are stripped)
SPICE="/path/to/ngspice"

## Options given to SPICE commands
OPTIONS=""
```

## Options of SPICE command

Some machine configurations may have SPICE commands aliased.

```
$ which hspice
hspice:    aliased to /path/to/hspice -mt 4 -hpp -i
```

Therefore, specify the original file in `SPICE` (In the above example, `/path/to/hspice`)
and specify the required command options in `OPTIONS` in the "VARIABLES" section.
