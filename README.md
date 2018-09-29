# calcardbackup

This Bash script exports calendars and addressbooks from ownCloud/Nextcloud to .ics and .vcf files and saves them as a compressed file. Additional options are available.  

__IMPORTANT__: starting with version 0.8.0 there is no need anymore for a file with user credentials because *calcardbackup* creates backups by fetching the relevant data directly from the database. All users upgrading calcardbackup from a previous version to version 0.8.0 are advised to delete the file with users credentials - or at least remove the cleartext passwords from this file!  

## Requirements

- local installation of ownCloud/Nextcloud >= 5.0 with MySQL/MariaDB, PostgreSQL or SQLite3
- the user running the script needs to be able to read the full path to ownClouds/Nextclouds `config.php`, all used configuration files and to the script itself
- *optional*: package `zip` to compress backup as zip-file  (instead of tar.gz)
- *optional*: package `curl` when using option `-g` (which is not recommended!) to get calendars/addressbooks via http(s)

## Quick Installation Guide

1. Clone the repository to your server (outside of webroot!) and enter the repo:  
`cd /usr/local/bin`   
`git clone https://github.com/BernieO/calcardbackup`  
`cd calcardbackup`

2. Change the ownership of repo to your webserver's user (here `www-data`):  
`chown -R www-data:www-data .`  

3. Run the script as user `www-data` and give as first argument the path to your ownCloud/Nextcloud instance (here `/var/www/nextcloud`):  
`sudo -u www-data ./calcardbackup "/var/www/nextcloud"`

4. Check output of script - it will tell, if it needs any other options.

5. Find your backup in directory `backups/`.

There are many more options available: have a look at sections "Options" and "Usage examples" below.

## Advanced

All options can be specified in a configuration file or as command line arguments. If started with no options at all or only `-b|--batch`, the script attempts to use file `calcardbackup.conf` in the script's directory as configuration file.  
If no configuration file via option `-c|--config` is given, the path to your ownCloud/Nextcloud instance must be the very first argument. Find detailed description of all available options below.

## Options

```
Usage: ./calcardbackup [DIRECTORY] [option [argument]] [option [argument]] [option [argument]] ...

Arguments in capital letters to options are mandatory.
Paths (FILE / DIRECTORY) are absolute paths or relative paths to working directory.

-a | --address URL
       Pass URL of ownCloud Installation to script.
       Only required when using option '-g|--get-via-http' for ownCloud < 7.0
-b | --batch
       Batch mode: print nothing to stdout, except for path to backup.
       Depending on configuration this will be:
         - absolute path of compressed backup file
       or if run with option '-x|--uncompressed' (see below)
         - absolute path of directory containing uncompressed files
-c | --configfile FILE
       Read configuration from FILE. See 'examples/calcardbackup.conf.example'
       All other options except for '-b|--batch' will be ignored!
-d | --date FORMAT
       Use FORMAT as file name extension for backup directory or compressed backup file.
       FORMAT needs to be a format descriptor of the command date().
       The default is -%Y-%m-%d and will result in a directory or file
       named: 'calcardbackup-2017-03-23' or 'calcardbackup-2017-03-23.tar.gz'
       Check 'man date' for more info about different formats and syntax.
-e | --encrypt FILE
       Encrypt backup file with AES256 (gnupg). First line of FILE will be used as passphrase
-f | --fetch-from-database
       Create calendar/addressbook backups by fetching the data directly from the database
       instead of downloading the according files from the ownCloud/Nextcloud webinterface.
       It is not required to give this option as this is the default for calcardbackup >= 0.8.0
-g | --get-via-http
       NOTE: this option is deprecated. It is only available due to backwards compatibility.
       Get calendar/addressbooks via http request from the ownCloud/Nextcloud server. This used to
       be the default behaviour until calcardbackup <= 0.7.2, but is not recommended anymore.
       When using this option, a file with usernames and according cleartext passwords (see option
       '-u|--users-file') is mandatory.
-h | --help
       Print version number and a short help text 
-i | --include-shares
       Backup shared addressbooks/calendars as well. Items will only be backed up once: e.g. a shared
       calendar won't be backed up, if the same calendar was already backed up for another user.
       NOTE: this option will be ignored, if not used together with option '-u|--users-file'.
-na | --no-addressbooks
       Do not backup addressbooks
-nc | --no-calendars
       Do not backup calendars
-o | --output DIRECTORY
       Use directory DIRECTORY to store backups.
       If this option is not given, folder 'backups/' in script's directory is created and used.
-p | --snap
       Use this option, if you are running nextcloud-snap (https://github.com/nextcloud/nextcloud-snap).
       calcardbackup will then use the in the snap package included cli utility 'nextcloud.mysql-client'
       to read the needed values from the database. Note that in order for this to work, calcardbackup has
       to be run with sudo (even running as root without sudo will fail!).
-r | --remove N
       Remove backups older than N days from backup folder (N needs to be a positive integer).
-s | --selfsigned
       Needs to be given if certificate is selfsigned or for any other reason not trustful to curl.
       This option might only be needed when using option '-g|--get-via-http'
-u | --usersfile FILE
       Give location of FILE, which contains users to be backed up. One user per line.
       See 'examples/users.txt.example'
       Due to option '-f|--fetch-from-database' being the default for calcardbackup >= 0.8.0
       there is no need anymore to give passwords in this file or to use this file at all.
       NOTE: this file is mandatory when run with option '-g|--get-via-http'
-x | --uncompressed
       Do not compress backup folder
-z | --zip
       Use zip to compress backup folder instead of creating a gzipped tarball (tar.gz)
```

## About option -f / -\-fetch-from-database (default)

