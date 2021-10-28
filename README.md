# snapsh
Btrfs snapshot managing bash script

### Requirements:
- `bash`
- GNU `setopt` - part of GNU `coreutils`
- `btrfs-progs`- Userspace programs for btrfs

### Why not `snapper`?
- This is a hobby project, but for my personal usage has been very useful. Hopefully it will be for someone else too.

### Installation:
- To start using snapsh, clone or download the repo and run `./snapsh --install`. This will copy the included config file to /etc/snapsh.conf, install the included systemd service (but not enable it), and copy the script itself to `/usr/local/bin/snapsh`. See the section about rollbacks below for details about the systemd service.

### Usage:
#### General:
- Script needs the toplevel subvolume (id=5) mounted somewhere. By default snapsh uses `/root/btrfs-toplevel`, but you can mount it anywhere you like and define it with `TOPLEVEL` variable in the config file. Toplevel subvolume is unmounted automatically after operation and the mountpoint removed to avoid problems with recursion on daily use. Please note that the toplevel subvolume is not the same thing as the subvolume mounted to `/`
- Will create a subvolume named `snapshots` by default to the toplevel. This subvolume for storing the snapshots can be changed with `SNAPSHOTS_LOCATION` in the config file.

##### `-h` or `--help`
- show usage instructions
##### `-v` or `--version`
- display the current version of snapsh
##### `--install`
- see "Installation" above
##### `-l` or `--list`
- List snapshots created by snapsh. The snapshots are always numbered sequentially from 1, starting from the oldest, so when deleting snapshots make sure you delete the newest ones first.
- The output can be limited to a specific type with the `-t` or `--type` flags, see the section about `-t` below
```
[user@localhost]$ sudo snapsh -l
Number | Time:                          |     Source |   Type | Description
     1 |   pe 29.10.2021 01.35.59 +0300 |       home |   auto | typetest 1
     2 |   pe 29.10.2021 01.36.09 +0300 |       home | manual | typetest1
     3 |   pe 29.10.2021 01.36.19 +0300 |       home |   auto | typetest 2
     4 |   pe 29.10.2021 01.36.29 +0300 |       home | manual | typetest2
     5 |   pe 29.10.2021 01.36.40 +0300 |       home |   auto | typetest 3
     6 |   pe 29.10.2021 01.36.50 +0300 |       home | manual | typetest3
     7 |   pe 29.10.2021 01.37.00 +0300 |       home |   auto | typetest 4
     8 |   pe 29.10.2021 01.37.10 +0300 |       home | manual | typetest4
     9 |   pe 29.10.2021 01.37.20 +0300 |       home |   auto | typetest 5
    10 |   pe 29.10.2021 01.37.30 +0300 |       home | manual | typetest5
    11 |   pe 29.10.2021 01.37.40 +0300 |       home |   auto | typetest 6
    12 |   pe 29.10.2021 01.37.51 +0300 |       home | manual | typetest6
    13 |   pe 29.10.2021 01.38.01 +0300 |       home |   auto | typetest 7
    14 |   pe 29.10.2021 01.38.11 +0300 |       home | manual | typetest7
    15 |   pe 29.10.2021 01.38.21 +0300 |       home |   auto | typetest 8
    16 |   pe 29.10.2021 01.38.31 +0300 |       home | manual | typetest8
    17 |   pe 29.10.2021 01.38.41 +0300 |       home |   auto | typetest 9
    18 |   pe 29.10.2021 01.38.51 +0300 |       home | manual | typetest9
    19 |   pe 29.10.2021 01.39.01 +0300 |       home |   auto | typetest 10
    20 |   pe 29.10.2021 01.39.12 +0300 |       home | manual | typetest10
[user@localhost]$
```
##### `--mount` and `--umount`
- If you wish to do something manually that requires accessing the toplevel subvolume, these are convenience options for quickly mounting/unmounting it to the location specified by TOPLEVEL variable in the config file (default `/root/btrfs-toplevel`)
##### `-s SUBVOLUME` or `--snapshot SUBVOLUME`
- Take a readonly snapshot of subvolume named SUBVOLUME.
  - example: `sudo snapsh -s root` will take a snapshot of subvolume named `root`
- The snapshots are stored in a separate subvolume (named according to the SNAPSHOTS_LOCATION config variable, default `snapshots`) under the toplevel id=5 subvolume.
```
[user@localhost]$ sudo snapsh -s home
Creating snapshot of subvolume home as     home_snapshot_2021.10.29-01.43.24
Create a readonly snapshot of '/root/btrfs-toplevel/home' in '/root/btrfs-toplevel/snapshots/home_snapshot_2021.10.29-01.43.24'
Snapshot created!
    home_snapshot_2021.10.29-01.43.24
    DATE=pe 29.10.2021 01.43.25 +0300
    SOURCE_SUBVOLUME=home
    DESCRIPTION=
    TYPE="manual"
[user@localhost]$
```
##### `-d STR` or `--description STR`
- Add a description when taking a snapshot.
  - example: `sudo snapsh -s root -d "Installing XFCE"` will take snapshot of subvolume named `root` and add a description "Installing XFCE"
