# debian-repo
create a local debian repository
## first you need to understand why would you want such a thing
I found out that maybe you are in a location that is far from universal mirrors and user may be in need of nearby source of packages so ia had to kind of find out a how to guide to create such a repository.
and being it was my first time doing , such a thing from the instructions at https://www.debian.org/mirror/ftpmirror could be a little confusing(at the time i was new to the whole IT field)
so i made a simple guide with a few steps to ease your way into it.

## first things you need to know
#### size of the repository you are trying to mirror
basically the thing you should do is find out what architectures you are going to need on your repository then check the sizes on the official https://www.debian.org/mirror/size 
after that you should allocate as much you think you are going to need and then a little more
#### partitioning
you need to create a seperate partition for better management of the repository
best thing to do is to mount the new partition at a custom directory you made for this purpose : {repository folder}

#### User
you'll need to create a new user to limit the repository syncing 
for doing that you can follow these commands in order:
  ```
  - adduser {username}
  - su {username}
  - chgrp {username} {repository folder}
  - chmod g+rwx {repository folder}
  ```

you can double check the permissions to the folder by stat command

#### web server
you can choose a webserver of your own choice to do this 
only remember to set the auto index on 
##### main steps of webserver setup:
  - `apt install nginx` (for me it was nginx)
  - edit nginx to point to the {repository folder} in the /etc/nginx/sites-enabled/default

*root {repository folder}*
  - set up domain and public ip setting
  - set up ssl certificate with certbot or other providers

## start initial mirroring
> do all the bellow instructions with the new user
#### first you should choose where to mirror from 
you can do this by visiting https://www.debian.org/mirror/list-full
> remmember to choose by geographical location and protocol + needed architectures

#### then you'll need this tool to do this right : ftpsync
download it to your {directory of choice} by following commands:
  ```
  - wget -O {directory of choice for ftpsync} https://ftp-master.debian.org/ftpsync.tar.gz
  - tar -zxf {directory of choice for ftpsync}/ftpsync.tar.gz --directory {directory of choice for ftpsync}
  ```
 
#### getting the ftpsync ready to go
step by step:
  - edit the file : /distrib/etc/ftpsync.conf by the editor of your choice
> if it doesnt exist: cp /distrib/etc/ftpsync.conf.sample /distrib/etc/ftpsync.conf
  - then edit these parameters for basic rsync mirroring:
    ```
    - MIRRORNAME="{your domain preferably}"
    - TO="absolute path to the directory set aside for repository"
    - RSYNC_PATH="debian"
    - RSYNC_HOST="{mirror Address}"
    ```
#### do an initial mirroring
 > depending on the mirror and architecture of your choice size can vary between 1-4TB
you can use this command to initiate the process:
  - `{directory of choice for ftpsync}/distrib/bin/ftpsync sync:all`
 > it is recommended to do this in the background since it can take longer than usual
  - like this : `nohup {directory of choice for ftpsync}/distrib/bin/ftpsync sync:all &`

## setup your mirror to sync periodiclly
to do this i used cronjobs
`crontab -e`

add these lines accordingly:
```
  00 03 * * * /distrib/bin/ftpsync sync:all
  00 06 * * * /distrib/bin/ftpsync sync:all
  00 12 * * * /distrib/bin/ftpsync sync:all
  00 21 * * * /distrib/bin/ftpsync sync:all
```

> you can of course change the timing of these crons

And you are Done!!!:)
