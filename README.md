# calcardbackup

This Bash-script exports calendars and addressbooks of given users from Owncloud/Nextcloud to .ics and .vcf files and saves them compressed (.tar.gz or .zip) in a given folder.<br>
Optionally the Owncloud/Nextcloud database can be backed up as well (default: no backup of database).  
Additionally there is a config option to delete backups that are older than X days (default: no delete).

Last but not least there it is possible to run own commands at certain stages of calcardbackup. This may be useful e.g. to add files to the backup before compressing or to copy the compressed backup to another location.  
See `examples/hook.example` for details.

## Requirements

- local installation of Own-/Nextcloud >= 5.0 with MySQL/MariaDB or SQLite3 (PostgreSQL unsupported)
- Packages `grep`, `sed`, `curl` and according database packages (`mysql-server`/`mariadb-server` or `sqlite3`)
- if backups shall be zipped (instead of default gzipped tarball): package `zip`

## Installation

1. Clone the repository to your server (outside of webroot!) and enter the repo:  
`git clone https://github.com/BernieO/calcardbackup`  
`cd calcardbackup`

2. Copy example files:  
`cp examples/calcardbackup.conf.example calcardbackup.conf`  
`cp examples/users.txt.example users.txt`

3. Change path to your Own-/Nextcloud installation in config file `calcardbackup.conf` with your favorite text editor.

4. In `users.txt`, insert usernames and according passwords separated by a colon with one user per line of all users to be backed up.

5. Change ownership of repo to your webservers user (here `www-data`) and restrict access to `users.txt`:  
`chown -R www-data:www-data .`  
`chmod 600 users.txt`

6. Run script as user `www-data`:  
`sudo -u www-data ./calcardbackup`

7. Check output of script - it will tell, if anything is missing or has to be configured in `calcardbackup.conf`.

8. Find your backup in directory `backups/`.

9. **Advanced**: if you need to run own commands at certain stages of calcardbackup, have a look at `examples/hook.example`.

## Interested in more details?

Find more details here (sorry, currently only in german):  
[Blog article about calcardbackup](https://bob.gatsmas.de/articles/calcardbackup-kalender-und-adressbuchbackup-von-owncloud-nextcloud)
