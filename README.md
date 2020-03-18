# gksudo-pk
A drop-in replacement for **gksudo**, with fewer options. For X11 only. **Pkexec** is used to launch graphical programs as root, or as another user. **Sudo** and **Zenity** are required dependencies. This bash script is **NOT SECURE** by modern standards. Use is not recommended with ssh or unless behind a firewall. Convenience is attained at the expense of security. **Use at YOUR OWN RISK**. Tested and hopefully works in multiple desktop environments, including KDE Plasma, XFCE, MATE, LXQT. Works in many systemd (Arch) and non-systemd (Void) systems. An important feature (and vulnerability) is that the administrator may assign programs to one of two strings within the script:

- **NO_PASSWD_LIST: These programs may be run without password authentication.**
- **NEVER_AUTH_LIST: These programs are prohibited entirely from running.**

All other progams will be subject to default polkit/pkexec rules. **THE NEED FOR A SEPARATE POLKIT RULE FOR EACH APPLICATION IS THEREFORE ELIMINATED**. Only one polkit action/rule pair is needed.

## Dependencies
**bash, sudo, zenity**

## Options
Only the **--user | -u** options are actually used.  All other options accepted by the original **gksudo** are looked for and stripped.  The reamaining arguments are then passed to pkexec with an environment (see below)

## Details
The script is best invoked by the user owning the current Xsession (it would be unusual for this not to be the case), or X-related errors may ensue. The invoking user also MUST be a member of $ADMIN_GRP, which defaults to **"wheel"**.  Either the group or the user **must be a sudoer**.  **gksudo-pk -u | --user** allows running a program as **ANY STANDARD USER, as well as root**.  However, $NOPASSWD_LIST will be ignored if the program will not be run as root, and default polkit rules will apply.  The script relies heavily on creating small temporary files in the /tmp directory.  Obviously, it will run faster and be easier on drives if /tmp is a tmpfs in RAM.

A first key feature of gksudo-pk is the creation of a proper environment for pkexec to use.  **/etc/environment** is sourced, and **Xauthority** and **Display** variables are borrowed/provided, as well as some specific variables for KDE, if needed.  While this is a larger evironment than the basic one pkexec uses by default, it is still minimal compared to that of a regular user.  Importantly, a polkit **"org.freedesktop.policykit.exec.allow_gui" annotation is NOT required.**

A second important feature is the placement of the environment within one of two temporary executable scripts, "**/tmp/passwd-env**" or "**/tmp/nopasswd-env**". Creation of a polkit rule then allows different authorization protocol based on THE NAME of the script.  The administrator may then assign programs to normal, lesser, (or greater, with modifications) polkit akuthorization, all by changing 1 or 2 strings within the gksudo-pk script.
The invoking user may (at least initially) be asked for a SUDO password, unless the individual or group is set NOPASSWD: by sudo.  This is separate authentication from policykit/pkexec, and required to chown temporary XDG_RUNTIME_DIR directories, as well as writing log entries.

## Applicability
gksudo-pk is designed to be fairly universal, but has not been extensively tested. Desktop environments tested so far include:
XFCE 4.14, KDE Plasma 5, LXQT 0.14, MATE 1.24. Both systemd (Arch) and non-systemd (Void) distributions have been tested.

## Logging
gksudo-pk by default will create it's own log at **/var/log/gksudo-pk.log**. This may be turned off by setting LOGGING="false". The entries are a simple record of the attempted calling of gksudo-pk, and are made whether the pkexec command actually succeeds or fails. Only root may write to the log, so sudo authentication will be required.

## Installation
It is not difficult to install this script, but there are no plans to "package" it.  As convenient as gksudo-pk may be, it is a security risk, so some manual work is needed to discourage the unwary! To install, install sudo and zenity, then clone or download the files. Modify the following if your polkit folders are located differently for your distribution.  Ensure that $PATH includes /usr/local/bin, unless placing the links in /bin or /usr/bin instead. Fron the download directory, do the following as root:

- cp 	gksudo.pk*-env.policy /usr/share/polkit-1/actions/
- cp 49-gksudo-pk-nopasswd-env.rules /etc/polkit-1/rules.d/
- cp gksudo-pk /usr/local/bin/
- chmod 0755 /usr/local/bin/gksudo-pk
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksudo  # recommended to call with "gksudo"
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksu    # optional, not recommended
 
## Notes
A common warning in complains about "inability to register with accessibility bus" or similar.  This warning can be silenced by appending **NO_AT_BRIDGE=1** to **/etc/environment**. For Plasma5 and Dolphin, "XDG_RUNTIME_DIR missing" (and other) warnings abound, but usually no functional errors occur. Gksudo-pk creates and assigns temporary $XDG_RUNTIME_DIR directories to start (dolphin) more quickly and reduce some warnings.

At least some marginal security is achieved by restricting write access to the log, but this, and the creation of XDG runtime directories, requires sudo, and some password prompts that this script otherwise could be configured to avoid.  This is not an issue for $ADMIN_GRP users (wheel by default), if the group or individual has a NOPASSWD: setting in /etc/sudoers or /etc/sudoers.d.  This avoids double authentication dialogs.  Again, assess your risk.
