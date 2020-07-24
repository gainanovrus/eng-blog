---
published: true
layout: single
title: DELL. File Management on S4048 switch (OS9-series system)
excerpt: >-
  Some useful notes for working with files (save, copy and transfer) on Dell Networking system.
categories: sysad
tags: dell networks
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
---

This article I will continue speaking about DELL PowerSwitch [S4048-ON][s4048-specs].
Recently have been showed a many helpful commands for working in console.
There is I want to say more about file system and files management on this switch series.

## Local and remote file system

The Dell Networking system can use the internal Flash, external Flash, or remote devices to store files.
Rename, delete, and copy files on the system from EXEC Privilege mode.
The system stores files on the **internal Flash** by default but can be configured to store files elsewhere.

### show file-systems

To view file system information, use the following command - .
```
DellEMC# show file-systems

      Size(b)     Free(b)      Feature      Type   Flags  Prefixes
   2368282624  2304512000        FAT32 USERFLASH      rw  flash:
            -           -  unformatted USERFLASH      rw  fcmfs:
    218103808    64708608      Unknown  NFSMOUNT      rw  nfsmount:
            -           -            -   network      rw  ftp:
            -           -            -   network      rw  tftp:
            -           -            -   network      rw  scp:
            -           -            -   network      rw  http:
            -           -            -   network      rw  https:
```

## Operations with local Filesystem

### dir

Display the files in a file system. The default is the current directory.

```
DellEMC# dir

Directory of flash:

  1  drwx       4096   Jan 01 1980 00:00:00 +00:00 .  
  2  drwx       3580   Jul 18 2020 22:09:12 +00:00 ..  
  3  drwx       4096   Dec 10 2019 14:49:54 +00:00 TRACE_LOG_DIR  
  4  drwx       4096   Dec 10 2019 14:49:54 +00:00 CONFD_LOG_DIR  
  5  drwx       4096   Dec 10 2019 14:49:54 +00:00 CORE_DUMP_DIR  
  6  d---       4096   Dec 10 2019 14:49:54 +00:00 ADMIN_DIR  
  7  drwx       4096   Dec 10 2019 14:49:54 +00:00 RUNTIME_PATCH_DIR  
  8  -rwx          1   Jul 03 2020 22:54:24 +00:00 boots.txt  
  9  drwx       4096   Jul 14 2020 15:28:26 +00:00 CONFIG_TEMPLATE  
 10  -rwx          0   Jul 14 2020 15:28:56 +00:00 pdtrc.lo0  
 11  -rwx       4276   Jul 14 2020 15:29:04 +00:00 last-cold-st-config  
 12  -rwx       4435   Jul 03 2020 23:45:50 +00:00 startup-config  
 13  -rwx     314038   Jul 03 2020 23:45:54 +00:00 confd_cdb.tar.gz  
 14  -rwx         56   Jul 03 2020 23:45:54 +00:00 confd_cdb.tar.gz.version  
 15  -rwx          0   Jul 14 2020 15:28:26 +00:00 earlyCliParserDbg  

flash: 2368282624 bytes total (2304438272 bytes free)
```

### mkdir/rmdir

To create a directory on file system uses `mkdir`, to remove - `rmdir`
```
DellEMC# mkdir tmp_dir

DellEMC# dir tmp_dir
Directory of flash:/tmp_dir

  1  drwx       4096   Jul 18 2020 22:35:30 +00:00 .  
  2  drwx       4096   Jan 01 1980 00:00:00 +00:00 ..  

DellEMC# rmdir tmp_dir
Proceed to remove the directory [confirm yes/no]: y   

DellEMC# dir tmp_dir  
% Error: The specified file or directory does not exist.
```

### cd/pwd

To change the default directory, use the following command.
```
cd directory
```

Display the current working directory.
```
pwd
```

### copy

Copy one file to another location.
Dell EMC Networking OS supports IPv4 and IPv6 addressing for FTP, HTTP, TFTP, and SCP (in the hostip field).

Syntax: `copy source-file-url destination-file-url`

Example:
```
DellEMC# copy startup-config startup-config.2
!
4435 bytes successfully copied
```

### rename

Rename a file in the local file system.
```
DellEMC# rename startup-config.2 startup-config.3
```

