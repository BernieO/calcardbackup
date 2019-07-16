# calcardbackup

[:de: auf deutsch lesen...](README_GER.md)

This Bash script exports calendars and addressbooks from ownCloud/Nextcloud to .ics and .vcf files and saves them to a compressed file. Additional options are available.

:warning: Starting with version 0.8.0, there is no need anymore for a file with user credentials because all data is fetched directly from the database.  
If only calendars/addressbooks of certain users shall be backed up, list them in `users.txt` without any passwords.

:bangbang: __All users upgrading *calcardbackup* from a previous version to version 0.8.0 or above are strongly advised to delete the file with users credentials - or at least to remove the cleartext passwords from this file!__

## Contents
- [Requirements](#requirements)
- [Quick Installation Guide](#quick-installation-guide)
- [Options](#options)
- [Usage Examples](#usage-examples)
- [Nextcloud-Snap Users](#nextcloud-snap-users)
- [Considerations about Encryption](#considerations-about-encryption)
- [Does this also work with a broken instance?](#does-this-also-work-with-a-broken-owncloudnextcloud-instance)
- [About Option -g|--get-via-http](#about-option--g----get-via-http)
- [About Option -i|--include-shares](#about-option--i----include-shares)
- [Links](#links)

## Requirements

- local installation of ownCloud/Nextcloud >= 5.0 with MySQL/MariaDB, PostgreSQL or SQLite3
- command line client appropriate for database type
- the user running the script needs to be able to read the full path to ownClouds/Nextclouds `config.php`, to the script itself and all used configuration files
- GNU Bash >= 4.2 (check with `bash --version`)
- *optional*: package `gnupg` to encrypt backup
- *optional*: package `zip` to compress backup as zip-file  (instead of tar.gz)
- *optional*: package `curl` when using deprecated option `-g|--get-via-http`

## Quick Installation Guide

1. Clone the repository to your server (outside of webroot!) and enter the repo:  
`cd /usr/local/bin`   
`git clone https://github.com/BernieO/calcardbackup`  
`cd calcardbackup`

2. Change the ownership of repo to your webserver's user (here `www-data`):  
`chown -R www-data:www-data .`  

3. Run the script as your webserver's user (here `www-data`) and give as first argument the path to your ownCloud/Nextcloud instance (here `/var/www/nextcloud`):  
`sudo -u www-data ./calcardbackup "/var/www/nextcloud"`

4. Check output of script - it will tell, if it needs any other options.

5. Find your backup in directory `backups/`.

There are many more options available: have a look at sections [Options](#options) and [Usage examples](#usage-examples).

## Options

All options can be specified in a configuration file or as command line arguments. If started with no options at all or only `-b|--batch`, the script attempts to use file `calcardbackup.conf` in the script's directory as configuration file.  
If no configuration file via option `-c|--configfile` is given, the path to your ownCloud/Nextcloud instance must be the very first argument.  
Find detailed description of all available options below.

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
       or, if run with option '-x|--uncompressed' (see below),
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
-g | --get-via-http
       NOTE: this option is deprecated and might be removed in a future version of calcardbackup;
       it is only available to provide backwards compatibility.
       Get calendar/addressbooks via http request from the ownCloud/Nextcloud server.
       When using this option, a file with usernames and according cleartext passwords (see option
       '-u|--usersfile') is mandatory.
       This used to be the default behaviour until calcardbackup <= 0.7.2, but is not recommended
       anymore due to the security issue of cleartext passwords in a separate file.
-h | --help
       Print version number and a short help text 
-i | --include-shares
       Backup shared addressbooks/calendars, too. Items will only be backed up once: e.g. a shared
       calendar won't be backed up if the same calendar was already backed up for another user.
       NOTE: this option will be ignored if not used together with option '-u|--usersfile'.
-ltm | --like-time-machine N
       keep all backups for the last N days, keep only backups created on mondays for the time before.
-na | --no-addressbooks
       Do not backup addressbooks
-nc | --no-calendars
       Do not backup calendars
-o | --output DIRECTORY
       Use directory DIRECTORY to store backups.
       If this option is not given, folder 'backups/' in script's directory is created and used.
-p | --snap
       This option is mandatory if you are running nextcloud-snap
       (https://github.com/nextcloud/nextcloud-snap). With this option, calcardbackup has to be
       run with sudo (even running as root without sudo will fail!).
-r | --remove N
       Remove backups older than N days from backup folder (N needs to be a positive integer).
-s | --selfsigned
       Let cURL ignore an untrustful (e.g. selfsigned) certificate. With option -g, this is
       mandatory for backup in that case. In any case, cURL is used to retrieve status.php
       of the ownCloud/Nextcloud installation to perform some additional checks. If cURL
       can't access the URL due to an untrustful certificate, calcardbackup will run without
       executing these checks.
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

NOTE:  Option '-f|--fetch-from-database' (introduced with calcardbackup 0.6.0) is set as
       default for calcardbackup >= 0.8.0, thus it has no function anymore.
```

## Usage Examples

1. `./calcardbackup /var/www/nextcloud -nc -x`  
Do not backup calendars (`-nc`) and store backed up files uncompressed (`-x`) in folder named `calcardbackup-YYYY-MM-DD` (default) under ./backups/ (default).

2. `./calcardbackup /var/www/nextcloud --no-calendars --uncompressed`  
This is exactly the same command as above but with long options instead of short options.

3. `./calcardbackup -c /etc/calcardbackup.conf`  
Use configuration file /etc/calcardbackup.conf (`-c /etc/calcardbackup.conf`). Parameters for desired behaviour have to be given in that file (see [examples/calcardbackup.conf.example](examples/calcardbackup.conf.example)).  
Don't give any other command line options in this case, because they will be ignored (except for `-b|--batch`).

4. `./calcardbackup`  
Use file calcardbackup.conf in the script's directory as configuration file.  
This is basically the same as example no.3, but with the default location of the configuration file.

5. `./calcardbackup /var/www/nextcloud -b -d .%d.%H -z -e /home/tom/key -o /media/data/backupfolder/ -u /etc/calcardbackupusers -i -r 15`  
Suppress output except for path to the backup (`-b`), use file name extension .DD.HH (`-d .%d.%H`), zip backup (`-z`), encrypt the zipped backup with using the first line in file /home/tom/key as encryption-key (`-e /home/tom/key`), save backup in folder /media/data/backupfolder/ (`-o /media/data/backupfolder/`), only back up items of usernames given in file /etc/calcardbackupusers (`-u /etc/calcardbackupusers`), include users' shared address books/calendars (`-i`) and delete all backups older than 15 days (`-r 15`).

6. `sudo ./calcardbackup /var/snap/nextcloud/current/nextcloud -p`  
This example is for [nextcloud-snap](https://github.com/nextcloud/nextcloud-snap) users. *calcardbackup* will use the cli utility from nextcloud-snap to access the database (`-p`) and backup all calendars/addressbooks found in the database.

7. `./calcardbackup /var/www/nextcloud -ltm 30 -r 180`  
Keep all backups for the last 30 days, but keep only backups created on mondays for the time before (`-ltm 30`) and remove all backups older than 180 days (`-r 180`).  
:warning: Make sure backups are also created on mondays when using option `-ltm`

8. `./calcardbackup /var/www/nextcloud -g -u /etc/calcardbackupusers -s -i`  
Use the deprecated method and get the addressbook/calendar files via https-request from the ownCloud/Nextcloud webinterface (`-g`, deprecated), find usernames and according cleartext passwords of users to be backed up in file /etc/calcardbackupusers (`-u calcardbackupusers`, mandatory with option -g), tell *calcardbackup* that the server is using a selfsigned certificate (`-s`, only needed with option -g) and include shared items (`-i`). The backup will be saved as compressed `*.tar.gz` file named `calcardbackup-YYYY-MM-DD.tar.gz` (default) in folder `./backups/` (default).  
:warning: Using option `-g` is deprecated and not recommended anymore, due to the mandatory file with user credentials and other drawbacks! It might be removed in a future version of *calcardbackup* (see [About Option -g|--get-via-http](#about-option--g----get-via-http)).

## Nextcloud-snap Users

If you are running [Nextcloud-Snap](https://github.com/nextcloud/nextcloud-snap), you have to use option `-p|--snap` to tell *calcardbackup* to use the cli utility `nextcloud.mysql-client` from the snap package.  
In order for this to work, *calcardbackup* has to be run with `sudo` (even running as root without `sudo` will fail).  
As path to Nextcloud use the path to the configuration files of nextcloud. In a standard installation this would be `/var/snap/nextcloud/current/nextcloud`. See [usage example no.6](#usage-examples).

## Considerations about Encryption

If you want to use the included encryption possibility, be aware that:
- the files are encrypted by [GnuPG](https://en.wikipedia.org/wiki/GNU_Privacy_Guard), [AES256](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) with the passphrase given in a separate file
- the passphrase is stored in a file. Other users with access to the server might be able to see the passphrase.
- *calcardbackup* is designed to run without user interaction, so there can't be a rock solid encryption. I consider the offered one as sufficient in most cases though.
- if you need rock solid encryption, don't let *calcardbackup* encrypt the backup. Instead, encrypt it yourself.
- command to decrypt (you will be prompted to enter the passphrase):  
`gpg -o OUTPUT_FILE -d FILE_TO_DECRYPT.GPG`

## Does this also work with a broken ownCloud/Nextcloud instance?

__Yes, it does!__

*calcardbackup* only needs the database (and access to it) from an ownCloud/Nextcloud installation to be able to extract calendars/addressbooks from the database and save them as .ics and .vcf files.  
Here is how this can be accomplished:

1. create a dummy Nextcloud directory including subdirectory `config`:  
`mkdir -p /usr/local/bin/nextcloud_dummy/config`

2. create and edit file `config.php` to fit your needs as follows:  
`nano /usr/local/bin/nextcloud_dummy/config/config.php`

   - add database type according to [config.sample.php](https://github.com/nextcloud/server/blob/v14.0.3/config/config.sample.php#L90-L101)

   - for MySQL/MariaDB/PostgreSQL:
     - add according database values according to [config.sample.php](https://github.com/nextcloud/server/blob/v14.0.3/config/config.sample.php#L103-L135)

   - for SQLite3:
     - add path to the nextcloud_dummy folder as 'datadirectory' according to [config.sample.php](https://github.com/nextcloud/server/blob/v14.0.3/config/config.sample.php#L76-L82)
     - copy the SQLite3 database to the nexcloud_dummy directory (filename of the SQLite3 database must be `owncloud.db`):  
     `cp /path/to/owncloud.db /usr/local/bin/nextcloud_dummy/owncloud.db`

   - if the database belongs to an installation of ownCloud <= 8.2, the following line needs to be added:  
     `'version' => '8.0.0',`

3. run *calcardbackup* and give as first argument the path to dummy Nextcloud directory created in step 1:  
`./calcardbackup /usr/local/bin/nextcloud_dummy`

## About Option -g | -\-get-via-http

:warning: This option is deprecated and not recommended anymore due to the necessity to give cleartext passwords in a separate file! It might be removed in a future version of *calcardbackup*.

As its default, *calcardbackup* creates calendar and addressbook backups by fetching the according data directly from the database. However, if invoked with option `-g|--get-via-http`, *calcardbackup* is using the legacy method of backing up addressbooks and calendars by downloading the according files from the ownCloud/Nextcloud webinterface. Thus, a file with usernames and passwords is necessary, passed to the script via option `-u|--usersfile`.

Using this option also carries the risk of timeouts resulting in *calcardbackup* not being able to back up large addressbooks.

__To make a long story short__: all you need to know about option `-g|--get-via-http` is to __NOT__ use it, unless you have a good reason to expose passwords of the users to be backed up.

## About Option -i | -\-include-shares

:warning: There is no need to read this section unless you want to run *calcardbackup* with the deprecated option `-g|--get-via-http`, which is not recommended (see [About Option -g|--get-via-http](#about-option--g----get-via-http)).  

If, for whatever reason, *calcardbackup* is being run with option `-g|--get-via-http` (not recommended!), this method may be used as follows to keep passwords of the main users secret:
- create a new user in your ownCloud/Nextcloud
- share addressbooks/calendars (to be backed up) with that new user
- in `users.txt` give only the username and password of this new user
- add option `-i` to back up shared items as well

__Benefit of this approach:__ if the file `users.txt` gets in wrong hands, only this new user account is being compromised.  
__Drawback:__ no automatic inclusion of newly created addressbooks/calendars. Items will not be backed up unless being shared with that new user account.  

Due to the new default behaviour of *calcardbackup* >= 0.8.0 (creating calendars/addressbooks by fetching data from the database), this option basically becomes pointless. If you still want to use this method of backing up shared items even without option `-g`, then there is of course no need to give passwords in the usernames file. If no usernames file is passed to the script via option `-u`, option `-i` will be ignored (because all existing calendars/addressbooks will be backed up anyway).

## Links

#### Related Forum Threads
- [help.nextcloud.com](https://help.nextcloud.com/t/calcardbackup-bash-script-to-backup-nextcloud-calendars-and-addressbooks-as-ics-vcf-files/11978) - Nextcloud
- [central.owncloud.org](https://central.owncloud.org/t/calcardbackup-bash-script-to-backup-owncloud-calendars-and-addressbooks-as-ics-vcf-files/7340) - ownCloud

#### Blog articles about *calcardbackup*
- [bob.gatsmas.de](https://bob.gatsmas.de/articles/calcardbackup-jetzt-erst-recht) - October 2018 (german)
- [strobelstefan.org](https://strobelstefan.org/?p=6094) - January 2019 (german)
- [newtoypia.blogspot.com](https://newtoypia.blogspot.com/2019/04/nextcloud.html) - April 2019 (taiwanese)

#### Docker Image for *calcardbackup*
- [hub.docker.com](https://hub.docker.com/r/waja/calcardbackup) - Docker Image by waja

#### ICS and VCF standard
- [RFC 5545](https://tools.ietf.org/html/rfc5545) - Internet Calendaring and Scheduling Core Object Specification (iCalendar)
- [RFC 6350](https://tools.ietf.org/html/rfc6350) - vCard Format Specification

#### Exporter Plugins from SabreDAV used by ownCloud/Nextcloud
- [ICSExportPlugin.php](https://github.com/sabre-io/dav/blob/master/lib/CalDAV/ICSExportPlugin.php) - ICS Exporter `public function mergeObjects`
- [VCFExportPlugin.php](https://github.com/sabre-io/dav/blob/master/lib/CardDAV/VCFExportPlugin.php) - VCF Exporter `public function generateVCF`
