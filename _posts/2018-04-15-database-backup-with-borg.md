---
published: true
layout: single
title:  "Create daily database backups with Borg"
excerpt: "This article shows how to create backups of your data from PostgreSQL DB to a remote server with a secure and effective approach"
toc: true
categories: linux
tags: postgres
---


In this post, I want to describe a creating backup of PostgreSQL database and transferring it to a remote server. I will show you how to automate it to get regular daily backup and use a deduplication tool as [Borgbackup][borg-offsite] for effective using storage. Also, the tool has methods for encryption your data and I will show how to use it. 
Backup database to borg repository is the efficient and secure way to store your critically important data.

## Prerequisites

Before following this tutorial, you should understand conditions that I have:
* Has two Linux servers:
  - one is a backup remote storage
  - second is a database server

I will use Centos 7 on the remote server, and Ubuntu 16.04 on another server with installed PostgreSQL 9.6. They both have connections to one network, an IP of Centos is 192.168.0.1 and Ubuntu has 192.168.0.2. Backup server has a special backup user with name `backup_user`

## Instructions

### Install borg on both servers

Is a simple action just use apt or yum install:
```
apt install borgbackup
```

And checking your version after that `borg --version`.
```
Centos: borg 1.1.4
Ubuntu: borg 1.0.11
```

Ok, some differences can't a problem with using it. If you want to use the last version follow [this][borg-install] installing instruction. But didn't use the version before 1.0.9 because that has a [vulnerability][borg-vulnerability].

Official docs have a good description of basic usage a borg commands and options,  see them for best understanding next commands. 

### Configure access to the backup server

Generate ssh key to remote access to the backup server if you don't get it.
```
ssh-keygen -b 2048 -t rsa -q -N '' -f ~/.ssh/id_rsa
```

Create a config file if you don't want to remember an IP of the remote server like me.
```
cat << EOT >> ~/.ssh/config
Host backup_server
User backup_user
HostName 192.168.0.1
Port 22
EOT
```

And run command to copy generated public key to the remote server.
```
ssh-copy-id backup_server
```

Test it.
```
# ssh backup_server
Last login: Sun Apr 15 10:18:16 2018
[backup_user@backup_server ~]$
```

All is OK.

### Create a backup script

Ok, let's begin to automate the process of creating a backup of the database. I am using a [`pg_dumpall`][pg_dumpall] to create a backup of a full database. A result of this command a SQL-file that can create all existing databases and users in a new place. This result before coping will been stored on local path `/root/db_dumps`. Also I make backup of database configs. 

```
#!/bin/bash
BACKUP_DIR=/root/db_dumps
BACKUP_DUMP_NAME="dump_$(date '+%Y-%m-%d_%H%M%S').sql"
BACKUP_CONFIG_NAME="config_$(date '+%Y-%m-%d_%H%M%S').tar.gz"
USERNAME=postgres
BORG_CMD="borgmatic"
DB_CONF_PATH="/etc/postgresql/9.6/main"

error_exit()
{
  echo "$1" 1>&2
  exit 1
}

debug_message()
{
    echo "$1" 1>&2
}

debug_message "[backup][START] Start creating backup"

#remove previous backup
[[ -d ${BACKUP_DIR} ]] &&  rm -rf ${BACKUP_DIR}
[[ ! -d ${BACKUP_DIR} ]] && mkdir ${BACKUP_DIR} && chmod 700 ${BACKUP_DIR}

#backup database data
cd ${BACKUP_DIR}
if ! pg_dumpall -U "$USERNAME" --file=${BACKUP_DUMP_NAME}; then
  error_exit "[backup][ERROR] Failed to produce data backup"
else
  debug_message "[backup][SUCCESS] Data backup success created"
fi

#backup database configures
cd ${DB_CONF_PATH}
if ! tar -cvzf ${BACKUP_CONFIG_NAME} *.conf > /dev/null 2>&1; then
  error_exit "[backup][ERROR] Cannot create archive with PG conf files"
else
  [[ -f ${BACKUP_CONFIG_NAME} ]] &&  mv ${BACKUP_CONFIG_NAME} ${BACKUP_DIR}
  debug_message "[backup][SUCCESS] Config files success created"
fi

#copy to backup server
if ! ${BORG_CMD} ; then
  error_exit "[backup][ERROR] Failed copy to backup server"
else
  debug_message "[backup][SUCCESS] Data backup success copied"
fi


#remove local backup files
#none

debug_message "[backup][COMPLETE] Backup success created"
```

I saved this script in the database server by path `/opt/postgres_backup.sh`. And set execute rights to file (`chmod 700 /opt/postgres_backup.sh`).

#### BorgBackup command

In my script has been a variable `BORG_CMD` is a wrapper of Borgbackup - [`borgmatic`][borgmatic-offsite]. It initiates a backup, prunes any old backups according to a retention policy, and validates backups for consistency. The script supports specifying your settings in a declarative configuration file rather than having to put them all on the command-line and handles common errors.

To install `borgmatic` use pip3:
```
pip3 install --upgrade borgmatic
```

#### Borgmatic settings

The settings of borgmatic is stored on `/etc/borgmatic/config.yaml`. And my ones to our task are described below.
```
location:
    source_directories:
        - /root/db_dumps

    repositories:
        - backup_server:/backups/db

storage:
    encryption_passcommand: "cat /root/.borg-passphrase"

retention:
    keep_daily: 30
    keep_weekly: 12
    keep_monthly: 6

consistency:
    checks:
        - repository
        - archives
    check_last: 7
```

