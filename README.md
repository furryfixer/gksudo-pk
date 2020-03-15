# gksudo-pk
A drop-in replacement for gksudo, with fewer options. For X11 only. PKEXEC is used to launch graphical programs. Sudo and Zenity are rquired dependencies. This bash script is NOT SECURE by modern standards. Use is never recommended with ssh or unless behind a firewall. Convenience is attained at the expense of security. Use at YOUR OWN RISK. Hopefully works in multiple desktop environments, including KDE Plasma, XFCE, MATE, LXQT. An important and unique feature (and vulnerability) is that the administrator may assign programs to one of two strings within the script:

NO_PASSWD_LIST: These programs may be run without password authentication.

NEVER_AUTH_LIST: These programs are prohibited entirely from running.

All other progams will be subject to default polkit/pkexec rules. THE NEED FOR A SEPARATE POLKIT RULE FOR EACH APPLICATION IS THEREFORE ELIMINATED. Only one polkit action/rule pair is needed.

# Dependencies
bash, sudo, zenity

# Options
Only the -u|--user options are actually used.  All other gksudo options are looked for and stripped.  The reamaining arguments are then passed to pkexec with an environment (see below)

# Details
The script MUST be invoked by the user owning the current Xsession, or it will fail (it would be unusual for this not to be the case).
gksudo-pk allows running a program as ANY STANDARD USER, as well as root.  However, $NOPASSWD_LIST will be ignored if the program will not be run as root, and default polkit rules will apply.  The script relies heavily on creating small temporary files in the /tmp directory.  Obviously, it will run faster and be easier on drives if /tmp is a tmpfs in RAM.

A first key feature of gksudo-pk is the creation of a proper environment for pkexec to use.  /etc/environment is sourced, and Xauthority and Display variables are borrowed/provided, as well as some specific variables for KDE, if needed.  While this is a larger evironment than the basic one used by pkexec by default, it is still minimal compared to that of a regular user.  Importantly, a polkit "org.freedesktop.policykit.exec.allow_gui" annotation is NOT required.

A second important feature is the placement of the environment within one of two temporary executable scripts, PASSWD-ENV or NOPASSWD-ENV. Creation of a polkit rule then allows different authorization protocol based on THE NAME of the script.  The administrator may then assign programs to normal, leseer, (or greater, with modifications) polkit akuthorization, all by changing 1 or 2 strings within the gksudo-pk script. 
