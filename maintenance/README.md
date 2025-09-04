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

In the process of hardening an Arch Linux system, there are a few concepts one should be aware of:

- There is a balance between security and convenience, the goal can only be a secure and useful system, not tightening security to the point the system is unusable.
- The biggest threat is, and will always be, the user.
- Each part of a system should only be able to access what is strictly required, the principle of least privilege.
- Security works best if, when one layer is breached, another independent layer stops the attack, defense in depth.
- It is best practice to be a little paranoid and suspicious.
- To make a system absolutely secure, the only way is to never use it.
- Create a plan ahead of time to follow if and when security is broken, prepate for failure.

Additionally, there are some mostly common sense directions to follow:

- Only create and use safe password, with length and entropy being key. Maintain the database containing your passwords securely, regularly check for compromised phrases and
  change those if necessary. It is useful to understand how the `passwd` command hashes passwords by stretching and salting to defend against attacks such as precomputed
  rainbow tables. One can also set up automatic checks that enforce password quality.
- Keep your processor microcode up to date for important security updates. Check for hardware vulnerabilities and mitigations by running:

  <pre>grep -r . /sys/devices/system/cpu/vulnerabilities/</pre>

  When hosting untrustworthy virtualization guests as a hypervisor, one might want to disable simultaneous multithreading by setting the `nosmt` kernel parameter.
