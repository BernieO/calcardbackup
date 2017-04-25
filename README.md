# calcardbackup

This Bash-script exports calendars and addressbooks of given users from Owncloud/Nextcloud to .ics and .vcf files, saves them as compressed file (on demand encrypted).  
On demand the Owncloud/Nextcloud database can be backed up as well. You probably don't need that if you don't know why (default: no backup of database).  
Additionally there is a config option to delete backups that are older than X days (default: no delete).

## Requirements

- local installation of Own-/Nextcloud >= 5.0 with MySQL/MariaDB or SQLite3 (PostgreSQL unsupported)
- Packages `grep`, `sed`, `curl` and according database packages (`mysql-server`/`mariadb-server` or `sqlite3`)
- if backups shall be zipped (instead of default gzipped tarball): package `zip`

## Installation

1. Clone the repository to your server (outside of webroot!) and enter the repo:  
`git clone https://github.com/BernieO/calcardbackup`  
`cd calcardbackup`

2. Copy example file `users.txt`:  
`cp examples/users.txt.example users.txt`

3. In `users.txt`, insert usernames and according passwords separated by a colon with one user per line of all users to be backed up. Be aware that other people with access to the server might be able to read the stored passwords.  

4. Change ownership of repo to your webservers user (here `www-data`) and restrict access to `users.txt`:  
`chown -R www-data:www-data .`  
`chmod 600 users.txt`

5. Run script as user `www-data` and give as first argument the path to your ownCloud/Nextcloud instance (here `/usr/share/nginx/nextcloud`  
(if you are using a self-signed certificate, you might have to use `-s` as well):  
`sudo -u www-data ./calcardbackup "/usr/share/nginx/nextcloud"`

6. Check output of script - it will tell, if it needs any other option.

7. Find your backup in directory `backups/`.

## Details

There are two ways to run the script:

**command line mode**  
In command line mode the path to your ownCloud/Nextcloud instance is mandatory to be given as very first argument unless option `-c|--configfile` is used.  
Find detailed description of options below.  

**lazy mode**  
In lazy mode you don't pass any options or arguments. Config file `calcardbackup.conf` and file with logins `users.txt` in scripts directory will be used. See `examples` for examples. Option `-b|--batch` is the only option that may be used in lazy mode.  


###

## Options for command line mode:
```
Usage: .\calcardbackup [URL] [option [argument]] [option [argument]] [option [argument]] ...

-a | --address URL
       Pass URL of Owncloud Installation to script.
       Only required for Owncloud < 7.0
-b | --batch
       Batch mode: print nothing to stdout, except for path to backup.
       Depending on configuration this is:
         - absolute path of compressed backup file
       or if rund with option -x|--uncompressed (see below)
         - absolute path of directory containing uncompressed files
-c | --configfile FILE
       Read configuration from FILE.
       All other options except for -b|--batch are ignored.
       FILE can be absolute or relative path to working directory.
-d | --date FORMAT
       Use FORMAT as extension for backup folder and compressed backup.
       FORMAT needs to be a format descriptor of command date().
       If not given default -%Y-%m-%d is used which will result in files named
       like: calcardbackup-2017-23-02.tar.gz
       Check "man date" for more info about different formats.
-e | --encrypt FILE
       Encrypt backup with AES256 (gnupg). Passphrase must be first line of FILE
-o | --output DIRECTORY
       Use directory DIRECTORY to store backups.
       If not given, folder "backups" in working directory is created and used.
-q | --database
       Backup database as well.
       You probably don't need this, if you don't know why you should backup
       database as well.
-r | --remove N
       Remove backups older than N days from backup folder.
-s | --selfsigned
       Use, if certificate is selfsigned or for other reasons not trustful to curl.
-u | --usersfile FILE
       Find usernames and passwords in FILE. One user with according password separated
       by a colon per line. See examples/users.txt.examples
       If this option is not given, calcardbackup will look for file users.txt in scripts
       directory
-x | --uncompressed
       don't compress backup folder
-z | --zip
       use zip to compress backup folder instead of creating a gzipped tarball (tar.gz)
```

## One word about the included encryption
If you want to use the included encryption possibility, be aware that:
- the files are encrypted by Gnupg, AES256 with the passphrase given in a separate file
- the passphrase is stored in a file. Other users with access to the server might be able to see the password.
- `calcardbackup` is designed to run without user interaction, so there can't be a rock solid encryption. The offered one should be sufficiont in most cases though.
- if you need better encryption, don't let `calcardbackup` encrypt the file. Instead encrypt the backup with your own settings.

## Interested in more details?

Find more details here (sorry, currently only in german):  
[Blog article about calcardbackup](https://bob.gatsmas.de/articles/calcardbackup-kalender-und-adressbuchbackup-von-owncloud-nextcloud)
