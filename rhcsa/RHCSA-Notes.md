
---
## Logging (rsyslog)

**Configuration locations**

- `/etc/rsyslog.conf`
    
- `/etc/rsyslog.d/*.conf`
    

**Permissions**

- Only **root** can edit rsyslog configuration files.
    

**Apply changes**

```
systemctl restart rsyslog
```

---

## AutoFS (Automount)

### Master map

- `/etc/auto.master`
    
- `/etc/auto.master.d/*.autofs` 
    

**Example master entry**

```
/mnt/auto  /etc/auto.mnt
```

### Indirect maps

```
*   -rw   server:/export/&
```

- Accessed as `/mnt/auto/userA`
    
- `&` is automatically replaced with the directory name being accessed
    

### Direct maps

**Master entry**

```
/-  /etc/auto.direct
```

**Direct map**

```
/home/userA  server:/export/userA
```

---

## SELinux

**Check and change mode**

```
getenforce
setenforce 0
setenforce 1
```

**Add or change a port**

```
semanage port -a -t http_port_t -p tcp 8080
```

**Change file context**

```
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web
```

**Troubleshooting (sealert is NOT installed in the test enviroment)**

```
ausearch -m avc
journalctl -xe
```

---

## DNF / YUM Repositories

**Repository location**

- `/etc/yum.repos.d/*.repo`
    
- File **must end in `.repo`**
    

**Example repo file**

```
[christian-repo]
name=lab.example.com/Repo
baseurl=http://example.repo.com/repo
enabled=1
gpgcheck=0
```

**Enable or disable repositories**

```
dnf config-manager --set-disabled appstream
dnf config-manager --set-enabled christian-repo
```

---

## Podman Registries (User-Level)

**Configuration file**

- `~/.config/containers/registries.conf`
    

**Example**

```
unqualified-search-registries = ["registry.location.example.com"]

[[registry]]
location = "registry.location.example.com"
insecure = true
blocked = false
```

---

## LVM (High-Probability Exam Topic)

### Extend an existing volume group

```
parted /dev/vdb mklabel gpt
parted /dev/vdb mkpart primary 1MiB 2GiB
pvcreate /dev/vdb1
vgextend vgname /dev/vdb1
lvextend -r -L +1G /dev/vgname/lvname
```

- `-r` resizes the filesystem automatically **VERY IMPORTANT**
    

### Create a new logical volume

```
lvcreate -L 2G -n lvdata vgname
mkfs.xfs /dev/vgname/lvdata
```

- Add the filesystem to `/etc/fstab` for persistence
    

---

## NFS Mounts

**Example: persistently mount the /export directory from remote server to /home/docs in `/etc/fstab` 

```
/home/docs/  server:/export  nfs  rw,sync  0 0
```

---

## systemd-tmpfiles

**Configuration file**
- `/etc/tmpfiles.d/example.conf`
    

**Apply configuration**

```
systemd-tmpfiles --create /etc/tmpfiles.d/example.conf
```

---

## Firewall and SELinux Ports

**Block an IP address**

```
firewall-cmd --add-source=1.2.3.4 --zone=block --permanent
firewall-cmd --reload
```

**Add an IP address**
```
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```


**Must add SELinux port**

```
semanage port -a -t http_port_t -p tcp 8080
```

---

## “Connection Refused” Troubleshooting Order

```
systemctl status service
ss -tulpn
firewall-cmd --list-all
ausearch -m avc
journalctl -xe
```

---

## Podman and SELinux Volume Mounts

**Correct volume syntax**

```
podman run -d --name mycont -v /host/path:/container/path:Z container:latest
```

**Exam gotcha**

- Log in **directly as the user**
    
- Do **not** use `su - user` (breaks rootless podman)
    

---

## Rootless Podman with systemd (User Services)

```
mkdir -p ~/.config/systemd/user
podman generate systemd --files --name containername
systemctl --user enable --now container-containername.service
```

---

## NTP (Chrony)

**Edit**

```
/etc/chrony.conf
```

**Verify**

```
chronyc sources
```

---

## Non-Interactive Users

**Create**

```
useradd -s /sbin/nologin username
```

**Modify existing**

```
usermod -s /sbin/nologin username
```

---

## Special Permission Bits

- setuid = **4**
    
- setgid = **2**
    
- sticky = **1**
    

**Examples**

```
chmod 1777 /shared
chmod 2755 /dir
chmod 4755 /bin/example
```

## Change Root Password
- Reboot machine
- press e to edit the rescue boot entry
- navigate to line that starts with linux and add rd.break at the end
- ctrl + x 
```
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch ./autorelabel
exit
exit
reboot
```


---

## High-Priority Practice Areas

- SELinux troubleshooting without `sealert`
    
- LVM resize workflow
    
- Autofs indirect and direct maps
    
- Rootless Podman with systemd
    
- Firewalld and `semanage` ports
    
- Non-interactive users
    
- Sticky bit and setgid directories

- Change root password 
    

---