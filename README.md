# snapsh
Btrfs snapshot managing bash script

### Requirements:
- `bash`
- GNU `setopt` - part of GNU `coreutils`
- `btrfs-progs`- Userspace programs for btrfs

### Instructions:
- Script needs the toplevel subvolume (id=5) mounted somewhere. Default location is `/root/btrfs-toplevel`, but you can mount it anywhere you like and define it with `TOPLEVEL` variable. (A separate config file will be implemented later).
- Will create a subvolume named snapshots by default to the toplevel. This can also be changed with `SNAPSHOTS_LOCATION`.
- Display usage instructions with `snapsh -h` or `snapsh --help`
- Taking snapshots requires root priviledges. Take a snapshot with `snapsh -s SUBVOLUME` or `snapsh --snapshot SUBVOLUME`, where `SUBVOLUME` is the name of the source subvolume. You can add a description for the snapshot with the `-d | --description` option<br><br>Example with Fedora default btrfs layout with `root` and `home` subvolumes: <br> `snapsh -s root -d "This is a snapshot" ` <br> This will create a snapshot called `root_snapshot_YYYY.MM.DD-hh:mm:ss` to the `snapshots` subvolume (or the one you defined with `SNAPSHOTS_LOCATION`), with a description "This is a snapshot"
- Snapshots can be listed with `snapsh -l` or `snapsh --list`
- Delete snapshots with the `-r` or `--remove` option. List snapshots first with `snapsh -l`, then delete snapshot with e.g. `snapsh -r 2`, where 2 is the number of the deletable snapshot in the `-l` listing. The list numbers always start from 1 and increment from there, so always check the number before deletion. Batch deletion might be implemented later.
- Rollback to a snapshot with `snapsh --rollback NUMBER`, where NUMBER is the number listed in `snapsh -l`. The subvolume will be detected from the snapshot. E.g. `snapsh --rollback 14`. Script will ask to reboot system once done.
- Snapshot type displayed in listing can be set with the `-t, --type` option. When used in conjunction with `-l | --list`, it will filter the results based on the type. E.g. `snapsh -l -t manual` will list all the snapshots with the type `manual`. Type can be one of `manual, auto, boot` or `backup`. Note that only `manual` and `backup` are implemented properly at this stage, but you can still use the other types as well.

### Planned features:
- separate config file
- `--install` option to integrate script into system
- Batch deletion of snapshots
- `systemd` timer for automating snapshots
- `--auto` option (quiet) for automated snapshots
- Option to snapshot multiple/all subvolumes

Root access is required for `btrfs-progs`.

### License:
GPL v3.0
