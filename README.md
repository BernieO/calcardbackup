# calcardbackup

This script extracts Own-/Nextcloud calendars and addressbooks and saves them as compressed file with date-extension in a backup-folder.
As default the database is being backed up as well and stored in the same compressed file.
Additionally backups older than X days can be deleted.

## Requirements
- Own-/Nextcloud >= 5.0 with MySQL/MariaDB or SQLite3 (PostgreSQL unsupported)
- Packages sed, curl and according database packages (mysql-server/mariadb-server or sqlite3)
- if backups shall be zipped (instead of defaul gzipped tarball): package zip
- this script needs to have read-access for Own-/Nextcloud path (to read values from config.php)

## Installation

1. Clone the repository to your server and enter the repo:<br>
`git clone https://github.com/BernieO/calcardbackup`<br>
`cd calcardbackup`

2. Copy example files:<br>
`cp -a calcardbackup.conf.example calcardbackup.conf`<br>
`cp -a users.txt.example users.txt`

3. Change path to your Own-/Nextcloud installation in config file `calcardbackup.conf` with your favorite text editor.

4. Fill file `users.txt` with usernames and according passwords separated by a colon with one user per line (again with your favorite text editor).

5. change ownership of repo to your webservers user (here `www-data`) and restrict access to `users.txt`:
`chown -R www-data:www-data .`<br>`
`chmod 600 users.txt`

5. Run the script as user `www-data`:<br>
`sudo -u www-data ./calcardbackup`

6. Check output of script - it will tell, if anything else has to be configured in calcardbackup.conf
