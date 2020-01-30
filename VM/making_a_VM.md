# Making a boot2root machine/VM

## Misc

* Remove a user from sudoer
    - `sudo deluser USERNAME sudo`
* Send message to another user
    - `mail -s "Your message" <username>`
    - EX: `mail -s "hey" www-data`
    - You might need to install mailutils to use `mail` command
* Restrict user to their own home directory
    - `sudo chown -R user:user userdir/`
    - `sudo chmod 0750 userdir/`
* If you want some kind of cronjob setup run
    - `crontab -e`
    - [crontab.guru](https://crontab.guru/)

* To make your own man page

```bash
$ cp nuseradd /usr/local/man/man8/nuseradd.8
$ gzip /usr/local/man/man8/nuseradd.8
$ man nuseradd
```

* Portknocking
    - https://blog.rapid7.com/2017/10/04/how-to-secure-ssh-server-using-port-knocking-on-ubuntu-linux/
    - http://technical-qa.blogspot.com/2014/10/solution-to-knockd-wont-work-open-port.html

* Use Fixed memory allocation since that is much faster
* Assign 1 GB RAM first and then test the VM if it's slow or something then extend the RAM .
    - This helps to test VM on low specs. We might have 8/16GB ram but the person doing might not have that.
* For ubuntu always use the `server` version and not the GUI/full ISO
    - http://old-releases.ubuntu.com/releases/
* For debian

## Supervisor - Systemd

### It's better to use normal systemd rather then putting your head under this supervisor

* For supervisor when you want to autostart some service on boot

https://gist.github.com/mozillazg/6cbdcccbf46fe96a4edd

```
[program:name]
directory=/opt/1337
command=flask run --port 1337
autostart=true
autorestart=true
stopsignal=INT
stopasgroup=true
killasgroup=true
```

Then restart the `supervisor` service
```bash
sudo systemctl restart supervisor.service
```

And then you can check if the service is running by executing
```
supervisorctl status
```

You should see the new app.

Sometime we end up getting error like
```
unix:\\\var\run\supervisor.sock no such file
```

or

```
error: <class socket.sock>..........
```

So the fix that seemed to work for me was to run `echo_supervisord_conf > /etc/supervisor/supervisord.conf `

and then reread the config with

```
supervisorctl -c /etc/supervisord/supervisord.conf reread
```

and then we should see all the services running.

* Instead of using supervisord prefer going directly for systemd.
    - https://blog.miguelgrinberg.com/post/running-a-flask-application-as-a-service-with-systemd

## Xinetd

* If you want to run a script with which a person can interact then use `xinted`
    - https://www.cyberciti.biz/faq/linux-how-do-i-configure-xinetd-service/
    - Sometimes when you start the xinetd service you might get error about `no service game/tcp` etc if this is the case just open `/etc/services` and add your service name with the port you are running it on.
    ```
    game        1337/tcp        #this is a game
    ```

## Python script to binary

* You can use `cython` or `nuitka` to compile python script to an elf.
    - nuitka does everything for you.
    - If using `cython` then do the following:
        * `cython file.py --embed`
            - use `-3` or `-2` to use python versions
        * `gcc -Os -I /usr/include/python3.5m -o file file.c -lpython3.5m -lpthread -lm -lutil -ldl`

## Fixing interface name

* When we need to fix the damn interface name so dynamic dhcp is working
    - https://mzfr.github.io/interface-names

## Add a new user:

```bash
sudo adduser <username>
```

## Setting up FTP server

* Install vsftpd
    - `sudo apt install vsftpd`
* make a backup copy of the config file.
    - `cp /etc/vsftpd.conf /etc/vsftpd.conf.orig`
* Edit the options accordingly

* https://www.tecmint.com/install-ftp-server-in-ubuntu/
* https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-anonymous-downloads-on-ubuntu-16-04

## Edit /etc/issue

To be able to display the IP of the machine right when it starts you can edit /etc/issue

```
IP: \4{eth0}
```


This only display the IP and if you want something else you can add that too like the name of the machine or something else.


## Setup Wordpress

The best thing is to follow this article

https://www.tecmint.com/install-wordpress-on-ubuntu-16-04-with-lamp/

Make sure to verify which is the latest version for PHP and wordpress.

## Setting up flask application with SYSTEMD

- https://blog.miguelgrinberg.com/post/running-a-flask-application-as-a-service-with-systemd

Basically make a file named `whatevernameyouwant.service` in `/etc/systemd/system` and write this:

```
[Unit]
Description=web application
After=network.target

[Service]
User=www-data
WorkingDirectory=/opt/webapp
ExecStart=/bin/bash -c "/usr/local/bin/flask run --host 0.0.0.0 --port 80 "
Restart=always

[Install]
WantedBy=multi-user.target
```
