# calcardbackup

This Bash script exports calendars and addressbooks of given users from ownCloud/Nextcloud to .ics and .vcf files and saves them as compressed file. Additional options are available.  

## Requirements

- local installation of ownCloud/Nextcloud >= 5.0 with MySQL/MariaDB or SQLite3 (PostgreSQL unsupported)
- Packages `grep`, `sed`, `curl` and according database packages (`mysql-server`/`mariadb-server` or `sqlite3`)

## Quick Installation Guide

1. Clone the repository to your server (outside of webroot!) and enter the repo:  
`git clone https://github.com/BernieO/calcardbackup`  
`cd calcardbackup`

2. Copy example file `users.txt`:  
`cp examples/users.txt.example users.txt`

3. In `users.txt`, insert ownCloud/Nextcloud usernames and according passwords separated by a colon with one user per line of all users to be backed up. Be aware that other people with access to the server might be able to read the stored passwords.  

4. Change ownership of repo to your webservers user (here `www-data`) and restrict access to `users.txt`:  
`chown -R www-data:www-data .`  
`chmod 600 users.txt`

5. Run script as user `www-data` and give as first argument the path to your ownCloud/Nextcloud instance (here `/usr/share/nginx/nextcloud`  
(if you are using a self-signed certificate, you might have to use `-s` as well):  
`sudo -u www-data ./calcardbackup "/usr/share/nginx/nextcloud"`

6. Check output of script - it will tell, if it needs any other options.

7. Find your backup in directory `backups/`.

## Advanced

Several options are available for advanced use. The path to your ownCloud/Nextcloud instance is mandatory to be given as very first argument unless option `-c|--configfile` is used. Find detailed description of available options below.  
If no options at all or only `-b|--batch` is passed to the script, it attempts to use files `calcardbackup.conf` and `users.txt` in scripts directory as configuration file and login information.  

## Options
```
Usage: ./calcardbackup [URL] [option [argument]] [option [argument]] [option [argument]] ...

Arguments in big letters to options are mandatory.
Paths (FILE / DIRECTORY) are absolute paths or relative paths to working directory.

-a | --address URL
       Pass URL of Owncloud Installation to script.
       Only required for Owncloud < 7.0
-b | --batch
       Batch mode: print nothing to stdout, except for path to backup.
       Depending on configuration this will be:
         - absolute path of compressed backup file
       or if run with option -x|--uncompressed (see below)
         - absolute path of directory containing uncompressed files
-c | --configfile FILE
       Read configuration from FILE. See examples/calcardbackup.conf.example
       All other options except for -b|--batch are ignored.
-d | --date FORMAT
       Use FORMAT as extension for backup folder and compressed backup.
       FORMAT needs to be a format descriptor of command date().
       If not given default -%Y-%m-%d is used which will result in files named
       like: calcardbackup-2017-23-02.tar.gz
       Check "man date" for more info about different formats.
-e | --encrypt FILE
       Encrypt backup with AES256 (gnupg). First line of FILE will be used as passphrase
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
       Needs to be given, if certificate is selfsigned or for other reasons not trustful to curl.
-u | --usersfile FILE
       Find usernames and passwords in FILE. One user with according password separated
       by a colon per line. See examples/users.txt.example
       If this option is not given, calcardbackup will attempt to use file 'users.txt' in scripts
       directory
-x | --uncompressed
       don't compress backup folder
-z | --zip
       use zip to compress backup folder instead of creating a gzipped tarball (tar.gz)
```

## Considerations about encryption
If you want to use the included encryption possibility, be aware that:
- the files are encrypted by Gnupg, AES256 with the passphrase given in a separate file
- the passphrase is stored in a file. Other users with access to the server might be able to see the password.
- `calcardbackup` is designed to run without user interaction, so there can't be a rock solid encryption. The offered one should be sufficient in most cases though.
- if you need rock solid encryption, don't let `calcardbackup` encrypt the backup. Instead encrypt it youself.

## Interested in more details?

Find a little more background information here (sorry, only in german):  
[Blog article about calcardbackup](https://bob.gatsmas.de/articles/calcardbackup-kalender-und-adressbuchbackup-von-owncloud-nextcloud)
