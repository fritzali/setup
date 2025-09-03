## Maintenance

### Administration

#### Users & Groups

On a new installation, only the superuser account is available. To secure the system, one should never stay logged in as root for prolonged periods. Instead, unprivileged
users should be created and used for most tasks. Group membership and ownership then serves to grant or deny a user or service access to system resources. When using Arch
Linux, information for users and groups is stored in the following files, which should never be manually edited without the appropriate `shadow` utility to not invalidate
the database format:

- secure user account information, `/etc/shadow`
- plain text user account information, `/etc/passwd`
- shadowed group account information, `/etc/gshadow`
- user groups definition, `/etc/group`

Managing users requires several different commands depending on the situation. Here, some common examples are listed. To view users logged on the system, the `who` command
is used. For all existing user accounts and their properties stored in the database, run:

<pre>passwd -Sa</pre>

To change the default values for newly created users, edit the `/etc/default/useradd` file, and display them via the

<pre>useradd --defaults</pre>

command. For example, the variable <code>SHELL=/usr/bin/<i>shell</i></code> sets the global default login shell. For the login to function, said shell must be available in
the `/etc/shells` file, which can be checked by running:

<pre>chsh -l</pre>

New user home directories can also be automatically populated with certain files, specified by adding them to the `/etc/skel` directory.

> **If an invalid shell is selected, the PAM module will deny the login request.**

Adding a user is done as follows:

<pre>useradd -m -G <i>groups</i> -s <i>shell</i> <i>username</i></pre>

Here, the `m` flag creates the <code>/home/<i>username</i></code> directory, which is owned by the new user. After `G` follows a comma separated list of supplementary groups
to which the new user also belongs, other than the default initial group of the same name. For a different initial group, use the `g` flag, which is however not recommended.
With the `s` parameter, the path to a login shell must be entered. The `h` flag prints all available options. 

> **If the login shell is intended to be disabled, for example when creating user accounts for specific services, the path `/usr/bin/nologin` may be used to refuse a login.**

After creating the new user, a password should be set using the

<pre>passwd <i>username</i></pre>

command. Creating a system user, useful for using permissions to harden the system and isolate services, with specified user and group identifiers would look something like this:

<pre>useradd -r -u <i>uid</i> -g <i>gid</i> -s /usr/bin/nologin <i>username</i></pre>

Editing an existing user account is also possible. Run

<pre>usermod -d /<i>new</i>/<i>home</i> -m <i>username</i></pre>

to change the home directory and move all content there. For programs with hardcoded paths, it can be useful to symlink from the previous to the new location:

<pre>ln -s /<i>new</i>/<i>home</i>/ /<i>former</i>/<i>home</i></pre>

> **Make sure there is no trailing `/` after the <code>/<i>former</i>/<i>home</i></code> path.**

In case a login name needs to be changed,

<pre>usermod -l <i>newname</i> <i>oldname</i></pre>

achieves just that.

> **Be certain that you are not logged in as the user whose name you are about to change.**

There are some more things to keep in mind after a name change:

- update your `/etc/sudoers` to reflect new usernames via the `visudo` command as root
- adjust crontabs by renaming the user file in `/var/spool/cron` from the old to the new name before opening `crontab` with the `e` flag to change any relevant
  paths and have it set the file permissions accordingly
- avoid problems with shell scripts by using the `~` or `$HOME` variables for the home directory location
- edit those configuration files in `/etc/` that rely on absolute paths, and find such files with

  <pre>grep -r <i>oldname</i> *</pre>

The following lists some more examples for user management actions. To interactively enter information for the GECOS comment, type:

<pre>chfn <i>username</i></pre>

A user password can be marked as expired, prompting the creation of a new one for the next login:

<pre>chage -d 0 <i>username</i></pre>

Delete accounts with the

<pre>userdel -r <i>username</i></pre>

command, where the `r` option specifies that the user home directory and mail spool should also be deleted.

Change the login shell:

<pre>usermod -s <i>shell</i> <i>username</i></pre>

As for the database, local user information is stored as plain text in `/etc/passwd` with each line referring to a different user:

<pre><i>account:password:UID:GID:GECOS:directory:shell</i></pre>

- `account` is the username and cannot be blank
- `password` is the user password and should only point to the associated shadowed file
- `UID` is the numerical user identifier
- `GID` is the numerical group identifier
- `GECOS` is an optional field used for informational purposes and usually contains the full username
- `directory` is used by the login command to set the home environment variable
- `shell` is the path to the user default command shell

Verifying the integrity of the user database is achieved via the

<pre>pwck -s</pre>

