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

command. For example, the variable <code>SHELL=/usr/bin/<i>shell</i></code> sets the global default login shell.

#### Permissions & Ownership

#### Security

#### Daemons

#### Updates & Upgrades

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
