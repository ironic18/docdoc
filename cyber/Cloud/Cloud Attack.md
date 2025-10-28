
# CLOUD *** Attack!

jueves, 26 de diciembre de 2024 15:26

Owasp a un top 10 pour ce genre d'attaque.
Le sujet se rapproche de prêt du sujet CI/CD (continuous Integration, Continuous Deployment, à mettre en rapport avec DevOps). Notions à creuser et comprendre: "builder" (jenkins?) et "agent". Bien comprendre qu'il y a deux manières d'accéder aux ressources (ex: storage s3) du cloud: directement via ce qui est accessible, ou via l'API. Si l'accès direct est parfois impossible ça peut être mal configuré pour l'API via une policy globale (ex: Allow any authenticated user).

If there is a large amount of sensitive information that could be valuable, we may attempt to exfiltrate data by copying it to another AWS S3 bucket rather than directly downloading it.

Using the AWS S3 cp command allows faster transfers between buckets and gives us more time to access the data later without drawing immediate attention. Always monitor unusual S3 bucket activities and apply strict access control policies

Pour chercher des leaks dans Git:
```
$\Gamma$ (root@kali)-[ /static_content/static_content]
$\sim$ gitleaks detect
0
1
10
00
$\square$ gitleaks
6:21AM INF 7 commits scanned.
6:21AM INF scan completed in 62.2 ms
6:21AM INF no leaks found
```

## Containers:

Both ifconfig and ip are missing on the host. It's starting to seem like we are in a container, since we are limited by what we can run. Container enumeration is very similar to standard Linux enumeration. However, there are some additional things we should search for. For example, we should check the container for mounts that might contain secrets. We can list the mounts by reviewing the contents of /proc/mounts.

Or : cat/proc/1/status | grep Cap

jenkins@fcd3cc360d9e: 〜 cat/proc/1/status | grep Cap

cat/proc/1/status | grep Cap

CapInh: 000000000000000

CapPrm: 0000003fffffffff

CapEff: 0000003fffffffff

CapBnd: 0000003fffffffff

CapAmb: 0000000000000000

The values in CapPrm, CapEff, and CapBnd represent the list of capabilities. However, they're
currently encoded, so we'll have to decode them into something more useful. We can do this using Kali's capsh utility.

kali@kali: "$ capsh --decode $=0000003fffffffff

0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_ kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadc ast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_ sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resou rce,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_ setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_au dit_read

The presence of cap_net_admin and cap_sys_admin indicates that this container is either running in a privileged context or, at the very least, with all capabilities added.

Based on the output, we find that our shell is in a Docker container. This changes our enumeration tactics a bit. To start, checking environment variables has increased in priority. Environment variables are often used to configure applications with secrets that we usually don't want in source code. We'll run printenv to list all the environment variables.

There are multiple methods that we can use to discover the permission boundaries of our current account. The easiest method would be to use the account to list its own information and policies, but this only works if the user has permissions to list its access. Another option is to brute force all API calls and log the successful ones. However, this option is very noisy, and we should avoid it when we can.

From the output, we find that the username is jenkins-admin. Next, let's discover what permissions our account has. There are three ways an administrator may attach a policy to a user:
_Inline Policy: Policy made only for a single user account and attached directly.
_Managed Policy Attached to User: Customer- or AWS-managed policy attached to one or more users.
_Group Attached Policy: Inline or Managed Policy attached to a group, which is assigned to the user.

# 25.7. Dependency Chain Abuse 

Dependency Chain Abuse happens when a malicious actor tricks the build system into downloading harmful code by hijacking or mimicking dependencies. Insufficient Pipeline-Based Access Controls occur when pipelines have excessive permissions, risking system compromise. Permissions should be tightly scoped to prevent this. Insecure System Configuration refers to vulnerabilities due to misconfigurations or insecure code, while Improper Artifact Integrity Validation allows attackers to push malicious code into a pipeline without proper checks. These OWASP risks often overlap and act as general guidelines.
