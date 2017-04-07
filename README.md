# calcardbackup

This script extracts Own-/Nextcloud calendars and addressbooks and saves them as compressed file with a date-extension in a backup-folder.<br>
Optional the Own-/Nextcloud-database can be backed up as well and stored in the same compressed file (default: no backup of database).<br>
Additionally there is an config option to delete backups that are older than X days (default: no delete).

## Requirements

- local installation of Own-/Nextcloud >= 5.0 with MySQL/MariaDB or SQLite3 (PostgreSQL unsupported)
- Packages `sed` `curl` and according database packages (`mysql-server`/`mariadb-server` or `sqlite3`)
- if backups shall be zipped (instead of default gzipped tarball): package `zip`

## Installation

1. Clone the repository to your server and enter the repo:<br>
`git clone https://github.com/BernieO/calcardbackup`<br>
`cd calcardbackup`

2. Copy example files:<br>
`cp calcardbackup.conf.example calcardbackup.conf`<br>
`cp users.txt.example users.txt`

3. Change path to your Own-/Nextcloud installation in config file `calcardbackup.conf` with your favorite text editor.

4. Fill file `users.txt` with usernames and according passwords separated by a colon with one user per line (again with your favorite text editor).

5. Change ownership of repo to your webservers user (here `www-data`) and restrict access to `users.txt`:<br>
`chown -R www-data:www-data .`<br>
`chmod 600 users.txt`

6. Run script as user `www-data`:<br>
`sudo -u www-data ./calcardbackup`

7. Check output of script - it will tell, if anything is missing or has to be configured in `calcardbackup.conf`

8. Find your backup in directory `backups/`

## Interested in more details?

Find more details here (sorry, currently only in german):
[Blog article about calcardbackup](https://bob.gatsmas.de/articles/kalender-und-adressbuchbackup-von-own-nextcloud-calcardbackup)
