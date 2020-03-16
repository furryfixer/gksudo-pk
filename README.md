# gksudo-pk
A drop-in replacement for **gksudo**, with fewer options. For X11 only. **Pkexec** is used to launch graphical programs. **Sudo** and **Zenity** are rquired dependencies. This bash script is NOT SECURE by modern standards. Use is never recommended with ssh or unless behind a firewall. Convenience is attained at the expense of security. **Use at YOUR OWN RISK**. Hopefully works in multiple desktop environments, including KDE Plasma, XFCE, MATE, LXQT. An important feature (and vulnerability) is that the administrator may assign programs to one of two strings within the script:

- **NO_PASSWD_LIST: These programs may be run without password authentication.**
- **NEVER_AUTH_LIST: These programs are prohibited entirely from running.**

All other progams will be subject to default polkit/pkexec rules. **THE NEED FOR A SEPARATE POLKIT RULE FOR EACH APPLICATION IS THEREFORE ELIMINATED**. Only one polkit action/rule pair is needed.

## Dependencies
**bash, sudo, zenity**

## Options
Only the **--user | -u** options are actually used.  All other options accepted by the original **gksudo** are looked for and stripped.  The reamaining arguments are then passed to pkexec with an environment (see below)

## Details
The script MUST be invoked by the user owning the current Xsession (it would be unusual for this not to be the case), or it will fail with X-related errors. The invoking user also MUST be a member of $ADMIN_GRP, which defaults to **"wheel"**.
**gksudo-pk -u | --user** allows running a program as **ANY STANDARD USER, as well as root**.  However, $NOPASSWD_LIST will be ignored if the program will not be run as root, and default polkit rules will apply.  The script relies heavily on creating small temporary files in the /tmp directory.  Obviously, it will run faster and be easier on drives if /tmp is a tmpfs in RAM.

A first key feature of gksudo-pk is the creation of a proper environment for pkexec to use.  **/etc/environment** is sourced, and **Xauthority** and **Display** variables are borrowed/provided, as well as some specific variables for KDE, if needed.  While this is a larger evironment than the basic one used by pkexec by default, it is still minimal compared to that of a regular user.  Importantly, a polkit **"org.freedesktop.policykit.exec.allow_gui" annotation is NOT required.**

A second important feature is the placement of the environment within one of two temporary executable scripts, "**/tmp/passwd-env**" or "**/tmp/nopasswd-env**". Creation of a polkit rule then allows different authorization protocol based on THE NAME of the script.  The administrator may then assign programs to normal, lesser, (or greater, with modifications) polkit akuthorization, all by changing 1 or 2 strings within the gksudo-pk script.
The invoking user may (at least initially) be asked for a SUDO password, unless the individual or group is set NOPASSWD: by sudo.  This is separate authentication from policykit/pkexec, and required to chown temporary XDG_RUNTIME_DIR directories, as well as writing log entries.

## Applicability
gksudo-pk is designed to be fairly universal, but has not been extensively tested. Desktop environments tested so far include:
XFCE 4.14, KDE Plasma 5, LXQT 0.14, MATE 1.24. Both systemd (Arch) and non-systemd (Void) distributions have been tested.

## Logging
gksudo-pk by default will create it's own log at /var/log/gksudo-pk.log. This may be turned off by setting LOGGING="false". The entries are a simple record of the attempted calling of gksudo-pk, and are made whether the pkexec command actually succeeds or fails.

## Installation
It is not difficult to install this script, but there are no plans to "package" it.  As convenient as gksudo-pk may be, it is a security risk, so some manual work is needed to discourage the unwary! To install, install sudo and zenity, then clone or download the files. Modify the following if your polkit folders are located differently for your distribution.  Ensure that $PATH includes /usr/local/bin, unless placing the links in /bin or /usr/bin instead. Fron the download directory, do the following as root:

- cp 	gksudo.pk*-env.policy /usr/share/polkit-1/actions/
- cp 49-gksudo-pk-nopasswd-env.rules /etc/polkit-1/rules.d/
- cp gksudo-pk /usr/local/bin/
- chmod 0755 /usr/local/bin/gksudo-pk
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksudo  # recommended to call with "gksudo"
- ln -s /usr/local/bin/gksudo-pk /usr/local/bin/gksu    # optional, not recommended
 