### show file

To show a content of text file use `show file` command. It will print a file into stdout of console. Example:
```
DellEMC# show file startup-config.3

! Version 9.13(0.1)
! Last configuration change at Fri Jul  3 23:43:25 2020 by default
! Startup-config last updated at Fri Jul  3 23:45:50 2020 by default
!
boot system stack-unit 1 primary system: A:
boot system stack....
```

### delete

Delete a file from the flash. After deletion, files cannot be restored.

Syntax: `delete [flash://]filepath [no-confirm]`
* `no-confirm` - does not require user input for each file prior to deletion.

Example:
```
DellEMC# delete flash://startup-config.2
Proceed to delete startup-config.2 [confirm yes/no]: y

DellEMC# delete startup-config.3 no-confirm
```

### save

The `save` command save the output of `show` command into file on flash memory.
The `save` entry must always be the last option.

For example:
```
DellEMC# show command-history | save flash://command-history.txt
Start saving show command report .......

DellEMC# dir command-history.txt
Directory of flash:
  1  -rwx      76096   Jul 18 2020 22:30:16 +00:00 command-history.txt
```



## Copy file from/to remote network device

Files can be copied from many places. You should input the URL to desired place.
* `ftp`: ftp://userid:password@hostip/filepath
* `http`: http://hostip/filepath
* `nfsmount`: nfsmount://<mount-point>/filepath
* `scp`: scp://userid:password@hostip/filepath
* `tftp`: tftp://hostip/filepath

When copying a file to a remote location (for example, `scp`), enter only the keywords and DellOS prompts you need for the rest of the information. For example, you can enter `copy running-config scp:`

To copy a file to NFS server the remote directory need to be mounted. Use a command `mount nfs`. For example, mount nfs
```
DellEMC# mount nfs 192.168.101.2:/dell_shared /dell_shared
```

Examples:
1. Copy from remote system to `flash:` by SCP
```
DellEMC# copy scp://admin:some_strong_password@192.168.101.2/firmware.bin  flash://firmware.bin
!
65059 bytes successfully copied
```
2. Copy back (from local to remote)
```
DellEMC# copy running-config scp://admin:some_strong_password@192.168.101.2/
!
4324 bytes successfully copied
```

### verify

Validate a file on the flash drive after it has been transferred to the system.

Syntax: `verify {md5|sha256} file [hash-value]`
Parameters:
* `md5` - to use the MD5 message-digest algorithm.
* `sha256` - to use the SHA256 Secure Hash Algorithm
* `file` - a filename that will verified
* `hash-value` - the hash that has gotten on the remote system

Example:
```
DellEMC# verify md5 firmware.bin 4659f3b278b52fb09c7ed759b2f10474
MD5 hash VERIFIED for firmware.bin
```

## Conclusion

As you can see above many of Dell commands seems on Unix shell commands.
In this post I would like to save in the "memory" my experience in file management Operations.
I test it on the Dell Networking OS9 switch, so it helps you easy find a desired command in one page.
This quick and small manual to work with files, firmware and cofigs without handling big official papers.
Besides to know more about this commands see the links in the next section.

## Additional information

* [How to manage files (Copy and Delete) on Dell Networking Force10 switches](https://www.dell.com/support/article/en-us/how10841/how-to-manage-files-copy-and-delete-on-dell-networking-force10-switches?lang=en) - some short note about managing files on Dell site
* [PowerSwitch S4048-ON Documentation](https://www.dell.com/support/home/en-us/product-support/product/force10-s4048-on/docs) - a page that consists all documentation for the 10G switch of Dell company
* [DELL EMC NETWORKING S4048-ON SWITCH](https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf) - Dell spec sheet for S4048–ON switch
* [Dell Command Line Reference Guide for the S4048–ON System](https://topics-cdn.dell.com/pdf/force10-s4048-on_administrator-guide7_en-us.pdf) - Dell man page included full command list
* [Dell Configuration Guide for the S4048–ON System](https://topics-cdn.dell.com/pdf/force10-s4048-on_administrator-guide8_en-us.pdf) - almost two thousands pages about configuration of Dell switch


[s4048-specs]: https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf
