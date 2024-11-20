# How-To for jsc-cordex data exchange server for EURO-CORDEX-CMIP6 simulations

Version: 2024-10-14

Authors/Revisions: k.goergen@fz-juelich.de, claas.teichmann@hereon.de, Jesus.Fernandez@unican.es, lars.buntemeyer@hereon.de

# Purpose of this document

Give an overview of the technical setup of and usage instructions for the EURO-CORDEX-CMIP6 intermediate, temporary shared storage. Shared storage is on a server called jsc-cordex at the [Juelich Supercomputing Centre](https://www.fz-juelich.de/ias/jsc/EN) at Forschungszentrum Juelich. The server is meant to exchange CMORized simulation result subsets for community analysis before the data is finally stored at ESGF data nodes. It may also serve as a place where these analysis may be done to a certain extent.

This document is accompanied (in e-mail and on the server) with "Privacy Policy" and a "Terms of Use" documents (`LICENSE.md` and `PRIVACY_STATEMENT.md`).

> [!IMPORTANT]
> - Pay attention to the "Directory structure" section
> - Pay attention to the backup information
> - This is an intermediate, temporary solution, not meant to store full
CMORized simulation output but selected variables needed for the initial 
community papers.
> - Disk space is limited

# Overview information on jsc-cordex, login and IP address etc.

- Type: VM in an OpenStack cloud coumputing infrastructure 
- OS: Ubuntu Linux
- Machine name: `jsc-cordex.fz-juelich.de`
- IP: `134.94.199.142` 
- Login: via ssh, with your personal account, using ssh key-based authentication

The server, services, and storage are provided by the [Juelich Supercomputing Centre](www.fz-juelich.de/ias/jsc/EN) as part of the [JSC Cloud](https://www.fz-juelich.de/en/ias/jsc/systems/scientific-clouds/jsc-cloud).

# Becoming a user of EURO-CORDEX-CMIP6 exchange server

jsc-cordex is also used by CORDEX FPSCONV and CORDEX FPS LUCAS. Projects are separate form each other via Linux groups and standard Linux filesystem permissions.

Each user needs a separate account. **No account sharing, ever.**

You may have the role of (i) a "user" from an institution with an account, or you may be (ii) a "user" and a point of contact ("POC") of an institution; POCs may even do not have an account but they are usually long-term, well-known CORDEX members.

Per institution there can be multiple users, each user with a seperate account. Per institution there is always one POC only.

## Option 1: You are an existing user of jsc-cordex

Users who have already an account on jsc-cordex (in the `esgf` or the `lucas` group), and who want to have access to the CORDEX-CMIP6 data, just let k.goergen@fz-juelich.de know via e-mail. I will add you to a separate `cdxcmip6` Linux group.

## Option 2: You are a new user of jsc-cordex, a user from your institution shall be added

All new users need to send a public ssh key; your institution's POC has to vouch for you. 

Please send a public ssh key to k.goergen@fz-juelich.de, put the institution POC, person who vouches, in CC, informally agree to the Terms of Use and the Privacy Policy. I'll forward the information to the jsc-cordex server sysadmins for account creation and inform you once your account has been created and login is possible.

(To limit the admin effort by creating as few new users as possible, ideally there is one user per institution (e.g., the point of contact per institution or a user from this institution) who already exists on the jsc-cordex machine. But if more users per institution are needed for different analyses etc., their accounts can for sure be created.)

Example on how to generate a new ssh key pair on the command line:
```
ssh-keygen -a 256 -t ed25519  -C "$(hostname)-$(date +'%d-%m-%Y')" -f id_ed25519_FZJ_`date -I`
```

**Please no putty-generated keys!** 

Naming scheme for user: `<1st letter surname><familyname>`, e.g. `jdoe`

# Accessing the system

From the command line:
```
ssh -X -i <your_secret_ssh_key> <your_username>@jsc-cordex.fz-juelich.de
```

Or `scp` or use any GUI-based client capable of logging in to SSH server from any operating system. You might also want to configure your `.ssh/config` like this:
```
Host jsc-cordex
User <your_username>
Hostname jsc-cordex.fz-juelich.de
IdentityFile ~/.ssh/<your_secret_ssh_key>
```
and login using
```
ssh jsc-cordex
```

# Directory structure, where to put your data, where to work

**There is no data officer, every user is responsible to adhere to the 
standards below to ensure an efficient operation.**

> [!IMPORTANT]
> - No simulation or analysis data under `$HOME` (small quota).
> - No software environment installation (e.g., Python `venv`) under `$HOME`.
> - Clean up after yourself, no excessive amount of "temp" files, please.
> - Best practice for file and directory names apply, no blanks or umlauts etc.
> - DO NOT CLUTTER our common data store.
> - Add READMEs where applicable.
> - If your needs are not covered here: Please ask k.goergen@fz-juelich.de

## root dir

Every CORDEX initiative has a root directory under which the complete directory tree resides:

```
export ROOTDIR_CDXCMIP6="/mnt/CORDEX_CMIP6_tmp"
```

## CORDEX-CMIP6 simulation results

**Selected CMORized CORDEX-CMIP6 similation results are stored under their full ESGF directory structure using DRS elements** as specified in the official [CORDEX-CMIP6 archive specifications](https://doi.org/10.5281/zenodo.10961069) starting with the inevitable `<project_id>/<mip_era>` directories:

```
${ROOTDIR_CDXCMIP6}/sim_data/CORDEX/CMIP6
```

If you provide `EUR-12` simulation results these should reside in the respective directory tree below, starting with your `<institution_id>` (see the archive specifications document from above):

```
${ROOTDIR_CDXCMIP6}/sim_data/CORDEX/CMIP6/DD/EUR-12
```

To synchronize your local data with `jsc-cordex`, you can use `rsync` on your institution's subfolder:

```
rsync -avz <local-path-to-cmor-output>/CORDEX/CMIP6/DD/EUR-12/<institution_id> jsc-cordex:/mnt/CORDEX_CMIP6_tmp/sim_data/CORDEX/CMIP6/DD/EUR-12
```

## Processing and analysis results

Any processing, analysis, visualisation and results thereof shall be done in temporary directories, on a per user basis (has proven to work to some extent with CORDEX FPSCONV on jsc-cordex):

```
$ROOTDIR_CDXCMIP6/user_tmp/${USER}
```

## Additional datasets of general interest

If more data is needed, e.g., validation data, grid specs, etc. it might be stored here, if it is used by more than one person, i.e., there is a common interest in the auxilliary data:

```
$ROOTDIR_CDXCMIP6/aux_data/<dataset_name>
```

## Specific software tools of general interest

If a specific software is needed, e.g., a Python `venv`, which is not part of the system-wide install and if it is of general interest, put it here for all to be used:

```
$ROOTDIR_CDXCMIP6/software/<software_tool_name>
```

# Permissions, co-existing data from different CORDEX initiatives

Shared read and individual write-access, ownership and provenance and separation of the different CORDEX initiatives, which store data on jsc-cordex, is via Linux group membership(s):

- `cdxcmip6` (CORDEX-CMIP6)
- `esgf` (CORDEX FPSCONV) 
- `lucas` (CORDEX FPS LUCAS)

> [!IMPORTANT]
> - Check your `umask` setting, or set to `umask 0027` (results in `-rw-r-----` for files, and in `drwxr-s---` for directories).
> - But make sure others can access the respective shared simulation data directories you create as part of the CORDEX-CMIP6 archive protocol directory trees.
> - Make sure that only you as the owner of the uploaded simulation data have write-permission.
> - Within `sim_data/` (see above) a directory structure has been created which has write-.permisison for the group down to the `<domain_id>` of the ESGF compliant directory structure.

# Naming scheme

See above. 

# Data processing, analysis and visualisation

Although jsc-cordex was set up to be a pure data exchange server for CORDEX FPSCONV, it is a multi-core system with 16 core and 128GB RAM and a few basic commonly used geoscience software tools installed with their recent versions:

- cdo
- nco
- ncview
- Python
- R

Send an e-mail to k.goergen@fz-juelich.de if you are missing a software, I will forward this to the sysadmins. Alternatively install by yourself under `$ROOTDIR_CDXCMIP6/software/<software_tool_name>`.

## Conda

If you want to setup a conda environment, you can run `conda init` from the miniforge installation at `/mnt/CORDEX_CMIP6_tmp/software/miniforge3`, e.g., using
```
export PATH="/mnt/CORDEX_CMIP6_tmp/software/miniforge3/bin/:$PATH"
conda init
```
To avoid cluttering your `HOME`, you should install environments in the `/mnt/CORDEX_CMIP6_tmp/user_tmp/<your user name>` directory. You can setup this in your `.condrac` in your `HOME` like this:
```
pkgs_dirs:
  - /mnt/CORDEX_CMIP6_tmp/user_tmp/$USER/.conda/pks
envs_dirs:
  - /mnt/CORDEX_CMIP6_tmp/user_tmp/$USER/conda_envs
channels:
  - conda-forge
auto_activate_base: false
solver: libmamba
default_threads: 16
```

# Physical data storage, data security and integrity

- Data is on a RAID filesystem.
- jsc-cordex storage is part of the JSC storage infrastructure.
- About 400TB storage is mounted to jsc-cordex.
- Less than 100TB is available for CORDEX-CMIP6 (as of 2024-09).
- Tape backups are done regularly.
- Groups are strongly advised to additionally keep local copies of their data, tools, analysis results at all times, there is no liability whatsoever by JSC.
- It is advised to use checksums to make sure data transfers went OK and to allow for integrity checks in case of a system failure and later-on restaging from tape.

# Formalisms

There are two separate documents with "Terms of Use" (LICENSE) and "Privacy Policy" (PRIVACY-STATEMENT) information that all users need to accept prior to being granted access to the system or being added to the CORDEX-CMIP6 Linux group. These documents are provided via e-mail and are stored in the `${ROOTDIR_CDXCMIP6}/READMEs` directory.

# Acknowledgements

If you make use of jsc-cordex, the JSC kindly asks that you use this acknowledgement:

*"The authors gratefully acknowledge the research data exchange infrastructure 
and services provided by the Jülich Supercomputing Centre, Germany, as part of 
the JSC Cloud."*

# Timeline

jsc-cordex is meant as a temporary, intermediate joint storage and analysis system. The longer-term goal is to storage CORDEX simulation results on ESGF data nodes and store analysis results and software along the respective publications.

# Point of contact for jsc-cordex

Klaus GOERGEN, k.goergen@fz-juelich.de
