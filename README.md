# gksudo-pk
### gksudo-pk [-u | --user \<user\>] \<command\>
#### gksudo [-u | --user \<user\>] \<command\>
#### gksu [-u | --user \<user\>] \<command\>
A drop-in replacement for **gksu** and **gksudo**, with fewer options. For X11 only. Gksudo-pk is a simple bash script. **Pkexec** is used to launch graphical programs as root, or as another user. It does not use xhost, or call xauth directly. **Sudo** and **Zenity** are required dependencies. This script is **NOT SECURE** by modern standards, although **it will always send a warning notification** to the X11 desktop. Use is not recommended on multiple networked machines, with ssh, or unless behind a firewall. Convenience is attained at the expense of security. **Use at YOUR OWN RISK**. Tested and hopefully works in multiple desktop environments, including **KDE Plasma, XFCE, MATE, GNOME(Xorg), LXQT**. Works in both **systemd (Arch)** and **non-systemd (Void)** systems. An important feature (and vulnerability) is that the administrator may assign programs to one of two strings within the script:

- **NOPASSWD_LIST: These programs may be run without password authentication.**
- **NEVER_AUTH_LIST: These programs are prohibited entirely from launching with gksudo-pk.**

**THE NEED FOR A SEPARATE POLKIT RULE FOR EACH APPLICATION IS THEREFORE ELIMINATED**. Only one new polkit action/rule pair is needed.  All other progams will be subject to default polkit/pkexec rules.  Also, If gksudo-pk polkit action/rules are not installed, NOPASSWD_LIST is ignored, and default polkit rules for pkexec will be used for all authorized programs.

## Dependencies
**bash, sudo, zenity**


## Options
Only the **--user | -u** options are actually used, and as with sudo, may be omitted for "**-u root**".  All other options accepted by the original **gksudo** are looked for and stripped.  The remaining arguments are then passed to pkexec with an environment (see below)

## Applicability
gksudo-pk is designed to be fairly universal, but has not been extensively tested. Desktop environments tested so far include:
**XFCE 4.14, KDE Plasma 5, MATE 1.24, Gnome 3.32**(Xorg only). Both **systemd** (Arch) and **non-systemd** (Void) distributions have been tested. **LXQT 0.14** works fully, but not with lxqt-policykit-agent (see notes below). gksudo-pk works with the following display managers: **none(startx), xdm, slim, lxdm, lightdm, gdm**.

## Details
The invoking user MUST be a member of **$Admin_Grp**, which defaults to **"wheel"**.  Also, either the group or the user **must be a sudoer**. The **-u | --user** option allows gksudo-pk to run a program as **ANY STANDARD USER, as well as root**.  However, $NOPASSWD_LIST will be ignored if the program will be run as a user other than root, and default polkit rules will apply.  

An important key feature of gksudo-pk is the creation of a proper environment for pkexec to use.  **/etc/environment** is sourced, and **Xauthority** and **Display** variables are borrowed/provided, as well as some specific variables for KDE, if needed.  While this is a larger environment than the basic one pkexec uses by default, it is still minimal compared to that of a regular user.  Importantly, a polkit **"org.freedesktop.policykit.exec.allow_gui"** annotation is **NOT required** for graphical programs to run.

A second important feature is the placement of the environment within one of two temporary executable scripts, "**/tmp/gk/passwd-env**" or "**/tmp/gk/nopasswd-env**".  Creation of a polkit rule then allows different authorization protocol based on THE NAME of the script.  The administrator may then assign programs to normal, lesser, (or greater, with modifications) polkit authorization, all by changing two strings within the gksudo-pk script.