__NOTE__: startin with version 0.8.0 this option is the default and does not have to be passed to the script.  
*calcardbackup* was traditionally backing up addressbooks and calendars by downloading the according files from the ownCloud/Nextcloud webinterface. However, with large addressbooks, this could lead to timeouts resulting in *calcardbackup* not being able to backup large addressbooks. Also there was a file with user credentials needed.  
With version 0.8.0 *calcardbackup* creates calendar and addressbook backups as default by fetching the according data directly from the database which speeds up the backup process for addressbooks massively resulting in much less server load.  
Additionaly the file with user credentials is not needed anymore: all available calendars/addressbooks from the database will be backed up (option `-i|--include-shares` will be ignored).  
If only calendars/addressbooks of certain users shall be backed up, `users.txt` may still be used, but there is no need anymore to give passwords in this file.  

## nextcloud-snap users

If you are running Nextcloud-snap (https://github.com/nextcloud/nextcloud-snap), you have to use option `-p|--snap` to tell calcardbackup to use the cli utility `nextcloud.mysql-client` from the snap package.  
In order for this to work, calcardbackup has to be run with `sudo` (even running as root without `sudo` will fail). As path to Nextcloud use the path to the configuration files of nextcloud. In a standard installation this would be `/var/snap/nextcloud/current/nextcloud`. See example no.6 below.

## Usage examples

1. `./calcardbackup /var/www/nextcloud -nc -x`  
Do not backup calendars (`-nc`) and store backed up files uncompressed (`-x`) in folder named `calcardbackup-YYYY-MM-DD` (default) under ./backups/ (default).

2. `./calcardbackup /var/www/nextcloud --no-calendars --uncompressed`  
This is exactly the same command like in the first example but with using long options instead of short options.

3. `./calcardbackup -c /etc/calcardbackup.conf`  
Use configuration-file /etc/calcardbackup.conf (`-c /etc/calcardbackup.conf`). Parameters for desired behaviour have to be given in that file (see examples/calcardbackup.conf.example).  
It doesn't make any sense to give more options on command line in this case, because they will be ignored (except for `-b|--batch`).  

4. `./calcardbackup /var/www/nextcloud -b -d .%d.%H -z -e /home/tom/key -o /media/data/backupfolder/ -u /etc/calcardbackupusers -i -r 15`  
Supress output except for path to the backup (`-b`), use extension .DD.HH (`-d .%d.%H`), zip backup (`-z`), encrypt the zipped backup with using the first line in file /home/tom/key as encryption-key (`-e /home/tom/key`), save backup in folder /media/data/backupfolder/ (`-o /media/data/backupfolder/`), only backup items of usernames given in file /etc/calcardbackupusers (`-u /etc/calcardbackupusers`), include shared items (`-i`) and delete all backups older than 15 days (`-r 15`)  

5. `./calcardbackup`  
Use file calcardbackup.conf in the script's directory as configuration file. This is basically the same as example no.3, but with a different location of the configuration file.

6. `sudo ./calcardbackup /var/snap/nextcloud/current/nextcloud -p`  
This example is for nextcloud-snap users. calcardbackup will use the cli utility from nextcloud-snap to access the database (`-p`) and backup all calendars/addressbooks found in the database.  

7. `./calcardbackup /var/www/nextcloud -g -u /etc/calcardbackupusers -s -i`  
Use the deprecated method and get the addressbook/calendar files via https-request from the ownCloud/Nextcloud webinterface (`-g`, deprecated), find usernames and according cleartext passwords of users to be backed up in file /etc/calcardbackupusers (`-u calcardbackupusers`, mandatory with option -g), tell calcardbackup, that the server is using a selfsigned certificate (`-s`, only needed with option -g) and include shared items (`-i`). The Backup will be saved as compressed `*-tar.gz` file in folder named `calcardbackup-YYYY-MM-DD` (default) under `./backups/` (default).  
__NOTE__: using option `-g` is deprecated and not recommended anymore, due to the mandatory file with user credentials and other drawbacks!  

## Considerations about encryption

If you want to use the included encryption possibility, be aware that:
- the files are encrypted by GnuPG, AES256 with the passphrase given in a separate file
- the passphrase is stored in a file. Other users with access to the server might be able to see the passphrase.
- `calcardbackup` is designed to run without user interaction, so there can't be a rock solid encryption. I consider the offered one as sufficient in most cases though.
- if you need rock solid encryption, don't let `calcardbackup` encrypt the backup. Instead, encrypt it yourself.
- command to decrypt (you will be prompted to enter the passphrase):  
`gpg -o OUTPUT_FILE -d FILE_TO_DECRYPT.GPG`

## Want to read some of that in german?

There is a little article about this also in german available:  
[Blog article about calcardbackup](https://bob.gatsmas.de/articles/calcardbackup-kalender-und-adressbuchbackup-von-owncloud-nextcloud)

## About option -i / -\-include-shares

__NOTE:__ You don't have to read this unless you want to run calcardbackup with option `-g|--get-via-http`, which is not recommended.
Due to the new default behaviour of calcardbackup >= 0.8.0 (creating calendars/addressbooks by fetching data from the database), this option is basically not needed anymore.  
If for whatever reason calcardbackup is being run with option `-g|--get-via-http` (not recommended!), this option may be used as follows to keep passwords of the main users secret:
- create a new user in your ownCloud/Nextcloud
- share addressbooks/calendars (to be backed up) with that new user
- in `users.txt` give only the username and password of this new user
- use calcardbackup with option `-i` to back up shared items as well

__Benefit of this approach:__ if the file `users.txt` gets in wrong hands, only this new user account is being compromised.  
__Drawback:__ no automatic inclusion of newly created addressbooks/calendars. Items will not be backed up unless being shared with that new user account.