```
[user@localhost]$ sudo snapsh -s home -d "This is a description for the snapshot"
Creating snapshot of subvolume home as     home_snapshot_2021.10.29-01.44.20
Create a readonly snapshot of '/root/btrfs-toplevel/home' in '/root/btrfs-toplevel/snapshots/home_snapshot_2021.10.29-01.44.20'
Snapshot created!
    home_snapshot_2021.10.29-01.44.20
    DATE=pe 29.10.2021 01.44.20 +0300
    SOURCE_SUBVOLUME=home
    DESCRIPTION=This is a description for the snapshot
    TYPE="manual"
[user@localhost]$
```
##### `-t TYPE` or `--type TYPE`
- `TYPE` can be one of `manual, auto, boot, backup`. Default is `manual`.
  - `auto` is intended to be used with automatic scheduled backups e.g. via `cron`
  - `backup` is used when generating a backup snapshot before rollbacks
  - `boot` is intended to be used with automatic backups taken on every boot
  - All options can be set manually
- If used when taking snapshot, mark that snapshot as the given type
  - example: `sudo snapsh -s root -d "Installing XFCE" -t auto` will take a snapshot of subvolume named `root`, add a description "Installing XFCE" and mark that snapshot as type `auto`
- If used when listing snapshots, list only snapshots of that specific type
  - example: `sudo snapsh -l -t backup` will list only the snapshots taken before rollbacks (if you have not marked others with that type)
```
[user@localhost]$ sudo snapsh -s home -d "This is a description for the snapshot" -t auto
Creating snapshot of subvolume home as     home_snapshot_2021.10.29-01.44.57
Create a readonly snapshot of '/root/btrfs-toplevel/home' in '/root/btrfs-toplevel/snapshots/home_snapshot_2021.10.29-01.44.57'
Snapshot created!
    home_snapshot_2021.10.29-01.44.57
    DATE=pe 29.10.2021 01.44.57 +0300
    SOURCE_SUBVOLUME=home
    DESCRIPTION=This is a description for the snapshot
    TYPE="auto"
[user@localhost]$
```
##### `-r N` or `--remove N`
- If `N` is a single number
  - Delete snapshot number N (use `-l, --list` to get the correct number)
- If `N` is a comma-separated list of numbers `N1,N2,N3`
  - Delete snapshots with numbers `N1 N2 N3`
- If `N` is a range separated with a hyphen `N1-N2`
  - Delete snapshots starting from number `N1` to number `N2` (e.g. 12-16 will delete snapshots 12,13,14,15 and 16)
- <b>Please be careful with snapshot numbers, as they will change after every deletion</b><br>
  If you are deleting multiple snapshots one by one, start from the newest one to delete, that way the indexes of older ones will stay the same.
##### `--rollback NUMBER`
- Rollback to the snapshot number NUMBER
  - The correct subvolume is detected automatically
- The process is the following:
  - Take a snapshot of type `backup` of the active target subvolume
  - Rename the currently used subvolume to `SUBVOLUME.previous` (e.g. home.previous)
  - Take a read/write snapshot of the target snapshot, name that as `SUBVOLUME` (e.g. home)
  - If the system is using SELinux, ask to enable automatic relabeling for the next boot. See below about SELinux.
  - Ask to reboot the system immediately
  - After the reboot, the included systemd service deletes the `SUBVOLUME.previous` subvolume.


#### Notes:

##### SELinux:
- After a rollback, SELinux might cause issues like not being able to log in, because SELinux prevents access to the home folder (Might happen for the root user as well). Snapsh asks to enable a relabeling on the next boot if it detects that SELinux is enforcing. This is recommended, but the relabeling might take time on a large filesystem. If you want to skip the long relabeling and face issues, SELinux can be set to permissive mode before booting by:
  - Adding the parameter `enforcing=0` to the kernel command line (e.g. press `e` on GRUB menu to edit the command line for the current boot)
  - Booting to a live USB, mounting the root subvolume and changing `SELINUX=enforcing` to `SELINUX=permissive` in `/etc/selinux/config`


### Planned features:
- `systemd` timer for automating snapshots
- `--auto` option (quiet) for automated snapshots
- Option to snapshot multiple/all subvolumes

Root access is required for `btrfs-progs`.

### License:
GPL v3.0
