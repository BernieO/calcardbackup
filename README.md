# calcardbackup

This script extracts Own-/Nextcloud calendars and addressbooks and saves them as compressed file with date-extension in a backup-folder.
As default the database is being backed up as well and stored in the same compressed file.
Additionally backups older than X days (see configuration below) can be deleted.

Requirements:
- Own-/Nextcloud >= 5.0 with MySQL/MariaDB or SQLite3 (PostgreSQL unsupported)
- Package curl and according database packages (mysql-server/mariadb-server or sqlite3)
- if backups shall be zipped (instead of defaul gzipped tarball): package zip
- this script needs to have read-access for Own-/Nextcloud path (to read values from config.php)
- file 'calcardbackup.config' in this scripts directory (see calcardbackup.config.example)
- currently (to be changed in the future, see below): file 'users.txt' in this scripts directory with usernames and according passwords separated by ":" with one user per line (see users.txt.example). Only data of mentioned users will be extracted.
Example:
```
Thomas:PA55W0RD
Emma:9876543
```
- IMPORTANT INFO about users.txt: userlist will soon be moved to config file thus making users.txt obsolete.