- To improve memory safety, hardened malloc, which was originally developed for [GrapheneOS](https://grapheneos.org/) Android versions, can be used to replace the `glibc`
  default.
- Encrypting data at rest, when the machine is turned off or the disks are unmounted, is the only way to guard against physical recovery. To provide data confidentiality,
  full disk encryption with a strong passphrase is preferred. The kernel automatically prevents security issues related to hardlinks and symlinks if the appropriate switches
  are enabled. Filesystems should be mounted with the most restrictive mount options possible, which are:
  - do not interpret character or block special devices on the file system, `nodev`
  - do not allow `setuid` and `setgid` bits to take effect, `nosuid`
  - do not allow direct execution of any binaries on the mounted filesystem, `noexec`
  Filesystems used for data should always have all of these set. One should also be aware that snapshots may contain sensitive information that users expect to be deleted.
  Additionally and as mentioned earlier, file access permissions should be restricted whenever feasible, especially for `nftables` and `iptables` as well as `boot` directories.
  Another consideration is changing the `umask` from `0022` to `0077` for maximum security, to make new files not readable for users other than the owner. It is important to be
  aware of any files with SUID or SGID bits set, to limit the attack surface from privilege escalation vulnerabilities. To search for such files, run:
  <br>
  <pre>find / -perm "/u=s,g=s" -type f 2>/dev/null</pre>
- In the user setup, one should ensure to not use the root account for daily tasks. To massively slow down even intelligent brute force attacks, enforce a delay after failed
  login attempts by adding this line to the `/etc/pam.d/system-login` file:

  <pre>auth optional pam_faildelay.so delay=<i>time</i></pre>

  Here, the time is given in units of microseconds.

  > **Other PAM modules besides `pam_faildelay` can suggest a delay as well, such as `pam_unix` and `pam_faillock` with a default of two seconds. In this case PAM will always
  use the longest time.**

  Further, users can be locked out after multiple failed login attempts, except the root user to prevent complete denial of service. Unlock a user with:

  <pre>faillock --user <i>username</i> --reset</pre>

  On systems with many or untrusted users, the number of processes each can run should be limited to prevent fork bombs, wherein a process continuously replicates itself to
  induce resource starvation. Finally, Wayland is preferrable versus Xorg from a security perspective, for example to prevent keystroke recording by inactive applications.
  If still required, it should be run rootless such as under the Xwayland compatibility layer.
- As the most powerful user on any system, root user account usage should be restricted as much as possible. Due to this, `sudo` should be used instead of `su` as it provides
  several advantages when giving privileged access while limiting the potential harm:
  - It keeps a log of which normal privilege user has run each privileged command.
  - The root user password need not be given out to each user who requires root access.
  - Users are prevented from accidentally running commands as root that do not need root access, because a full root terminal is not created.
  - Individual programs may be enabled per user, instead of offering complete root access just to run one command.
  For `sudo` settings, edit the `/etc/sudoers` file with the `visudo` command, which locks the file, creates a temporary version and only copies to the original after checking
  for syntax errors. If `vi` is not your preferred editor, you can `visudo` to set the default in the `/etc/sudoers` file:
  <br>
  <pre>Defaults editor=/usr/bin/<i>editor</i>, !env_editor</pre>

  Recommended editors are the restricted `rvim` or `rnano` versions. The above line also disallows `visudo` to export the `EDITOR` and `VISUAL` environment variables, which
  would be a severe security risk, because anything can be chosen as an editor. Some example entries that define rights are given below. To give a user access to a
  particular program, use `visudo` to enter:

  <pre><i>username</i> ALL = NOPASSWD: /<i>path</i>/<i>to</i>/<i>program</i></pre>

  And to allow members of group `wheel` access to the `sudo` command:

  <pre>%wheel ALL=(ALL:ALL) ALL</pre>

  > **The most customized option should go at the end of the file, as the later lines override the previous ones.**

  The file owner, group and permissions for `/etc/sudoers` should always be set to the defaults, otherwise `sudo` will fail:
  <pre>chown -c root:root /etc/sudoers</pre>
  <pre>chmod -c 0440 /etc/sudoers</pre>

  A common annoyance is a long running process that runs on a background terminal somewhere that runs with normal permissions and elevates only when needed. This leads to a `sudo`
  password prompt which goes unnoticed and times out, at which point the process dies and the work done is lost or, at best, cached. To deal with this, the prompt timeout can be
  disabled, and since it does not serve any reasonable security purpose it should be the solution here, despite contradicting some other common advice:

  <pre>Defaults passwd_timeout=0</pre>

  Since Zsh and Bash normally only expand aliases for the first word in a command, this will not work after `sudo` is envoked. To circumvent this, you can make the next word
  expand by aliasing `sudo` to end in a space, by entering the following in your shell configuration file:

  <pre>alias sudo='sudo '</pre>

  In order to draw attention to a `sudo` password prompt, even when running in a background terminal, the `SUDO_PROMPT` environment variable can be exported in your shell
  initialization with `tput` usage, setting for example a bold colored prompt, or including a bell character. If you do not want to enter your password for each newly opened
  terminal, use `visudo` to set:

  <pre>Defaults timestamp_type=global</pre>

  > **This will let any process use your sudo session.**

  Reduce the number of times you have to type your password by increasing the default five minute interval:

  <pre>Defaults timestamp_timeout=<i>minutes</i></pre>

  This can be refreshed with the `v` and immediately revoked with the `K` flag. If you have a lot of environment variables, or you export your proxy settings, when using `sudo`
  these variables do not get passed to the root account unless you run with the `E` option. The recommended way of preserving environment variables is to append them like this:

  <pre>Defaults env_keep += "ftp_proxy http_proxy https_proxy no_proxy"</pre>

  Users can configure `sudo` to ask for the root password instead of the user password by adding a `targetpw` or `rootpw` default:

  <pre>Defaults targetpw</pre>

  To prevent exposing your root password to users, you can restrict this to a specific group:

  <pre>Defaults:%wheel targetpw<br>%wheel ALL=(ALL) ALL</pre>

  Disable asking password prompts altogether for a specific user as follows:

  <pre>Defaults:<i>username</i> !authenticate</pre>

  > **This will allow any process running with your user name to use sudo without asking for permission.**

  To conveniently edit files, use `sudoedit` as an equivalent to the `e` flag. Set `SUDO_EDITOR` and pass it to the command, for example:

  <pre>SUDO_EDITOR=vim sudoedit <i>file</i></pre>

  If multiple files are passed, all are opened in the editor with a single invocation, a feature useful for merging files:

  <pre>SUDO_EDITOR=vimdiff sudoedit <i>file</i> <i>file</i>.pacnew</pre>

  Lastly, writing to protected files after forgetting `sudo` before starting Vim, for example, you can do the following in Vim to still save the file:

  <pre>:w !sudo tee %</pre>

  Add this to your `~/.vimrc` with the common `:w!!` mapping:

  <pre>cmap w!! w !sudo tee > /dev/null %</pre>

  Once `sudo` is properly configured, full root access can be heavily restricted or denied without losing much usability. To disable `root` and still allow using sudo, you
  can use

  <pre>passwd --lock root</pre>

  and even edit `/etc/security/access.conf` to specify acceptable login combinations.
- Another type of security policy that differs significantly from the default discretionary access control is mandatory access control, checking any action against a ruleset.
- Kernel hardening includes self protection and exploit mitigation, hiding `pid` values and restricting module loading, as well as disabling `kexec` and the emergency shell.
- Applications can be sandboxed up to full virtualization containers like VirtualBox or KVM to improve isolation and security, especially in the event of running risky
  applications or browsing dangerous websites.
- Setting up a basic firewall is highly recommended to protect services running on the system from network attacks. The stock Arch Linux kernel is capable of using `iptables`
  and `nftables` provided by the Netfilter framework, although these services are not enabled by default. As a general recommendation, use the more modern and efficient, non
  locking `nftables` over the older `iptables` option, with translation layers like `iptables-nft` being available and usually sufficient for `docker` and other containers,
  for example. The current ruleset can be printed with:

  <pre>nft list ruleset</pre>

  To remove all rulesets, leaving the system with no firewall, run:

  <pre>nft flush ruleset</pre>

  A simple firewall configuration is stored in `/etc/nftables.conf` and loaded by `nftables.service` when starting or enabling it. Tables hold chains, which themselves contain
  rules that are constructed from expressions or statements, including named or anonymous sets. Atomic reloading is achieved as follows:

  - Flush the current ruleset:

    <pre>echo "flush ruleset" > /tmp/nftables</pre>

  - Dump the current ruleset:

    <pre>nft -s list ruleset >> /tmp/nftables</pre>

  - Now you can edit `/tmp/nftables` and apply your changes:

    <pre>nft -f /tmp/nftables</pre> 

  Save the current ruleset:

  <pre>nft -s list ruleset | tee <i>filename</i></pre>

  In lieu of not explicitly knowing which services are worth protecting, it is a good precaution to enable a stateful firewall. You can log traffic,

  <pre>nft add rule inet filter input log</pre>

  for all incoming packets, or monitor events by listening to all:

  <pre>nft monitor</pre>

  To avoid hardcoding, `systemd-networkd` can be used for dynamic named sets.

  Some services listen for inbound traffic on open network ports. It is important to only bind these services to the addresses and interfaces that are strictly necessary, otherwise
  it may be possible for a remote attacker to exploit flawed network protocols to access exposed services. This can even happen with processes bound to localhost. In general, if a
  service only needs to be accessible to the local system, bind to a Unix domain socket or a loopback address such as localhost instead of a non loopback address. If a service needs
  to be accessible to other systems via the network, control the access with strict firewall rules and configure authentication, authorization and encryption whenever possible. You
  can show all listening processes as well as their numeric `tcp` and `udp` port numbers:

  <pre>ss -lpntu</pre>

  Kernel parameters affecting networking can be set with `sysctl` for stack hardening. To mitigate SSH brute force attacks, enforce key based authentication, with some procedures
  using a second OTP factor. The default DNS configuration sacrifices privacy and security for compatibility, so consider adapting it to your needs. To prevent remote code
  execution due to DNS resolver bugs, a caching server can be used as a proxy. Also manage trust for TLS and deprecated SSL certificates.
- Physical access to a computer will grant root access with enough time and resources, but a high practical level of security can still be achieved by putting up enough barriers.
  Determined and knowledgeable attackers can gain control during the next boot by attaching malicious FireWire, Thunderbolt or PCIe devices, which are given full memory access by
  default, or gain access to your data by installing malicious malware. Important steps to harden the physical security include:
  - Lock down BIOS access by adding a password so that removable devices cannot be booted from. Make sure your drive is first in the boot order and disable other drives from being
    bootable if possible.
  - It is highly important to protect your boot loader, because while being unprotected, it can bypass any login restriction by setting the `init=/bin/sh` kernel parameter, for
    example. Editing kernel parameters is disabled when `systemd-boot` is used, at least as long as Secure Boot is enabled. Other boot managers will differ.
  - Turning on Secure Boot, a feature of UEFI to allow authentication of the files your computer boots, helps in preventing evil maiden attacks. Normally, computers come with OEM
    vendor keys, which can however be removed, allowing users to enroll and manage their own keys with Setup Mode at the risk of irreversibly bricking the hardware.
  - The TPM includes hardware microprocessors which have cryptographic keys embedded. This forms the fundamental root of trust of most modern computers and allows end to end
    verification of the boot chain. They can be used as internal smartcards, attest the firmware running on the computer and allow users to insert secrets into a tamper proof
    and brute force resistant store.
  - One popular idea is to place the boot partition on a separate flash drive in order to render the system unbootable without it. Proponents of this idea often use full disk
    encryption alongside, and some also use detached encryption headers placed on the boot partition.
  - In Bash and Zsh, a `TMOUT` can be set to automatically logout from shells after a timeout.
  - Another protection to consider is one against rogue USB devices.
  - Lastly, powered on computers may be vulnerable to volatile data collection. Prevent this by turning the computer off whenever sensible, especially when the physical security
    is temporarily compromised, such as at security checkpoints.
- Packages should be authenticated with proper signature systems and upgraded regularly to minimize attack surfaces. Follow vulnerability alerts and rebuild packages stripped of
  undesired functions.

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
