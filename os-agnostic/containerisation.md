# Containerisation



## Docker

If you have code-execution in a host with docker installed, and have the appropriate permissions to use it, then you can escalate your permissions fairly trivially.  The process via the docker group is explained in the [Linux Privilege Escalation](https://booj.gitbook.io/virgil/linux/privilege-escalation#docker) section.

### docker.sock

### --privileged

```text
/sbin/capsh --print
```

If you see `CAP_SYS_ADMIN`, then congratulations, you won.

