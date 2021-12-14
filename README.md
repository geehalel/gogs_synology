# Branch `dsm-6.0`

**This branch requires DSM version 6.0.5940 or higher installed on your NAS.**

**This has only been tested on a DS213j (armv7) running DSM 6.2.4-25556 Update 2.**

- uninstall David Muller gogs package and remove root-owned permanent etc: `rm -rf /usr/syno/etc/packages/Gogs/`
- eventually remove cookies (from cadenas on site) if invalid csrf token

- **Add new user Gogs**: add a file called `conf/privilege` (see [dev guide](https://help.synology.com/developer-guide/privilege/privilege_config.html)) (6.0-5940)
  - if not specifying user/groupname, it uses Gogs:Gogs (package name)
  - $HOME is /var/packages/Gogs/target 
  - repository disappears as it is deleted when uninstalled. Use a shared folder.

```
{
  "defaults":{
    "run-as": "package"
  },
}
```

- **Create new shared folder for gogs repositories** `conf/resource` (6.0-5914 )
  - add ui to select volume: UNUSEFUL may not be specified in data-share
  - add ui to select shared folder, default name to Gogs: ok
  - create /volume1/Gogs (root:root/700) with nas control access:
  	- visible for admin logged in DSM
  	- non visible for other users logged in DSM
  	- checked permission in FileStation and Control Panel->shared edit: ok
  - test if survive `Git` update: it stops `Gogs` before updating `Git`. ok.
  - test if survive Gogs reinstallation reusing same default shared folder: ok.
```
{
    "data-share": {
        "shares": [
            {
                "name": "{{pkgwizard_foldername}}",
                "permission": {
                    "ro": ["admin"],
                    "rw": ["Gogs"]
                }
            }
        ]
    }
}
```

- **port configuration** `conf/resource` port-config points to `gogs.sc` file moved to target: ok.
- **create app.ini**: option ? Not implemented.
  - use /var/packages/Gogs/target (not /volume1/@appstore/Gogs if system install_type)
  - or display a message with relevant appp.ini settings
- **add reverse proxy for https access**: Not implemented. 
  - before first launch, in post-inst. Needs root privilege.
- **MariaDB creation**: `conf/resource` mariadb10-db. Not implemented.
- **Manage public keys for remote repo mirroring**: Not implemented.
  - put Gogs `$HOME/.ssh` in permanent storage `/var/packages/Gogs/target/etc/`
- **Manage certificates for private repo mirroring**: Not implemented.
  - otherwise remote user/passwd stored in repo/config file
  - modify Gogs internal user `.gitconfig` file
  
  ```
vi /var/packages/Gogs/target/gogs/.gitconfig
[http "https://gogs.remote"]
	sslCAPath = /etc/ssl/certs/
	sslCAInfo = /etc/ssl/certs/myHomeCA.crt
  ``` 

# gogs-synology

[Gogs](https://gogs.io) (Go Git Service) SPK package ([Synology Packages](https://www.synology.com/en-us/dsm/app_packages))

Install Gogs into a Synology NAS.


## Requirements

<sub>this package, to see Gogs requirements check https://gogs.io</sub>

* armv7 (DS213j, DS211j, Marvell Armada 370) 
* intel Atom D2700 (RS2414rp+)
* intel Atom CE5335 (DS214 play)
* MariaDB, SQLite
* Git Server

## Usage

Change **Package Center -> Trust Level** to **Any Publisher** and import manually the package from **Manual install**.
Finally, install with Gogs web installation.

## To use with another arch

Download the binary from https://gogs.io/docs/installation/install_from_binary, replace the content from **1_create_package/gogs** directory and exec create_spk.sh:

```alex@vostok:/Volumes/HD/Development/synology/gogs-spk(master)$ rm -rf 1_create_package/gogs/ && tar zxvf gogs_v0.9.13_linux_386.tar.gz -C 1_create_package/```

```$ sh create_spk.sh```


## Compiled from source

Suggested by [hirakujira](https://github.com/hirakujira)

```
GOOS=linux GOARCH=arm GOARM=7 go get -u github.com/gogits/gogs
```


## Screenshots

![Install](screenshots/install2.png)

![Install](screenshots/install.png)

![Install](screenshots/started.png)

Gogs screenshots
https://github.com/gogits/gogs


## ToDo

- Don't force to use Git Server and MariaDB (PostgreSQL? Gogs ARM version haven't Sqlite/TiDB)
- Create gogs db on MySQL on first installation
- Support to archs (and DBs)
- Don't use **root** user and create and use **gogs** user, if possible
