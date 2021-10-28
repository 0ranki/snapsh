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
- The output can be limited to a specific type with the `-t` or `--type` flags, see below
##### `--mount` and `--umount`
- If you wish to do something manually that requires accessing the toplevel subvolume, these are convenience options for quickly mounting/unmounting it to the location specified by TOPLEVEL variable in the config file (default `/root/btrfs-toplevel`)
##### `-s SUBVOLUME` or `--snapshot SUBVOLUME`
- Take a readonly snapshot of subvolume named SUBVOLUME.
  - example: `sudo snapsh -s root` will take a snapshot of subvolume named `root`
- The snapshots are stored in a separate subvolume (named according to the SNAPSHOTS_LOCATION config variable, default `snapshots`) under the toplevel id=5 subvolume.
##### `-d STR` or `--description STR`
- Add a description when taking a snapshot.
  - example: `sudo snapsh -s root -d "Installing XFCE"` will take snapshot of subvolume named `root` and add a description "Installing XFCE"
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
##### `-r N` or `--remove N`
- Delete snapshot number N (use `-l, --list` to get the correct number)
- <b>Please be careful with snapshot numbers, as they will change after every deletion</b><br>
  If you are deleting multiple snapshots, start from the newest one to delete, that way the indexes of older ones will stay the same.
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
- Batch deletion of snapshots
- `systemd` timer for automating snapshots
- `--auto` option (quiet) for automated snapshots
- Option to snapshot multiple/all subvolumes

Root access is required for `btrfs-progs`.

### License:
GPL v3.0