### Create a passphrase on the database server

Next command creating a key file to use as a passphrase of an encrypted repository. Borg are using system environments to get some parameters. This one show borg how to get a passphrase to either new or existing repository.
```
head -c 1024 /dev/urandom | base64 > /root/.borg-passphrase
chmod 400 /root/.borg-passphrase
export BORG_PASSCOMMAND="cat /root/.borg-passphrase"
```

### Create a repository on the backup server

Using a `repokey` modes to “passphrase-only” security. The key will be stored inside the repository (in its “config” file). In above mentioned attack scenario, the attacker will have the key (but not the passphrase). 
```
# borg init --encryption=repokey backup_server:/backups/db

By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam db_test

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.
```

The output has some warnings about a using old version.

### Create a deamon to automate running a backup script

Ubuntu has a nice mechanism to auto-running scripts or applications in the background. His name - `SystemD`. And if exists a necessity to run a script by a scheduler (i.e. every night) you should use timers that is part of systemd.

I write my own daemon  to run an above script. It has stored on `/etc/systemd/system/backup.service`.
```
[Unit]
Description=Backup PostgreSQL database

[Service]
Type=oneshot
ExecStart=/bin/bash /opt/postgres_backup.sh
```

And I has a timer unit that runs a backup.service every day at midnight
```
[Unit]
Description=Run backup script every day at midnight

[Timer]
OnCalendar=daily

[Install]
WantedBy=timers.target
```

After that you should reload daemon-service.
```
systemctl daemon-reload
```

### Manual run script and view logs

To run our systemd unit (or daemon) just run next command on a terminal:
```
systemctl start backup
```

And anfter this command completed view logs by `journalctl -u backup`
```
Apr 14 11:07:48 ubuntu systemd[1]: Starting Backup PostgreSQL database...
Apr 14 11:07:48 ubuntu bash[234]: [backup][START] Start creating backup
Apr 14 11:08:48 ubuntu bash[234]: [backup][SUCCESS] Data backup success created
Apr 14 11:08:48 ubuntu bash[234]: [backup][SUCCESS] Config files success created
Apr 14 11:10:27 ubuntu bash[234]: [backup][SUCCESS] Data backup success copied
Apr 14 11:10:27 ubuntu bash[234]: [backup][COMPLETE] Backup success created
Apr 14 11:10:27 ubuntu systemd[1]: Started Backup PostgreSQL database.
```

### Finish preparations and check the results

Don't forget setup `enable` flag on the timer for working them after reboot system.
```
systemctl enable backup.timer
systemctl start backup.timer
```

And check active system timers with `systemctl list-timers`.
```
NEXT                         LEFT     LAST                         PASSED       UNIT                         ACTIVATES
Mon 2018-04-16 00:00:00 MSK  9h left  Sun 2018-04-15 00:00:52 MSK  14h ago      backup.timer                 backup.service
...

5 timers listed.
```

Ok, I just run a backup script two times on my database and have got two backups.
```
# borg list backup_server:/backups/db
ubuntu-2018-04-15T00:05:46.344337 Sun, 2018-04-15 00:05:28
ubuntu-2018-04-15T00:08:46.344337 Sun, 2018-04-15 00:08:47
```

Let's see the information about second one.
```
# borg info backup_server:/backups/db::ubuntu-2018-04-15T00:08:46.344337
Name: ubuntu-2018-04-15T00:08:46.344337
Fingerprint: 19314fb992f83d9e746eda534d22fc8375a8d20c0cf8828a4353463574asdw
Hostname: ubuntu
Username: root
Time (start): Sun, 2018-04-15 00:08:47
Time (end):   Sun, 2018-04-15 00:08:47
Number of files: 2

                       Original size      Compressed size    Deduplicated size
This archive:                8.50 GB              8.50 GB                624 B
All archives:               17.00 GB             17.00 GB              6.64 GB

                       Unique chunks         Total chunks
Chunk index:                    2548                 6750
```

One dump file has a size of 8G, but repository using deduplication stores two ones with less disk usage space. All archives has reduce about 6,6G. Next command will agree this words
```
backup_server ># du -hd0 /backups/db
6,2G    db
```

And at the end, I want to show the command that extracts ("restore") backup from an archive repository to a local path.
```
mkdir old_data
cd old_data
borg extract backup_server:/backups/db::ubuntu-2018-04-15T00:08:46.344337
```

In conclucion I want to say that the Borg is a simple, reliable and secure tool to store your backup data. The data deduplication technique used makes Borg suitable for daily backups since only changes are stored. The authenticated encryption technique makes it suitable for backups to not fully trusted targets. And it has a simple interface to run with many parameters with borgmatic.

## Additional information:

This links might useful to get any additional information:
* [Borg by Thomas Waldmann](
http://slides.com/thomaswaldmann/borg)
* [BorgBackup Offsite](https://www.borgbackup.org/)
* [borgmatic Offsite](https://torsion.org/borgmatic/)
* [Is Borg backup suitable for the production?](https://www.reddit.com/r/linux/comments/69lm87/is_borg_backup_suitable_for_the_production/)

[borg-install]: https://borgbackup.readthedocs.io/en/stable/installation.html
[borg-vulnerability]: https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability-cve-2016-10099
[borgmatic-offsite]: https://torsion.org/borgmatic/
[borg-offsite]: https://borgbackup.readthedocs.io/en/stable/index.html
[pg_dumpall]: https://www.postgresql.org/docs/current/static/app-pg-dumpall.html
