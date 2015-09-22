To check the version of Ubuntu that is running: `lsb_release -a`

To change the editor that visudo uses: `sudo update-alternatives --config editor`

To create a new group: `sudo addgroup <groupname>`

To create a new user: `sudo adduser --ingroup <groupname> <username>`

To give a user sudo access:
* `sudo visudo`
* To require a password for each sudo command, add this line: `<username> ALL=(ALL:ALL) ALL`
* To allow sudo without a password, add this line: `<username> ALL=(ALL:ALL) NOPASSWD: ALL`
* Note: there is a tab character between the username and the rest of the line

To regenerate missing SSH host keys: `sudo ssh-keygen -A`

To see which groups a user belongs to: `groups <username>`

To add an existing user to an existing group: `sudo usermod -a -G <group> <username>`

To add support for the Google Protobuf compiler: `sudo apt-get install protobuf-compiler`