The administrator/installer is expected to check or modify four variables near the beginning of the script:
- **NOPASSWD_LIST**   (author's list left as default, change to fit your situation)
- **NEVER_AUTH_LIST**  (author's list left as default, change to fit your situation)
- **Admin_Grp="wheel"**   (only one group is allowed to run gksudo-pk. A change here must by carried over to polkit action/rule if in use)
- **Limit_Sudo_Use=false**   (Allows secure logging, and creation of XDG_RUNTIME_DIR directories, which reduce warnings in Plasma5, and potential errors.  See notes below)

## Logging
gksudo-pk by default will create it's own log at **/var/log/gksudo-pk.log** if $Limit_Sudo_Use=false. If $Limit_Sudo_Use=true, the log will be kept in the invoking user's home directory **~/gksudo.pk.log**.  The entries are not errors, which usually log elsewhere, but instead simple records of the attempted calling of gksudo-pk, and are made whether the pkexec command actually succeeds or fails. 

## Installation
It is not difficult to install this script, but the author does not encourage "packaging" it, due to the security concerns.  To install, install sudo and zenity, then download the files. Modify the following if your polkit folders are located differently for your distribution.  Ensure that $PATH includes /usr/local/bin, unless placing the links in /bin or /usr/bin instead. From the download directory, do the following AS ROOT:

- cp 	gksudo.pk*-env.policy /usr/share/polkit-1/actions/
- cp 49-gksudo-pk-nopasswd-env.rules /etc/polkit-1/rules.d/
- cp gksudo-pk /usr/local/bin/
- chmod 0755 /usr/local/bin/gksudo-pk
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksudo  # recommended to replace "gksudo"
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksu    # recommended to replace "gksu"
 
## Notes
A common warning complains about "inability to register with accessibility bus" or similar.  This warning can be silenced by appending **NO_AT_BRIDGE=1** to **/etc/environment**.

For Plasma5 and Dolphin, "XDG_RUNTIME_DIR missing" (and other) warnings abound if $Limit_Sudo_Use is true, but usually no functional errors occur. With $Limit_Sudo_Use = false (the default), gksudo-pk creates and assigns temporary $XDG_RUNTIME_DIR directories to start (dolphin) more quickly and reduce some warnings.  "$Limit_Sudo_Use=false" also provides at least some marginal security by restricting write access to the log, but this will generate more sudo password prompts that are a separate authentication from polkit/pkexec.  This can be avoided by giving the group or user a NOPASSWD: setting in /etc/sudoers or /etc/sudoers.d, or by setting $Limit_Sudo_Use=true. Again, assess your risk.

gksudo-pk, or it's gksu/gksudo link, can be used in .desktop files or custom actions, allowing the elevation of privileges with a right mouse click or menu selection from within most common file managers. You will be warned when doing this. In single or 2-3 user situations, this can be very convenient for EXAMINING protected files or directories, but should not be abused by indiscriminately (or accidentaly) deleting or modifying them with gksudo-pk.  Never elevate to superuser when there is no need.

The script relies heavily on creating small temporary files in the /tmp directory.  Obviously, it will run faster and be easier on drives if /tmp is a tmpfs in RAM.

lxqt-policykit-agent seems to be partially broken as of 3/2020, both in systemd and non-systemd distros. This only affects invoking gksudo-pk to run as another sudoer or wheel member, **other than root**.  The blame has been placed on upstream polkit, yet no other polkit agent seems to have the same problem.  **IF you only use gksudo-pk -u root** (the default), **the script will work as expected in LXQT**.  If there are two or more users who are sudoers or members of $Admin_Grp (wheel), it seems that ANY use of auth_admin in a polkit rule will fail after the password is entered, **if the user change is other than to root itself**. The error is "Error executing command as another user: Not authorized". This occurs even for the SAME USER. This forces the administrator to change relevant polkit rules to return "YES", or change polkit actions from "auth_admin" to "YES", which lessens security even more. Consider installing another polkit agent, such as **polkit-kde-agent**. It is unnecessary to remove lxqt-policykit-agent, just uncheck the box for it in LXQT preferences. Then in Preferences > LXQt Settings > Session Settings, add an autostart item for polkit-kde-agent.
