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
- Taking snapshots requires root priviledges. Take a snapshot with `snapsh -s SUBVOLUME` or `snapsh --snapshot SUBVOLUME`, where `SUBVOLUME` is the name of the source subvolume. You can add a description for the snapshot with the `-d | --description` option (must be used before the `-s` option)<br><br>Example with Fedora default btrfs layout with `root` and `home` subvolumes: <br> `snapsh -d "This is a snapshot" -s root` <br> This will create a snapshot called `root_snapshot_YYYY.MM.DD-hh:mm:ss` to the `snapshots` subvolume (or the one you defined with `SNAPSHOTS_LOCATION`), with a description "This is a snapshot"
