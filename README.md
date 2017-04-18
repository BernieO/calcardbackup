# calcardbackup

This Bash-script exports calendars and addressbooks of given users from Owncloud/Nextcloud to .ics and .vcf files, saves them compressed and optional encrypted in a given folder.  
On demand the Owncloud/Nextcloud database can be backed up as well (default: no backup of database).  
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

3. Change path to your Own-/Nextcloud installation in config file `calcardbackup.conf` and adjust the other options to fit your needs.

4. In `users.txt`, insert usernames and according passwords separated by a colon with one user per line of all users to be backed up. Be aware that other people with access to the server might be able to read the stored passwords.  

5. Change ownership of repo to your webservers user (here `www-data`) and restrict access to `users.txt` and `calcardbackup.conf`:  
`chown -R www-data:www-data .`  
`chmod 600 users.txt calcardbackup.conf`

6. Run script as user `www-data`:  
`sudo -u www-data ./calcardbackup`

7. Check output of script - it will tell, if anything is missing or has to be configured in `calcardbackup.conf`.

8. Find your backup in directory `backups/`.

9. **Advanced**: if you need to run own commands at certain stages of calcardbackup, have a look at `examples/hook.example`.

10. If you want to use the included encryption possibility, be aware that:
- the files are encrypted by Gnupg, AES256 with the passphrase given in `calcardbackup.conf`
- the passphrase is stored in a file. Other users with access to the server might be able to see the password.
- `calcardbackup` is designed to run without user interaction, so there can't be a rock solid encryption. The offered one should be sufficiont in most cases though.
- if you need better encryption, don't let `calcardbackup` encrypt the file. Instead use `hook prefinish` to encrypt the backup with your own settings.

## Interested in more details?

Find more details here (sorry, currently only in german):  
[Blog article about calcardbackup](https://bob.gatsmas.de/articles/calcardbackup-kalender-und-adressbuchbackup-von-owncloud-nextcloud)
