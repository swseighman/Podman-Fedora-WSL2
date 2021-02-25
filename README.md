## Install `podman` on Fedora 33 in WSL2

![](images/podman-windows.png)

### Credits
This tutorial is based on an article authored by Andy Wong, see the full article [here](https://oldgitops.medium.com/setting-up-podman-on-wsl2-in-windows-10-be2991c2d443).

*These steps assume you have already installed WSL.  Reference [this article](https://docs.microsoft.com/en-us/windows/wsl/install-win10) if you need WSL installation instructions.*


**Note**: Throughout this tutorial, when you see a ![red computer](images/userinput.png) icon, it indicates a command that you'll need to enter in your terminal. 

### Installation

First, let's install `podman`:

![red computer](images/userinput.png)

```
$ sudo dnf install podman
```
In order to have a properly installed `newgidmap` and `newuidmap` you should reinstall `shadow-utils`:

![red computer](images/userinput.png)

```
$ sudo dnf reinstall shadow-utils
```

### Rootless Configuration

Podman can also be used as non-root user. When `podman` runs in rootless mode, a user namespace is automatically created for the user, defined in `/etc/subuid` and `/etc/subgid`.  Containers created by a non-root user are not visible to other users and are not seen or managed by `podman` running as root.It is required to have multiple uids/gids set for an user. Make certain the user is present in the files `/etc/subuid` and `/etc/subgid`. 

![red computer](images/userinput.png)

```
$ cat /etc/subuid<your-username>:100000:65536
$ cat /etc/subgid<your-username>:100000:65536
```

Execute the following commands to add the necessary ranges to the files:

![red computer](images/userinput.png)

```
$ sudo usermod --add-subuids 10000-75535 <your-username>$ sudo usermod --add-subgids 10000-75535 <your-username>
```
Or just add the content manually.```$ echo <your-username>:10000:65536 >> /etc/subuid$ echo <your-username>:10000:65536 >> /etc/subgid
```

You'll now notice a second entry in the `/etc/subuid` and `/etc/subgids` files:

![red computer](images/userinput.png)

```
$ cat /cat /etc/subgid<your-username>:100000:65536<your-username>:10000:65536$ cat /cat /etc/subuid<your-username>:100000:65536<your-username>:10000:65536
```

### Configuration Tasks

Without `systemd` (currently not *easily* supported in WSL2), the `$XDG_RUNTIME_DIR` was not available for `podman` to use for temporary files.  So, in your `~/.bashrc`, add the following section:

![red computer](images/userinput.png)

```
if [[ -z "$XDG_RUNTIME_DIR" ]]; then  export XDG_RUNTIME_DIR=/run/user/$UID  if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then    export XDG_RUNTIME_DIR=/tmp/$USER-runtime    if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then      mkdir -m 0700 "$XDG_RUNTIME_DIR"    fi  fifi
```

![red computer](images/userinput.png)

```
$ source ~/.bashrc
```


We'll also need to update the `/etc/containers/containers.conf` file. If the file `containers.conf` does not exist, you can download a version [here](https://github.com/containers/common/blob/master/pkg/config/containers.conf).

![red computer](images/userinput.png)

```
$ sudo vi /etc/containers/containers.conf
```

In the `[engine]` section, change *(and/or uncomment)*:

`cgroup_manager = "systemd"`to:**`cgroup_manager = "cgroupfs"`**And change:
`events_logger = "journald"`to:
**`events_logger = "file`**

To run run containers on privileged ports (like port 80), we'll need to edit `/etc/sysctl.conf` and add an entry.

![red computer](images/userinput.png)

```
$ sudo vim /etc/sysctl.conf
```

Add the following line:
```
net.ipv4.ip_unprivileged_port_start=0
```

Apply the change:

![red computer](images/userinput.png)
```$ sudo sysctl -p
```

In order to access your containers, you'll need to identify the IP address of your WSL2 instance, so execute the following command:

![red computer](images/userinput.png)

```
$ ip addr | grep 172
      inet 172.28.143.196/20 brd 172.28.143.255 scope global eth0
```

### Testing our Installation

Next, let's test our `podman` installation:

![red computer](images/userinput.png)

```
$ podman run -it -p 80:80 nginx
```

To test, browse to: **http://172.28.143.196** *(from ip addr command above)*


### Learn more

To display `podman` related system information, execute:

![red computer](images/userinput.png)

```
$ podman info
```

You can learn more about `podman` [here](http://docs.podman.io/en/latest/).