# gksudo-pk
### gksudo-pk [-u | --user \<user\>] \<command\>
### gksudo [-u | --user \<user\>] \<command\>
A drop-in replacement for **gksudo**, with fewer options. For X11 only. **Pkexec** is used to launch graphical programs as root, or as another user. **Sudo** and **Zenity** are required dependencies. This bash script is **NOT SECURE** by modern standards. Use is not recommended on multiple networked machines, with ssh, or unless behind a firewall. Convenience is attained at the expense of security. **Use at YOUR OWN RISK**. Tested and hopefully works in multiple desktop environments, including KDE Plasma, XFCE, MATE, LXQT. Works in both systemd (Arch) and non-systemd (Void) systems. An important feature (and vulnerability) is that the administrator may assign programs to one of two strings within the script:

- **NOPASSWD_LIST: These programs may be run without password authentication.**
- **NEVER_AUTH_LIST: These programs are prohibited entirely from running.**

**THE NEED FOR A SEPARATE POLKIT RULE FOR EACH APPLICATION IS THEREFORE ELIMINATED**. Only one new polkit action/rule pair is needed.  All other progams will be subject to default polkit/pkexec rules.  Also, If gksudo-pk polkit action/rules are not installed, NOPASSWD_LIST is again ignored, and default polkit rules for pkexec will be used for all authorized programs.

## Dependencies
**bash, sudo, zenity**

##

## Options
Only the **--user | -u** options are actually used, and as with sudo, may be omitted for "**-u root**".  All other options accepted by the original **gksudo** are looked for and stripped.  The remaining arguments are then passed to pkexec with an environment (see below)

## Applicability
gksudo-pk is designed to be fairly universal, but has not been extensively tested. Desktop environments tested so far include:
**XFCE 4.14, KDE Plasma 5, LXQT 0.14, MATE 1.24**. Both **systemd** (Arch) and **non-systemd** (Void) distributions have been tested.

## Details
The invoking user MUST be a member of **$Admin_Grp**, which defaults to **"wheel"**.  Either the group or the user **must be a sudoer** if **$Use_Sudo=true** is set in the script.  **gksudo-pk -u | --user** allows running a program as **ANY STANDARD USER, as well as root**.  However, $NOPASSWD_LIST will be ignored if the program will not be run as root, and default polkit rules will apply.  

An important key feature of gksudo-pk is the creation of a proper environment for pkexec to use.  **/etc/environment** is sourced, and **Xauthority** and **Display** variables are borrowed/provided, as well as some specific variables for KDE, if needed.  While this is a larger evironment than the basic one pkexec uses by default, it is still minimal compared to that of a regular user.  Importantly, a polkit **"org.freedesktop.policykit.exec.allow_gui"** annotation is **NOT required** for graphical programs to run.

A second important feature is the placement of the environment within one of two temporary executable scripts, "**/tmp/passwd-env**" or "**/tmp/nopasswd-env**". Creation of a polkit rule then allows different authorization protocol based on THE NAME of the script.  The administrator may then assign programs to normal, lesser, (or greater, with modifications) polkit authorization, all by changing two strings within the gksudo-pk script.

The administrator/installer is expected to check or modify four variables near the beginning of the script:
- **NOPASSWD_LIST**   (author's list left as default, change to fit your situation)
- **NEVER_AUTH_LIST**  (author's list left as default, change to fit your situation)
- **Admin_Grp="wheel"**   (only one group is allowed to run gksudo-pk. A change here must by carried over to polkit action/rule if in use)
- **Use_Sudo=true**   (Allows secure logging, and creation of XDG_RUNTIME_DIR directories, which reduce annoying warnings in Plasma5.  See notes below)

## Logging
gksudo-pk by default will create it's own log at **/var/log/gksudo-pk.log** if $Use_Sudo=true. If $Use_Sudo=false, the log will be kept in the invoking user's home directory **~/gksudo.pk.log**.  The entries are a simple record of the attempted calling of gksudo-pk, and are made whether the pkexec command actually succeeds or fails. 

## Installation
It is not difficult to install this script, but the author does not encourage "packaging" it, due to the security concerns.  To install, install sudo and zenity, then download the files. Modify the following if your polkit folders are located differently for your distribution.  Ensure that $PATH includes /usr/local/bin, unless placing the links in /bin or /usr/bin instead. From the download directory, do the following as root:

- cp 	gksudo.pk*-env.policy /usr/share/polkit-1/actions/
- cp 49-gksudo-pk-nopasswd-env.rules /etc/polkit-1/rules.d/
- cp gksudo-pk /usr/local/bin/
- chmod 0755 /usr/local/bin/gksudo-pk
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksudo  # recommended to call with "gksudo"
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksu    # optional, not recommended
 
## Notes
A common warning complains about "inability to register with accessibility bus" or similar.  This warning can be silenced by appending **NO_AT_BRIDGE=1** to **/etc/environment**. For Plasma5 and Dolphin, "XDG_RUNTIME_DIR missing" (and other) warnings abound, but usually no functional errors occur. Gksudo-pk optionally creates and assigns temporary $XDG_RUNTIME_DIR directories to start (dolphin) more quickly and reduce some warnings.

At least some marginal security is achieved by restricting write access to the log ($Use_Sudo = true), but this, and the creation of XDG runtime directories, requires sudo, and will generate sudo password prompts that are a separate authentication from polkit/pkexec.  This can be avoided by giving the group or user a NOPASSWD: setting in /etc/sudoers or /etc/sudoers.d, or by setting $Use_Sudo = false. Again, assess your risk.

The script relies heavily on creating small temporary files in the /tmp directory.  Obviously, it will run faster and be easier on drives if /tmp is a tmpfs in RAM.

