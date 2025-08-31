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
- `password` is the user password and should only point to the shadowed file
- `UID` is the numerical user identifier
- `GID` is the numerical group identifier
- `GECOS` is an optional field used for informational purposes and usually contains the full username

#### Wildcards

#### Permissions & Ownership

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
