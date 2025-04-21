# Interaction of Allow Privesc false and cap sys admin

This is a repository to experiment with combinations of adding CAP_SYS_ADMIN/SYS_ADMIN to a pod alongside `allowPrivilegeEscalation: false` in relation to [this issue](https://github.com/kubernetes/kubernetes/issues/131336)


## Manifests

- `capsysadminpod.yaml` - This pod adds CAP_SYS_ADMIN to the capabilities list
- `sysadminpod.yaml` - This pod adds SYS_ADMIN to the capabilities list
- `dontallowprivesccapsysadminpod.yaml` - This has `allowPrivilegeEscalation: false` set and adds `CAP_SYS_ADMIN` to the capabilities list
- `dontallowprivescsysadminpod.yaml` - This has `allowPrivilegeEscalation: false` set and adds `SYS_ADMIN` to the capabilities list
- `invalidcap.yaml` - This pod has an invalid capability (lorem) set.

## Experiment results

All experiments completed using a kind cluster version v1.31.2.

### sysadminpod.yaml

- Creation, works fine.

**amicontained**

shows that `sys_admin` is on the capability list

```bash
Container Runtime: docker
Has Namespaces:
        pid: true
        user: false
AppArmor Profile: unconfined
Capabilities:
        BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot sys_admin mknod audit_write setfcap
Seccomp: disabled
Blocked Syscalls (13):
        MSGRCV SETSID VHANGUP ACCT SETTIMEOFDAY REBOOT INIT_MODULE DELETE_MODULE KEXEC_LOAD OPEN_BY_HANDLE_AT FINIT_MODULE KEXEC_FILE_LOAD USERFAULTFD
Looking for Docker.sock
```

**capsh**

Output of capsh on the bash process

```bash
CapInh: 0000000000000000
CapPrm: 00000000a82425fb
CapEff: 00000000a82425fb
CapBnd: 00000000a82425fb
CapAmb: 0000000000000000
```

Decoded shows CAP_SYS_ADMIN

```bash
capsh --decode=00000000a82425fb
0x00000000a82425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_admin,cap_mknod,cap_audit_write,cap_setfcap
```

### capsysadmin.yaml

- Creation works fine

**amicontained**

does not show sys_admin capability.

```bash
Container Runtime: docker
Has Namespaces:
        pid: true
        user: false
AppArmor Profile: unconfined
Capabilities:
        BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot mknod audit_write setfcap
Seccomp: disabled
Blocked Syscalls (21):
        MSGRCV SYSLOG SETSID VHANGUP PIVOT_ROOT ACCT SETTIMEOFDAY SWAPON SWAPOFF REBOOT SETHOSTNAME SETDOMAINNAME INIT_MODULE DELETE_MODULE KEXEC_LOAD PERF_EVENT_OPEN FANOTIFY_INIT OPEN_BY_HANDLE_AT FINIT_MODULE KEXEC_FILE_LOAD USERFAULTFD
Looking for Docker.sock
```

**capsh**

```bash
CapInh: 0000000000000000
CapPrm: 00000000a80425fb
CapEff: 00000000a80425fb
CapBnd: 00000000a80425fb
CapAmb: 0000000000000000
```

Decoded does not show CAP_SYS_ADMIN

```bash
capsh --decode=00000000a80425fb
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
```

### dontallowprivescsysadminpod.yaml

- Creation works fine

**amicontained**

does show the sys_admin capability

```bash
Container Runtime: docker
Has Namespaces:
        pid: true
        user: false
AppArmor Profile: unconfined
Capabilities:
        BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot sys_admin mknod audit_write setfcap
Seccomp: disabled
Blocked Syscalls (13):
        MSGRCV SETSID VHANGUP ACCT SETTIMEOFDAY REBOOT INIT_MODULE DELETE_MODULE KEXEC_LOAD OPEN_BY_HANDLE_AT FINIT_MODULE KEXEC_FILE_LOAD USERFAULTFD
Looking for Docker.sock
```

**capsh**

```bash
CapInh: 0000000000000000
CapPrm: 00000000a82425fb
CapEff: 00000000a82425fb
CapBnd: 00000000a82425fb
CapAmb: 0000000000000000
```

Decoded does show CAP_SYS_ADMIN

```bash
capsh --decode=00000000a82425fb
0x00000000a82425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_admin,cap_mknod,cap_audit_write,cap_setfcap
```

### dontallowprivesccapsysadminpod.yaml

- Creation fails with the error described in the issue

```bash
The Pod "pod-with-dont-allow-privesc-cap-sys-admin" is invalid: spec.containers[0].securityContext: Invalid value: core.SecurityContext{Capabilities:(*core.Capabilities)(0xc009611a10), Privileged:(*bool)(nil), SELinuxOptions:(*core.SELinuxOptions)(nil), WindowsOptions:(*core.WindowsSecurityContextOptions)(nil), RunAsUser:(*int64)(nil), RunAsGroup:(*int64)(nil), RunAsNonRoot:(*bool)(nil), ReadOnlyRootFilesystem:(*bool)(nil), AllowPrivilegeEscalation:(*bool)(0xc008a4fdac), ProcMount:(*core.ProcMountType)(nil), SeccompProfile:(*core.SeccompProfile)(nil), AppArmorProfile:(*core.AppArmorProfile)(nil)}: cannot set `allowPrivilegeEscalation` to false and `capabilities.Add` CAP_SYS_ADMIN
```

### invalidcap.yaml

- Creation works fine.