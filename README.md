# gksudo-pk
a drop-in replacement for gksudo, with fewer options. For X11 only. PKEXEC is used to launch graphical programs. Sudo and Zenity are rquired dependencies. This bash script is NOT SECURE by modern standards. Use is never recommended with ssh or unless behind a firewall. Convenience is attained at the expense of security. Use at YOUR OWN RISK. Hopefully works in multiple desktop environments, including KDE Plasma, XFCE, MATE, LXQT.

A polkit "org.freedesktop.policykit.exec.allow_gui" annotation is NOT required, as a temporary environment will be provided to run graphically. User must be a member of an $ADMIN_GRP, ("wheel" by default) to invoke it. An important and unique feature (and vulnerability) is that the administrator may assign programs to one of two strings within the script:

NO_PASSWD_LIST: These programs may be run without password authentication.

NEVER_AUTH_LIST: These programs are prohibited entirely from running.

All other progams will be subject to default polkit/pkexec rules. THE NEED FOR A SEPARATE POLKIT RULE FOR EACH APPLICATION IS THEREFORE ELIMINATED. Only one polkit action/rule pair is needed.