command, with the `s` flag sorting by `UID` at the same time for ease of comparison. These checks can be automated with the `shadow.timer` that starts `shadow.service` in
daily intervals to run `pwck` and `grpck` for both password and group files. If discrepancies are reported, the `vigr` and `vipw` commands can edit groups and users,
respectively, while locking the databases for editing as an extra margin of protection. Security can also be improved by packages migrating from sysusers to fully locked
system accounts that enable locked and expired status. These users need to be modified manually.

Similar to users, groups are defined in `/etc/group` as well as the accompanying but rarely used shadowed file, and are managed as shown in the following examples. Display
membership with the

<pre>groups <i>user</i></pre>

command and list additional details by running:

<pre>id <i>user</i></pre>

To list all groups on the system:

<pre>cat /etc/group</pre>

Create new groups:

<pre>groupadd <i>group</i></pre>

> **If the user is currently logged in, they must log out and in again for changes to take effect.**

Add users to a group using the `a` option:

<pre>gpasswd -a <i>user</i> <i>group</i></pre>

Alternatively, add a user to additional groups:

<pre>usermod -aG <i>groups</i> <i>username</i></pre>

> **If the `a` flag is omitted, the user will be removed from any previous groups not listed in the command.**

Modify groups, such as renaming with the `n` option:

<pre>groupmod -n <i>newgroup</i> <i>oldgroup</i></pre>

This will change the name but not the identifier, meaning ownership is transferred. To delete existing groups:

<pre>groupdel <i>group</i></pre>

To remove users from a group:

<pre>gpasswd -d <i>user</i> <i>group</i></pre>

> **Arch Linux defaults of group files are created as `.pacnew` files by new releases of the `filesystem` package. Unless Pacman outputs related messages for action, these
`.pacnew` files can, and should, be disregarded or removed.**

This last section explains the purpose of some essential groups from the `filesystem` package. There are many other groups, which will be created with correct identifiers
when the relevant package is installed.

> **A later removal of a package does not remove the automatically created user and group again. This is intentional because any files created during its usage would
otherwise be left orphaned as a potential security risk.**

User Groups:

- `adm` has full read access to journal files, commonly gives access to protected logs, administration group
- `ftp` has access to files served by FTP servers, affects `/srv/ftp`
- `games` has access to some game software, affects `/var/games`
- `http` has access to files served by HTTP servers, affects `/srv/http`
- `log` has access to log files in `/var/log/`
- `rfkill` has the right to control wireless devices power state, affects `/dev/rfkill`
- `sys` has right to administer printers in CUPS
- `wheel` has the right to administer printers in CUPS and full read access to journal files, commonly gives privileges to perform administrative actions, can give access to
  `sudo` and `su` utilities although not as default, administration group

System Groups:

- `dbus` is used internally by `dbus`
- `locate` is used by `plocate`
- `nobody` is an unprivileged group
- `proc` is authorized to learn processes information otherwise prohibited by the `hidepid` mount option of the proc filesystem, must be explicitly set with the `gid` mount
  option, affects <code>/proc/<i>pid</i>/</code>
- `root` is for complete system administration

Legacy Groups:

- `audio` grants direct access to sound hardware, required for remote ALSA or OSS sessions, otherwise not recommended
- `disk` grants access to block devices not affected by other groups
- `input` grants access to input devices
- `kvm` grants access to virtual machines using KVM
- `optical` grants access to optical devices like CD or DVD drives
- `storage` grants access to removable drives, enables mounting storage devices, required for manipulating some devices via `udisks` or `udisksctl`
- `video` grants acces to video capture devices, 2D and 3D hardware acceleration, framebuffer

> **Before `systemd` was introduced, users had to be added manually to these. Now, `udev` marks devices with a `uaccess` tag and `logind` assigns user permissions dynamically
according to the active session. There are still some exceptions, especially for remote access.**

#### Wildcards

Standard globbing patterns are widely used as placeholders with different functions:

- `?` represents any single character or number
- `*` represents any number of characters
- `[]` specifies a range `-` or list `,` to match
- `{}` expands a list `,` where each item has to be a filename or wildcard
- `[!]` specifies a range `-` or list `,` to exclude
- `\` and `''` are escape characters that protect special symbols

#### Permissions & Ownership

One of the core unifying ideas behind UNIX operating systems is the mantra that almost everything is a file. Files provide a common abstraction for documents, directories,
storage, networking, input and output devices, all exposed through the same API and beholden to the same set of basic commands. For example, on many systems, audio recording
and playback can be done via the `cat` utility, just as easily as piping content to standard out:
<pre>cat /dev/audio > <i>file</i></pre>
<pre>cat <i>file</i> > /dev/audio</pre>
Each file has permissions that can be given to the owning user and group individually, as well as others without ownership over said file. At this point, to explain these
parameters, it is useful to look at the output of the
<pre>ls -lha</pre>
command, which displays the total number of blocks as the first line and proceeds to list each file stored in the current directory according to the following format:
<pre><i>permissions linkcount owner group size moddate name</i></pre>

- `permissions` defines who can do what with the file, as is explained below
- `linkcount` gives the number of links contained
  - for regular files, this is just the link to itself, so always `1`
  - for directories, which are files themselves, it is at least `2` for the self `.` and the parent `..` directory, with each subdirectory incrementing the counter by one
- `owner` is the owning user account name
- `group` is the owning group name
- `size` gives the amount of storage reserved
  - for regular files, this is just the actual size in bits or bytes
  - for directories, it is the size required to store the links to the contained files, for typical filesystems always as integer multiples of the block size
- `moddate` is the date on which the file was last modified and takes up three columns
  - first, the month is displayed as `MMM`
  - second, the day is displayed as `D`
  - third, if the date is in the same calendar year, the time is displayed as `hh:mm` or else, the year is displayed as `YYYY`
- `name` is the file identifier

> **With the `a` flag, running `ls` shows hidden files which have names starting with `.` and are otherwise excluded. The `h` option makes the size formats human readable.
And the `l` parameter makes it so that the long format is output instead of just the names in a shortened list.**

Now, the structure of the mode string that defines the file permissions can be split into five fields:

- The first field comprises one character and indicates the file type, with POSIX specifying the following options:
  - regular file, `-`
  - directory, `d`
  - symbolic link, `l`
  - FIFO pipe special, `p`
  - block special, `b`
  - character special, `c`
  - socket, `s`
- The second, third and fourth fields comprise three characters each, with every character triple indicating the permissions for the owner, group and others, respectively:
  - The first character defines read permissions:
    - `-` means that the file cannot be read or that the directory contents cannot be shown
    - `r` means that the file can be read or that the directory contents can be shown
  - The second character defines write permissions:
    - `-` means that the file or the directory contents cannot be modified
    - `w` means that the file or the directory contents can be modified, for directories this enables creating new files or renaming and deleting existing ones, and only
      has an effect if the execute permission is also set
  - The third character defines the execute permissions:
    - `-` means that the file cannot be executed or that the directory cannot be accessed with `cd`
    - `x` means that the file can be exectued or that the directory can be accessed with `cd`
    - `s` sets the bit for `setuid` in the user triad or `setgid` in the group triad to run binary executables under owner filesystem permissions, with no effect in the
      others triad, and also implies `x` is set
    - `S` does the same as `s` but without setting `x`
    - `t` sets the sticky bit in the others triad, with no effect otherwise, to protect directories from hijacking by deleting, overwriting of renaming files from anyone
      but the file or directory owner, and also implies that `x` is set
    - `T` does the same as `t` but without setting `x`
- The fifth field comprises one character and indicates alternate access methods, if not left empty with a space:
  - `.` marks a file with a security context but no further alternate access method
  - `+` marks a file with any other combination of alternate access methods

Some examples:

- `drwxr-xr-x` describes a directory whose owner can view and modify its content, as well as use the `cd` command, whereas the group and others can only view and `cd` into
  it without making modifications, and no alternate access methods being available
- `-rw-r--r--` describes a regular file whose owner can read and write, but is not able to execute it, whose group and others can only read its contents, and for which no
  alternate access method is available

It is also possible to get just the owning user or group, or the access rights for a specific file:

<pre>stat -c %U <i>file</i></pre>
<pre>stat -c %G <i>file</i></pre>
<pre>stat -c %A <i>file</i></pre>

To list all files owned by a specific user or group, use:

<pre>find / -group <i>groupname</i></pre>
<pre>find / -group <i>groupnumber</i></pre>
<pre>find / -user <i>user</i></pre>

If one wants to change the permissions of a file, there are different methods based on the `chmod` command. In the text based method, setting access modes follows

<pre>chmod <i>who</i>=<i>permissions</i> <i>filename</i></pre>

as the syntax. The `permissions` are the same as before with `r` for read, `w` for write, and `x` for execute rights, while the `who` argument can take four letter values:

- `u` for the user that owns the file
- `g` for the group the file belongs to
- `o` for any others without ownership
- `a` for all the above, replacing the `ugo` combination

A thing to look out for when using this approach is that any previously defined permissions of the specified triads are overwritten. To only modify existing permissions,
use `+` or `-` to grant or revoke rights instead of the `=` sign. One can also copy permissions from one class to another, although this cannot be combined with modifying
permissions in the same command:

<pre>chmod <i>who</i>=<i>who</i> <i>filename</i></pre>

Now some examples:

- <code>chmod o= <i>filename</i></code> denies any permissions to the others class
- <code>chmod go=rw <i>filename</i></code> grants read and write permissions to both group and others classes
- <code>chmod g+w <i>filename</i></code> adds write access to the permissions of the group class
- <code>chmod a-w <i>filename</i></code> removes write access from the permissions of all classes
- <code>chmod g=u <i>filename</i></code> gives the group the same rights as those belonging to the owner class

In the numeric method, numbers are used instead of letters for all three classes at the same time, making the process shorter but also less readable. In the

<pre>chmod <i>nnn</i> <i>filename</i></pre>

command, `nnn` is a three digit number, with each digit having a value in the range of `0` to `7` and subsequent digits applying permissions for owner, group, and others
classes, respectively. The previously defined rights are mapped to numbers as follows:

- `r` to `4`
- `w` to `2`
- `x` to `1`

Summing any combination of these results in a value between `0` and `7` unique to this specific combination, allowing one to set the same permissions options as before.
To view the access mode of a file in this form, run:

<pre>stat -c %a <i>file</i></pre>

Typically, directories have `755` to allow reading, writing and execution to the owner, but deny writing to anyone else, and regular files are normally `644` to allow
reading and writing to the owner but only reading to anyone else. A couple examples of this:

- <code>chmod 775 <i>filename</i></code> is equivalent to combining <code>chmod ug=rwx <i>filename</i></code> and <code>chmod o=rx <i>filename</i></code>
- <code>chmod 664 <i>filename</i></code> is equivalent to combining <code>chmod ug=rw <i>filename</i></code> and <code>chmod o=r <i>filename</i></code>

One can also use binary numbers by simply converting each decimal system digit, such that:

- `000` is `0`
- `001` is `1`
- `010` is `2`
- `011` is `3`
- `100` is `4`
- `101` is `5`
- `110` is `6`
- `111` is `7`

Furthermore, the `setuid` and `setgid` as  well as `sticky` bits can be set as the first of four decimal digits, with encoding:

- `setuid` to `4`
- `setgid` to `2`
- `sticky` to `1`

Directories and files should generally not have the same permissions. If it is necessary to bulk modify a directory tree, use find to selectively modify one or the other:

<pre>find <i>directory</i> -type d -exec chmod 755 {} +</pre>
<pre>find <i>directory</i> -type f -exec chmod 644 {} +</pre>

In case the owner of a file needs to be changed, the `chown` command is employed. To set a new owning user, run:

<pre>chown <i>username</i> <i>filename</i></pre>

> **This always clears the `setuid` and `setgid` bits.**

Apart from the mode bits discussed until now, several filesystems support more attributes that allow for further customization of performed file operations.

> By default, these file attributes are not preserved by programs such as `cp` or the `rsync` utility.

A few useful attributes:

- allow opening only to append, `a`
- enable compression on filesystem level, `c`
- make immutable, cannot be modified, deleted, renamed or linked to, `i`
- use the journal for file data writes and metadata, `j`
- disable compression on filesystem level, `m`

These can be set and removed with `chattr` and the `+` or `-` options. For example, enabling immutability or turning off data journaling would look like this:

<pre>chattr +i /<i>path</i>/<i>to</i>/<i>file</i></pre>
<pre>chattr -j /<i>path</i>/<i>to</i>/<i>file</i></pre>

Lastly, a tip to prevent `chmod` from recursively removing the executable bit systemwide when acting on `/` and thus breaking the system, use the `--preserve-root` flag or
set it as an alias.

#### Security

#### Daemons

#### Updates & Upgrades & Backup

<br><br>

### Packages

#### Manager

#### Repositories

#### Mirrors

#### ABS

#### AUR

<br><br>

### Booting

#### Hardware Recognition

#### Microcode

#### Retaining Messages

#### Function Lock

<br><br>

### User Graphics

#### Display Server

#### Drivers

#### Window Managers & Compositors

#### Login Manager

#### User Directories

<br><br>

### Power Management

#### ACPI Events

#### CPU Frequency Scaling

#### Laptops

#### Suspend & Hibernate

<br><br>

### Multimedia

#### Sound System

<br><br>

### Networking

#### DNS

#### Firewall

#### Network Shares

<br><br>

### Input Devices

#### Keyboard Layout

#### Mouse Buttons

#### Touchpads

<br><br>

### Optimization

#### Benchmarks

#### Performance

#### TRIM

<br><br>

### System Services

#### File Index & Search

#### Local Mail Delivery

#### Printing

<br><br>

### Appearance

#### Fonts

#### Themes

<br><br>

### Console Improvements

#### Completion Enhancements

#### Aliases

#### Alternative Shells

#### Colored Output

#### Compressed Files

#### Pointer Support

#### Session Attachments

<br><br>

*Adapted from the [Arch Wiki](https://wiki.archlinux.org/)*.
