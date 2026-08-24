

---
title: Linux containers in 500 lines of code
author: lizzie
source: https://blog.lizzie.io/linux-containers-in-500-loc.html
tags:
  - linux
  - containers
  - security
  - c-programming
  - namespaces
  - cgroups
created: 2026-08-20
---

# Linux containers in 500 lines of code

> [!info] Подписка
> Like this writing? Subscribe to receive updates on vulnerabilities and software projects as soon as I publish them!

## Table of Contents
- [Container setup](#container-setup)
- [`contained.c`](#containedc)
  - [Namespaces](#namespaces)
  - [Capabilities](#capabilities)
    - [Dropped capabilities](#dropped-capabilities)
    - [Retained Capabilities](#retained-capabilities)
  - [Mounts](#mounts)
  - [System Calls](#system-calls)
    - [Disallowed System Calls](#disallowed-system-calls)
    - [Allowed System Calls](#allowed-system-calls)
  - [Resources](#resources)
  - [Networking](#networking)
- [Footnotes & References](#footnotes--references)

---

I've used Linux containers directly and indirectly for years, but I wanted to become more familiar with them. So I wrote some code. This used to be 500 lines of code, I swear, but I've revised it some since publishing; I've ended up with about 70 lines more.

I wanted specifically to find a minimal set of restrictions to run untrusted code. This isn't how you should approach containers on anything with any exposure: you should restrict everything you can. But I think it's important to know which permissions are categorically unsafe! I've tried to back up things I'm saying with links to code or people I trust, but I'd love to know if I missed anything.

This is a `noweb`-style piece of literate code. References named `<<x>>` will be expanded to the code block named `x`. This document and this code are licensed under the GPLv3.

## Container setup

There are several complementary and overlapping mechanisms that make up modern Linux containers. Roughly,

- `namespaces` are used to group kernel objects into different sets that can be accessed by specific process trees. For example, pid namespaces limit the view of the process list to the processes within the namespace. There are a couple of different kind of namespaces. I'll go into this more later.
- `capabilities` are used here to set some coarse limits on what uid 0 can do.
- `cgroups` is a mechanism to limit usage of resources like memory, disk io, and cpu-time.
- `setrlimit` is another mechanism for limiting resource usage. It's older than cgroups, but can do some things cgroups can't.

These are all Linux kernel mechanisms. Seccomp, capabilities, and `setrlimit` are all done with system calls. `cgroups` is accessed through a filesystem.

There's a lot here, and the scope of each mechanism is pretty unclear. They overlap a lot and it's tricky to find the best way to limit things. User namespaces are somewhat new, and promise to unify a lot of this behavior. But unfortunately compiling the kernel with user namespaces enabled complicates things. Compiling with user namespaces changes the semantics of capabilities system-wide, which could cause more problems or at least confusion[^1]. There have been a large number of privilege-escalation bugs exposed by user namespaces. "Understanding and Hardening Linux Containers" explains:

> Despite the large upsides the user namespace provides in terms of security, due to the sensitive nature of the user namespace, somewhat conflicting security models and large amount of new code, several serious vulnerabilities have been discovered and new vulnerabilities have unfortunately continued to be discovered. These deal with both the implementation of user namespaces itself or allow the illegitimate or unintended use of the user namespace to perform a privilege escalation. Often these issues present themselves on systems where containers are not being used, and where the kernel version is recent enough to support user namespaces.

It's turned off by default in Linux at the time of this writing[^2], but many distributions apply patches to turn it on in a limited way[^3].

But all of these issues apply to hosts with user namespaces compiled in; it doesn't really matter whether we use user namespaces or not, especially since I'll be preventing nested user namespaces. So I'll only use a user namespace if they're available.

*(The user-namespace handling in this code was originally pretty broken. Jann Horn in particular gave great feedback. Thanks!)*

## `contained.c`

This program can be used like this, to run `/misc/img/bin/sh` in `/misc/img` as `root`:

```bash
[lizzie@empress l-c-i-500-l]$ sudo ./contained -m ~/misc/busybox-img/ -u 0 -c /bin/sh
=> validating Linux version...4.7.10.201610222037-1-grsec on x86_64.
=> setting cgroups...memory...cpu...pids...blkio...done.
=> setting rlimit...done.
=> remounting everything with MS_PRIVATE...remounted.
=> making a temp directory and a bind mount there...done.
=> pivoting root...done.
=> unmounting /oldroot.oQ5jOY...done.
=> trying a user namespace...writing /proc/32627/uid_map...writing /proc/32627/gid_map...done.
=> switching to uid 0 / gid 0...done.
=> dropping capabilities...bounding...inheritable...done.
=> filtering syscalls...done.
/ # whoami
root
/ # hostname
05fe5c-three-of-pentacles
/ # exit
=> cleaning cgroups...done.
```

So, a skeleton for it:

**Listing 7: `contained.c`**

```cpp
/* -*- compile-command: "gcc -Wall -Werror -lcap -lseccomp contained.c -o contained" -*- */
/* This code is licensed under the GPLv3. You can find its text here:
   https://www.gnu.org/licenses/gpl-3.0.en.html */

#define _GNU_SOURCE
#include <errno.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <sched.h>
#include <seccomp.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <sys/capability.h>
#include <sys/mount.h>
#include <sys/prctl.h>
#include <sys/resource.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <sys/syscall.h>
#include <sys/utsname.h>
#include <sys/wait.h>
#include <linux/capability.h>
#include <linux/limits.h>

struct child_config {
	int argc;
	uid_t uid;
	int fd;
	char *hostname;
	char **argv;
	char *mount_dir;
};

<<capabilities>>
<<mounts>>
<<syscalls>>
<<resources>>
<<child>>
<<choose-hostname>>

int main (int argc, char **argv)
{
	struct child_config config = {0};
	int err = 0;
	int option = 0;
	int sockets[2] = {0};
	pid_t child_pid = 0;
	int last_optind = 0;
	while ((option = getopt(argc, argv, "c:m:u:"))) {
		switch (option) {
		case 'c':
			config.argc = argc - last_optind - 1;
			config.argv = &argv[argc - config.argc];
			goto finish_options;
		case 'm':
			config.mount_dir = optarg;
			break;
		case 'u':
			if (sscanf(optarg, "%d", &config.uid) != 1) {
				fprintf(stderr, "badly-formatted uid: %s\n", optarg);
				goto usage;
			}
			break;
		default:
			goto usage;
		}
		last_optind = optind;
	}
finish_options:
	if (!config.argc) goto usage;
	if (!config.mount_dir) goto usage;

<<check-linux-version>>

	char hostname[256] = {0};
	if (choose_hostname(hostname, sizeof(hostname)))
		goto error;
	config.hostname = hostname;

<<namespaces>>

	goto cleanup;
usage:
	fprintf(stderr, "Usage: %s -u -1 -m . -c /bin/sh ~\n", argv[0]);
error:
	err = 1;
cleanup:
	if (sockets[0]) close(sockets[0]);
	if (sockets[1]) close(sockets[1]);
	return err;
}
```

Since I'll be blacklisting system calls and capabilities, it's important to make sure there aren't any new ones.

**Listing 8: `<<check-linux-version>>` =**

```cpp
	fprintf(stderr, "=> validating Linux version...");
	struct utsname host = {0};
	if (uname(&host)) {
		fprintf(stderr, "failed: %m\n");
		goto cleanup;
	}
	int major = -1;
	int minor = -1;
	if (sscanf(host.release, "%u.%u.", &major, &minor) != 2) {
		fprintf(stderr, "weird release format: %s\n", host.release);
		goto cleanup;
	}
	if (major != 4 || (minor != 7 && minor != 8)) {
		fprintf(stderr, "expected 4.7.x or 4.8.x: %s\n", host.release);
		goto cleanup;
	}
	if (strcmp("x86_64", host.machine)) {
		fprintf(stderr, "expected x86_64: %s\n", host.machine);
		goto cleanup;
	}
	fprintf(stderr, "%s on %s.\n", host.release, host.machine);
```
*(This had a bug. captainjey on reddit let me know. Thanks!)*

And I wasn't quite at 500 lines of code, so I thought I had some space to build nice hostnames.

**Listing 9: `<<choose-hostname>>` =**

```cpp
int choose_hostname(char *buff, size_t len)
{
	static const char *suits[] = { "swords", "wands", "pentacles", "cups" };
	static const char *minor[] = {
		"ace", "two", "three", "four", "five", "six", "seven", "eight",
		"nine", "ten", "page", "knight", "queen", "king"
	};
	static const char *major[] = {
		"fool", "magician", "high-priestess", "empress", "emperor",
		"hierophant", "lovers", "chariot", "strength", "hermit",
		"wheel", "justice", "hanged-man", "death", "temperance",
		"devil", "tower", "star", "moon", "sun", "judgment", "world"
	};
	struct timespec now = {0};
	clock_gettime(CLOCK_MONOTONIC, &now);
	size_t ix = now.tv_nsec % 78;
	if (ix < sizeof(major) / sizeof(*major)) {
		snprintf(buff, len, "%05lx-%s", now.tv_sec, major[ix]);
	} else {
		ix -= sizeof(major) / sizeof(*major);
		snprintf(buff, len,
			 "%05lxc-%s-of-%s",
			 now.tv_sec,
			 minor[ix % (sizeof(minor) / sizeof(*minor))],
			 suits[ix / (sizeof(minor) / sizeof(*minor))]);
	}
	return 0;
}
```

### Namespaces

`clone` is the system call behind `fork()` et al. It's also the key to all of this. Conceptually we want to create a process with different properties than its parent: it should be able to mount a different `/`, set its own hostname, and do other things. We'll specify all of this by passing flags to `clone`[^4].

The child needs to send some messages to the parent, so we'll initialize a socketpair, and then make sure the child only receives access to one.

**Listing 10: `<<namespaces>>` +=**

```cpp
	if (socketpair(AF_LOCAL, SOCK_SEQPACKET, 0, sockets)) {
		fprintf(stderr, "socketpair failed: %m\n");
		goto error;
	}
	if (fcntl(sockets[0], F_SETFD, FD_CLOEXEC)) {
		fprintf(stderr, "fcntl failed: %m\n");
		goto error;
	}
	config.fd = sockets[1];
```

But first we need to set up room for a stack. We'll `execve` later, which will actually set up the stack again, so this is only temporary[^5].

**Listing 13: `<<namespaces>>` +=**

```cpp
	#define STACK_SIZE (1024 * 1024)

	char *stack = 0;
	if (!(stack = malloc(STACK_SIZE))) {
		fprintf(stderr, "=> malloc failed, out of memory?\n");
		goto error;
	}
```

We'll also prepare the cgroup for this process tree. More on this later.

**Listing 14: `<<namespaces>>` +=**

```cpp
	if (resources(&config)) {
		err = 1;
		goto clear_resources;
	}
```

We'll namespace the mounts, pids, IPC data structures, network devices, and hostname / domain name. I'll go into these more in the code for capabilities, cgroups, and syscalls.

**Listing 15: `<<namespaces>>` +=**

```cpp
	int flags = CLONE_NEWNS
		| CLONE_NEWCGROUP
		| CLONE_NEWPID
		| CLONE_NEWIPC
		| CLONE_NEWNET
		| CLONE_NEWUTS;
```

Stacks on x86, and almost everything else Linux runs on, grow downwards, so we'll add `STACK_SIZE` to get a pointer just below the end[^6]. We also `|` the flags with `SIGCHLD` so that we can `wait` on it.

**Listing 16: `<<namespaces>>` +=**

```cpp
	if ((child_pid = clone(child, stack + STACK_SIZE, flags | SIGCHLD, &config)) == -1) {
		fprintf(stderr, "=> clone failed! %m\n");
		err = 1;
		goto clear_resources;
	}
```

Close and zero the child's socket, so that if something breaks then we don't leave an open fd, possibly causing the child to or the parent to hang.

**Listing 17: `<<namespaces>>` +=**

```cpp
	close(sockets[1]);
	sockets[1] = 0;
```

The parent process will configure the child's user namespace and then pause until the child process tree exits[^7].

**Listing 21: `<<child>>` +=**

```cpp
#define USERNS_OFFSET 10000
#define USERNS_COUNT 2000

int handle_child_uid_map (pid_t child_pid, int fd)
{
	int uid_map = 0;
	int has_userns = -1;
	if (read(fd, &has_userns, sizeof(has_userns)) != sizeof(has_userns)) {
		fprintf(stderr, "couldn't read from child!\n");
		return -1;
	}
	if (has_userns) {
		char path[PATH_MAX] = {0};
		for (char **file = (char *[]) { "uid_map", "gid_map", 0 }; *file; file++) {
			if (snprintf(path, sizeof(path), "/proc/%d/%s", child_pid, *file)
			    > sizeof(path)) {
				fprintf(stderr, "snprintf too big? %m\n");
				return -1;
			}
			fprintf(stderr, "writing %s...", path);
			if ((uid_map = open(path, O_WRONLY)) == -1) {
				fprintf(stderr, "open failed: %m\n");
				return -1;
			}
			if (dprintf(uid_map, "0 %d %d\n", USERNS_OFFSET, USERNS_COUNT) == -1) {
				fprintf(stderr, "dprintf failed: %m\n");
				close(uid_map);
				return -1;
			}
			close(uid_map);
		}
	}
	if (write(fd, & (int) { 0 }, sizeof(int)) != sizeof(int)) {
		fprintf(stderr, "couldn't write: %m\n");
		return -1;
	}
	return 0;
}
```

The child process will send a message to the parent process about whether it should set uid and gid mappings. If that works, it will `setgroups`, `setresgid`, and `setresuid`. Both `setgroups` and `setresgid` are necessary here since there are two separate group mechanisms on Linux[^9]. I'm also assuming here that every uid has a corresponding gid, which is common but not necessarily universal.

**Listing 23: `<<child>>` +=**

```cpp
int userns(struct child_config *config)
{
	fprintf(stderr, "=> trying a user namespace...");
	int has_userns = !unshare(CLONE_NEWUSER);
	if (write(config->fd, &has_userns, sizeof(has_userns)) != sizeof(has_userns)) {
		fprintf(stderr, "couldn't write: %m\n");
		return -1;
	}
	int result = 0;
	if (read(config->fd, &result, sizeof(result)) != sizeof(result)) {
		fprintf(stderr, "couldn't read: %m\n");
		return -1;
	}
	if (result) return -1;
	if (has_userns) {
		fprintf(stderr, "done.\n");
	} else {
		fprintf(stderr, "unsupported? continuing.\n");
	}
	fprintf(stderr, "=> switching to uid %d / gid %d...", config->uid, config->uid);
	if (setgroups(1, & (gid_t) { config->uid }) ||
	    setresgid(config->uid, config->uid, config->uid) ||
	    setresuid(config->uid, config->uid, config->uid)) {
		fprintf(stderr, "%m\n");
		return -1;
	}
	fprintf(stderr, "done.\n");
	return 0;
}
```

And this is where the child process from `clone` will end up. We'll perform all of our setup, switch users and groups, and then load the executable. The order is important here: we can't change mounts without certain capabilities, we can't `unshare` after we limit the syscalls, etc.

**Listing 24: `<<child>>` +=**

```cpp
int child(void *arg)
{
	struct child_config *config = arg;
	if (sethostname(config->hostname, strlen(config->hostname))
	    || mounts(config)
	    || userns(config)
	    || capabilities()
	    || syscalls()) {
		close(config->fd);
		return -1;
	}
	if (close(config->fd)) {
		fprintf(stderr, "close failed: %m\n");
		return -1;
	}
	if (execve(config->argv[0], config->argv, NULL)) {
		fprintf(stderr, "execve failed! %m.\n");
		return -1;
	}
	return 0;
}
```

### Capabilities

`capabilities` subdivide the property of "being root" on Linux. It's useful to compartmentalize privileges so that, for example a process can allocate network devices (`CAP_NET_ADMIN`) but not read all files (`CAP_DAC_OVERRIDE`). I'll use them here to drop the ones we don't want.

But not all of "being root" is subvidivided into capabilities. For example, writing to parts of procfs is allowed by root even after having dropped capabilities[^10]. There are a lot of things like this: this is part of why need other restrictions beside capabilities.

It's also important to think about how we're dropping capabilities. `man 7 capabilities` has an algorithm for us:

```cpp
	During  an   execve(2),  the   kernel  calculates   the  new
	capabilities of the process using the following algorithm:

	    P'(ambient) = (file is privileged) ? 0 : P(ambient)

	    P'(permitted) = (P(inheritable) & F(inheritable)) |
					(F(permitted) & cap_bset) | P'(ambient)

	    P'(effective) = F(effective) ? P'(permitted) : P'(ambient)

	    P'(inheritable) = P(inheritable)    [i.e., unchanged]

	where:

	    P         denotes the  value of a thread  capability set
			    before the execve(2)

	    P'        denotes the  value of a thread  capability set
			    after the execve(2)

	    F         denotes a file capability set

	    cap_bset  is the  value of  the capability  bounding set
			    (described below).
```

We'd like `P'(ambient)` and `P(inheritable)` to be empty, and `P'(permitted)` and `P(effective)` to only include the capabilities above. This is achievable by doing the following:
- Clearing our own inheritable set. This clears the ambient set; `man 7 capabilities` says "The ambient capability set obeys the invariant that no capability can ever be ambient if it is not both permitted and inheritable." This also clears the child's inheritable set.
- Clearing the bounding set. This limits the file capabilities we'll gain when we `execve`, and the rest are limited by clearing the inheritable and ambient sets.

If we were to only drop our own effective, permitted and inheritable sets, we'd regain the permissions in the child file's capabilities. This is how `bash` can call `ping`, for example[^11].

#### Dropped capabilities

**Listing 29: `<<capabilities>>` +=**

```cpp
int capabilities()
{
	fprintf(stderr, "=> dropping capabilities...");
```

`CAP_AUDIT_CONTROL`, `_READ`, and `_WRITE` allow access to the audit system of the kernel (i.e. functions like `audit_set_enabled`, usually used with `auditctl`). The kernel prevents messages that normally require `CAP_AUDIT_CONTROL` outside of the first pid namespace, but it does allow messages that would require `CAP_AUDIT_READ` and `CAP_AUDIT_WRITE` from any namespace[^12]. So let's drop them all. We especially want to drop `CAP_AUDIT_READ`, since it isn't namespaced[^13] and may contain important information, but `CAP_AUDIT_WRITE` may also allow the contained process to falsify logs or DOS the audit system.

**Listing 32: `<<capabilities>>` +=**

```cpp
	int drop_caps[] = {
		CAP_AUDIT_CONTROL,
		CAP_AUDIT_READ,
		CAP_AUDIT_WRITE,
```

`CAP_BLOCK_SUSPEND` lets programs prevent the system from suspending, either with `EPOLLWAKEUP` or `/proc/sys/wake_lock`[^14]. Suspend isn't namespaced, so we'd like to prevent this.

**Listing 34: `<<capabilities>>` +=**

```cpp
		CAP_BLOCK_SUSPEND,
```

`CAP_DAC_READ_SEARCH` lets programs call `open_by_handle_at` with an arbitrary `struct file_handle *`. `struct file_handle` is in theory an opaque type, but in practice it corresponds to inode numbers. So it's easy to brute-force them, and read arbitrary files. This was used by Sebastian Krahmer to write a program to read arbitrary system files from within Docker in 2014[^15].

**Listing 36: `<<capabilities>>` +=**

```cpp
		CAP_DAC_READ_SEARCH,
```

`CAP_FSETID`, without user namespacing, allows the process to modify a setuid executable without removing the setuid bit. This is pretty dangerous! It means that if we include a setuid binary in a container, it's easy for us to accidentally leave a dangerous setuid root binary on our disk, which any user can use to escalate privileges[^16].

**Listing 40: `<<capabilities>>` +=**

```cpp
		CAP_FSETID,
```

`CAP_IPC_LOCK` can be used to lock more of a process' own memory than would normally be allowed[^17], which could be a way to deny service.

**Listing 43: `<<capabilities>>` +=**

```cpp
		CAP_IPC_LOCK,
```

`CAP_MAC_ADMIN` and `CAP_MAC_OVERRIDE` are used by the mandatory access control systems Apparmor, SELinux, and SMACK to restrict access to their settings. These aren't namespaced, so they could be used by the contained programs to circumvent system-wide access control.

**Listing 44: `<<capabilities>>` +=**

```cpp
		CAP_MAC_ADMIN,
		CAP_MAC_OVERRIDE,
```

`CAP_MKNOD`, without user namespacing, allows programs to create device files corresponding to real-world devices. This includes creating new device files for existing hardware. If this capability were not dropped, a contained process could re-create the hard disk device, remount it, and read or write to it[^18].

**Listing 47: `<<capabilities>>` +=**

```cpp
		CAP_MKNOD,
```

I was worried that `CAP_SETFCAP` could be used to add a capability to an executable and `execve` it, but it's not actually possible for a process to set capabilities it doesn't have[^19]. But! An executable altered this way could be executed by any unsandboxed user, so I think it unacceptably undermines the security of the system.

**Listing 51: `<<capabilities>>` +=**

```cpp
		CAP_SETFCAP,
```

`CAP_SYSLOG` lets users perform destructive actions against the syslog. Importantly, it doesn't prevent contained processes from reading the syslog, which could be risky. It also exposes kernel addresses, which could be used to circumvent kernel address layout randomization[^20].

**Listing 54: `<<capabilities>>` +=**

```cpp
		CAP_SYSLOG,
```

`CAP_SYS_ADMIN` allows many behaviors! We don't want most of them (`mount`, `vm86`, etc). Some would be nice to have (`sethostname`, `mount` for bind mounts…) but the extra complexity doesn't seem worth it.

**Listing 55: `<<capabilities>>` +=**

```cpp
		CAP_SYS_ADMIN,
```

`CAP_SYS_BOOT` allows programs to restart the system (the `reboot` syscall) and load new kernels (the `kexec_load` and `kexec_file` syscalls)[^21]. We absolutely don't want this. `reboot` is user-namespaced, and the `kexec*` functions only work in the root user namespace, but neither of those help us.

**Listing 59: `<<capabilities>>` +=**

```cpp
		CAP_SYS_BOOT,
```

`CAP_SYS_MODULE` is used by the syscalls `delete_module`, `init_module`, `finit_module`[^22], by the code for `kmod`[^23], and by the code for loading device modules with ioctl[^24].

**Listing 66: `<<capabilities>>` +=**

```cpp
		CAP_SYS_MODULE,
```

`CAP_SYS_NICE` allows processes to set higher priority on given pids than the default[^25]. The default kernel scheduler doesn't know anything about pid namespaces, so it's possible for a contained process to deny service to the rest of the system[^26].

**Listing 71: `<<capabilities>>` +=**

```cpp
		CAP_SYS_NICE,
```

`CAP_SYS_RAWIO` allows full access to the host systems memory with `/proc/kcore`, `/dev/mem`, and `/dev/kmem`[^27], but a contained process would need `mknod` to access these within the namespace[^28]. But it also allows things like `iopl` and `ioperm`, which give raw access to the IO ports[^29].

**Listing 76: `<<capabilities>>` +=**

```cpp
		CAP_SYS_RAWIO,
```

`CAP_SYS_RESOURCE` specifically allows circumventing kernel-wide limits, so we probably should drop it[^30]. But I don't think this can do more than DOS the kernel, in general[^31].

**Listing 78: `<<capabilities>>` +=**

```cpp
		CAP_SYS_RESOURCE,
```

`CAP_SYS_TIME`: setting the time isn't namespaced, so we should prevent contained processes from altering the system-wide time[^32].

**Listing 79: `<<capabilities>>` +=**

```cpp
		CAP_SYS_TIME,
```

`CAP_WAKE_ALARM`, like `CAP_BLOCK_SUSPEND`, lets the contained process interfere with suspend[^33], and we'd like to prevent that.

**Listing 81: `<<capabilities>>` +=**

```cpp
		CAP_WAKE_ALARM
	};
```

**Listing 82: `<<capabilities>>` +=**

```cpp
	size_t num_caps = sizeof(drop_caps) / sizeof(*drop_caps);
	fprintf(stderr, "bounding...");
	for (size_t i = 0; i < num_caps; i++) {
		if (prctl(PR_CAPBSET_DROP, drop_caps[i], 0, 0, 0)) {
			fprintf(stderr, "prctl failed: %m\n");
			return 1;
		}
	}
	fprintf(stderr, "inheritable...");
	cap_t caps = NULL;
	if (!(caps = cap_get_proc())
	    || cap_set_flag(caps, CAP_INHERITABLE, num_caps, drop_caps, CAP_CLEAR)
	    || cap_set_proc(caps)) {
		fprintf(stderr, "failed: %m\n");
		if (caps) cap_free(caps);
		return 1;
	}
	cap_free(caps);
	fprintf(stderr, "done.\n");
	return 0;
}
```

#### Retained Capabilities

It's important to keep track of the capabilities I'm not dropping, too.

I've heard multiple places[^34] that `CAP_DAC_OVERRIDE` might expose the same functionality as `CAP_DAC_READ_SEARCH` (i.e. `open_by_handle_at`), but as far as I can tell that isn't true. `shocker.c` doesn't get anywhere with only `CAP_DAC_OVERRIDE`[^35], and the only usage in the kernel is in the Unix permission-checking code[^36]. So my understanding is that `CAP_DAC_OVERRIDE` on its own doesn't allow processes to read outside of their mount namespaces ("DAC" or "Discretionary Access Control" refers here to ordinary unix permissions).

`CAP_FOWNER`, `CAP_LEASE`, and `CAP_LINUX_IMMUTABLE` all operate on files inside of the mount namespace.

Likewise, `CAP_SYS_PACCT` allows processes to switch accounting on and off for itself. The `acct` system call takes a path to log to (which must be within the mount namespace), and only operates on the calling process. We're not using process accounting in our containerization, so turning it off should be harmless as well[^37].

`CAP_IPC_OWNER` is only used by functions that respect IPC namespaces[^38]; since we're in a separate IPC namespace from the host, we can allow this.

`CAP_NET_ADMIN` lets processes create network devices; `CAP_NET_BIND_SERVICE` lets processes bind to low ports on those devices; `CAP_NET_RAW` lets processes send raw packets on those devices. Since we're going to isolate the networking with a virtual bridge, and the contained process is inside of a network namespace, these shouldn't be an issue[^39]. I was wondering whether we could recreate an existing device like `mknod` does, but I don't think it's possible[^40].

`CAP_SYS_PTRACE` doesn't allow ptrace across pid namespaces[^41]. `CAP_KILL` doesn't allow signals across pid namespaces[^42].

`CAP_SETUID` and `CAP_SETGID` have similar behaviors[^43]:
- `Make arbitrary manipulations of process UIDS and GIDs and supplementary GID list`, which will only apply to pids in the namespace.
- `forge UID (GID) when passing socket credentials via UNIX domain sockets` the mount namespace should prevent us from reading the host system's unix domain sockets.
- `write a user(group ID) mapping in a user namespace (see user_namespaces(7))`: this is `/proc/self/uid_map`, which will be hidden inside the container.

`CAP_SETPCAP` only lets processes add or drop capabilities they already effectively have; `man 7 capabilities` says:
> If file capabilities are supported: add any capability from the calling thread's bounding set to its inheritable set; drop capabilities from the bounding set (via prctl(2) PR_CAPBSET_DROP); make changes to the securebits flags.

We've dropped everything relevant from the bounding set, and dropping further capabilities should be harmless.

`CAP_SYS_CHROOT` is traditionally abused by changing root to a directory with a setuid root binary and tampered-with dynamic libraries[^44]. Additionally, it can be used to escape a chroot "jail"[^45]. Neither of those should be relevant in our setup so this should be harmless.

Brad Spengler, in "False Boundaries and Arbitrary Code Execution" says that `CAP_SYS_TTYCONFIG` can "temporarily change the keyboard mapping of an administrator's tty via the KDSETKEYCODE ioctl to cause a different command to be executed than intended", but again this is an `ioctl` against a device that should be impossible to access within the mount namespace.

### Mounts

The child process is in its own mount namespace, so we can unmount things that it specifically shouldn't have access to. Here's how:
- Create a temporary directory, and one inside of it.
- Bind mount of the user argument onto the temporary directory
- `pivot_root`, making the bind mount our root and mounting the old root onto the inner temporary directory.
- `umount` the old root, and remove the inner temporary directory.

But first we'll remount everything with `MS_PRIVATE`. This is mostly a convenience, so that the bind mount is invisible outside of our namespace.

**Listing 108: `<<mounts>>` =**

```cpp
<<pivot-root>>

int mounts(struct child_config *config)
{
	fprintf(stderr, "=> remounting everything with MS_PRIVATE...");
	if (mount(NULL, "/", NULL, MS_REC | MS_PRIVATE, NULL)) {
		fprintf(stderr, "failed! %m\n");
		return -1;
	}
	fprintf(stderr, "remounted.\n");

	fprintf(stderr, "=> making a temp directory and a bind mount there...");
	char mount_dir[] = "/tmp/tmp.XXXXXX";
	if (!mkdtemp(mount_dir)) {
		fprintf(stderr, "failed making a directory!\n");
		return -1;
	}

	if (mount(config->mount_dir, mount_dir, NULL, MS_BIND | MS_PRIVATE, NULL)) {
		fprintf(stderr, "bind mount failed!\n");
		return -1;
	}

	char inner_mount_dir[] = "/tmp/tmp.XXXXXX/oldroot.XXXXXX";
	memcpy(inner_mount_dir, mount_dir, sizeof(mount_dir) - 1);
	if (!mkdtemp(inner_mount_dir)) {
		fprintf(stderr, "failed making the inner directory!\n");
		return -1;
	}
	fprintf(stderr, "done.\n");

	fprintf(stderr, "=> pivoting root...");
	if (pivot_root(mount_dir, inner_mount_dir)) {
		fprintf(stderr, "failed!\n");
		return -1;
	}
	fprintf(stderr, "done.\n");

	char *old_root_dir = basename(inner_mount_dir);
	char old_root[sizeof(inner_mount_dir) + 1] = { "/" };
	strcpy(&old_root[1], old_root_dir);

	fprintf(stderr, "=> unmounting %s...", old_root);
	if (chdir("/")) {
		fprintf(stderr, "chdir failed! %m\n");
		return -1;
	}
	if (umount2(old_root, MNT_DETACH)) {
		fprintf(stderr, "umount failed! %m\n");
		return -1;
	}
	if (rmdir(old_root)) {
		fprintf(stderr, "rmdir failed! %m\n");
		return -1;
	}
	fprintf(stderr, "done.\n");
	return 0;
}
```

`pivot_root` is a system call lets us swap the mount at `/` with another. Glibc doesn't provide a wrapper for it, but includes a prototype in the man page. I don't really understand, but OK, we'll include our own.

**Listing 109: `<<pivot-root>>` =**

```cpp
int pivot_root(const char *new_root, const char *put_old)
{
	return syscall(SYS_pivot_root, new_root, put_old);
}
```

It's worth noting that I'm avoiding packing and unpackaging containers. This is fertile ground for vulnerabilities[^46]; I'll count on the user to ensure that the mounted directory doesn't contain trusted or sensitive files or hard links.

### System Calls

I'll be blacklisting system calls that I can demonstrate causing harm or sandbox escapes. Again this isn't the best way to do this, but it seems like the most illustrative.

Docker's documentation and default seccomp profile are reasonable sources for dangerous system calls[^47]. They also include obsolete sytem calls and calls that overlap with restricted capabilities; I'll ignore those.

#### Disallowed System Calls

**Listing 113: `<<syscalls>>` +=**

```cpp
#define SCMP_FAIL SCMP_ACT_ERRNO(EPERM)

int syscalls()
{
	scmp_filter_ctx ctx = NULL;
	fprintf(stderr, "=> filtering syscalls...");
	if (!(ctx = seccomp_init(SCMP_ACT_ALLOW))
```

We want to prevent new setuid / setgid executables from being created, since in the absence of user namespaces the contained process could create a setuid binary that could be used by any user to get root[^48].

**Listing 116: `<<syscalls>>` +=**

```cpp
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(chmod), 1,
				SCMP_A1(SCMP_CMP_MASKED_EQ, S_ISUID, S_ISUID))
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(chmod), 1,
				SCMP_A1(SCMP_CMP_MASKED_EQ, S_ISGID, S_ISGID))
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(fchmod), 1,
				SCMP_A1(SCMP_CMP_MASKED_EQ, S_ISUID, S_ISUID))
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(fchmod), 1,
				SCMP_A1(SCMP_CMP_MASKED_EQ, S_ISGID, S_ISGID))
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(fchmodat), 1,
				SCMP_A2(SCMP_CMP_MASKED_EQ, S_ISUID, S_ISUID))
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(fchmodat), 1,
				SCMP_A2(SCMP_CMP_MASKED_EQ, S_ISGID, S_ISGID))
```

Allowing contained processes to start new user namespaces can allow processes to gain new (albeit limited) capabilities, so we prevent it.

**Listing 117: `<<syscalls>>` +=**

```cpp
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(unshare), 1,
				SCMP_A0(SCMP_CMP_MASKED_EQ, CLONE_NEWUSER, CLONE_NEWUSER))
	    || seccomp_rule_add(ctx, SCMP_FAIL, SCMP_SYS(clone), 1,
				SCMP_A0(SCMP_CMP_MASKED_EQ, CLONE_NEWUSER, CLONE_NEWUSER))
```

*(... and other disallowed syscalls like TIOCSTI, ptrace, userfaultd, perf_event_open, etc.)*

#### Allowed System Calls

Here are the system calls that are disallowed by the default Docker policy but permitted by this code:

- `_sysctl` is obsolete and disabled by default[^57].
- `alloc_hugepages` and `free_hugepages`[^58], `bdflush`[^59], `create_module`[^60], `nfsservctl`[^61], `perfctr`[^62], `get_kernel_syms`[^63], and `setup`[^64] are not present on modern Linux.
- `clock_adjtime`, `clock_settime`[^65], and `adjtime`[^66] depend on `CAP_SYS_TIME`.
- `pciconfig_read` and `pciconfig_write`[^67] and all of the side-effecting operations of `quotactl`[^68] are prevented by `CAP_SYS_ADMIN`.
- `get_mempolicy` and `getpagesize` reveal information about the memory layout of the system, but they can be made by unprivileged processes, and are probably harmless. `pciconfig_iobase` can be made by unprivileged processes, and reveals information about PCI devices.
- `ustat`[^69] and `sysfs`[^70] leak some information about the filesystems, but are nothing that I see as critical. `uselib` is more-or-less obsolete, but is just used for loading a shared library in userspace[^71].
- `sync_file_range2` is `sync_file_range` with swapped argument order[^72].
- `readdir` is mostly obsolete, but probably harmless[^73].
- `kexec_file_load` and `kexec_load` are prevented by `CAP_SYS_BOOT`[^74].
- `nice` can only be used to lower priority without `CAP_SYS_NICE`[^75].
- `oldfstat`, `oldlstat`, `oldolduname`, `oldstat`, and `olduname` are just older versions of their respective functions. I expect them to have the same security properties as the modern ones.
- `perfmonctl`[^76] is only available on IA-64. `ppc_rtas`[^77], `spu_create`[^78] and `spu_run`[^79], and `subpage_prot`[^80] are only available on PowerPC. `utrap_install` is only available on Sparc[^81]. `kern_features` is only available on Sparc64, and should be harmless anyway[^82].
- I don't believe `pivot_root` is a problem in our setup (but it could probably be used to circumvent path-based MAC).
- `preadv2` and `pwritev2` are just extensions to `preadv` and `pwritev` / `readv` and `writev`, which are "scatter input" / "gather output" extensions to `read` and `write`[^83].

### Resources

We'd like to prevent badly-behaved child processes from denying service to the rest of the system[^84]. Cgroups let us limit memory and cpu time in particular; limiting the pid count and IO usage is also useful.

I'll set up a struct so I don't have to repeat myself too much, with the following instructions:
- Set `memory/$hostname/memory.limit_in_bytes`, so the contained process and its child processes can't total more than 1GB memory in userspace[^86].
- Set `memory/$hostname/memory.kmem.limit_in_bytes`, so that the contained process and its child processes can't total more than 1GB memory in kernel space[^87].
- Set `cpu/$hostname/cpu.shares` to 256. CPU shares are chunks of 1024; 256 * 4 = 1024, so this lets the contained process take a quarter of cpu-time on a busy system at most[^88].
- Set the `pids/$hostname/pids.max`, allowing the contained process and its children to have 64 pids at most. This is useful because there are per-user pid limits that we could hit on the host if the contained process occupies too many[^89].
- Set `blkio/$hostname/blkio.weight` to 50, so that it's lower than the rest of the system and prioritized accordingly[^90].

> [!warning] OOM Killer Consideration
> If a process consumes a lot of memory, and has a better `badness` score than some other critical host-side process, the host-side process will be killed by the kernel's out-of-memory killer. Limiting container memory prevents the container from unfairly influencing the host's OOM killer.

### Networking

Container networking takes a little too much explanation for this space. It usually works like this:
- Create a bridge device.
- Create a virtual ethernet pair and attach one end to the bridge.
- Put the other end in the network namespace.
- For outside networking access, the host needs to be set to forward (and possibly NAT) packets.

Having multiple contained processes sharing a bridge device would mean they're both on the same LAN from the host's perspective. So ARP spoofing is a recurring issue with containers that work this way[^94].

The canonical way to do this from C is the `rtnetlink` interface; it would probably be easier to use `ip link ...`.

We could also limit the network usage with the `net_prio` cgroup controller[^95].

---

## Footnotes & References

**[1]** Compiling with user namespaces changes the semantics of capabilities system-wide, which could cause more problems or at least confusion.

**[2, 3]** User namespaces are turned off by default in Linux at the time of writing, but many distributions apply patches to turn it on in a limited way.

**[4]** `clone` flags used: `CLONE_NEWNS`, `CLONE_NEWCGROUP`, `CLONE_NEWPID`, `CLONE_NEWIPC`, `CLONE_NEWNET`, `CLONE_NEWUTS`.

**[5]** Stack setup is temporary because `execve` will set it up again.

**[6]** Stacks on x86 grow downwards, so we add `STACK_SIZE` to get a pointer just below the end.

**[7]** Parent configures child's user namespace and pauses until child exits.

**[9]** Both `setgroups` and `setresgid` are necessary due to two separate group mechanisms on Linux.

**[10]** Writing to parts of procfs is allowed by root even after dropping capabilities.

**[11]** Dropping only effective/permitted/inheritable sets would regain permissions from child file's capabilities (e.g., `bash` calling `ping`).

**[12, 13]** `CAP_AUDIT_READ` isn't namespaced and may contain important info; `CAP_AUDIT_WRITE` could falsify logs.

**[14]** `CAP_BLOCK_SUSPEND` prevents system suspend via `EPOLLWAKEUP` or `/proc/sys/wake_lock`.

**[15]** `CAP_DAC_READ_SEARCH` allows `open_by_handle_at` brute-forcing (used in Docker escape in 2014).

**[16]** `CAP_FSETID` allows modifying setuid executables without removing the setuid bit.

**[17]** `CAP_IPC_LOCK` can lock more memory than allowed, enabling DoS.

**[18]** `CAP_MKNOD` allows creating device files for real-world hardware.

**[19]** `CAP_SETFCAP` could undermine security if altered executable is run by unsandboxed user.

**[20]** `CAP_SYSLOG` exposes kernel addresses, circumventing KASLR.

**[21]** `CAP_SYS_BOOT` allows `reboot` and `kexec_load`/`kexec_file`.

**[22, 23, 24]** `CAP_SYS_MODULE` used by `delete_module`, `init_module`, `finit_module`, `kmod`, and ioctl.

**[25, 26]** `CAP_SYS_NICE` allows setting higher priority, potentially denying service to the host system.

**[27, 28, 29]** `CAP_SYS_RAWIO` allows full access to host memory (`/proc/kcore`, `/dev/mem`) and raw IO ports (`iopl`, `ioperm`).

**[30, 31]** `CAP_SYS_RESOURCE` allows circumventing kernel-wide limits.

**[32]** `CAP_SYS_TIME`: setting time isn't namespaced.

**[33]** `CAP_WAKE_ALARM` lets contained process interfere with suspend.

**[34, 35, 36]** `CAP_DAC_OVERRIDE` does not expose `open_by_handle_at` functionality; only used in Unix permission-checking.

**[37]** `CAP_SYS_PACCT` operates only on calling process within mount namespace.

**[38]** `CAP_IPC_OWNER` respects IPC namespaces.

**[39, 40]** `CAP_NET_*` capabilities are safe due to network namespace isolation.

**[41, 42]** `CAP_SYS_PTRACE` and `CAP_KILL` do not work across pid namespaces.

**[43]** `CAP_SETUID`/`CAP_SETGID` manipulations apply only to pids in the namespace.

**[44, 45]** `CAP_SYS_CHROOT` abuses are not relevant in this `pivot_root` setup.

**[46]** Packing/unpacking containers is fertile ground for vulnerabilities.

**[47]** Docker's documentation and default seccomp profile are reasonable sources for dangerous syscalls.

**[48]** Preventing setuid/setgid creation stops contained processes from creating root-access binaries.

**[57]** `init/Kconfig:1420@c8d2bc`
```text
config SYSCTL_SYSCALL
	bool "Sysctl syscall support" if EXPERT
	depends on PROC_SYSCTL
	default n
	select SYSCTL
	---help---
	  sys_sysctl uses binary paths that have been found challenging
	  to properly maintain and use.  The interface in /proc/sys
	  using paths with ascii names is now the primary path to this
	  information.

	  Almost nothing using the binary sysctl interface so if you are
	  trying to save some space it is probably safe to disable this,
	  making your kernel marginally smaller.

	  If unsure say N here.
```

**[58]** `man 2 alloc_hugepages`
```text
DESCRIPTION
	The system calls alloc_hugepages() and free_hugepages() were
	introduced  in Linux  2.5.36  and removed  again in  2.5.54.
	They  existed  only  on  i386  and  ia64  (when  built  with
	CONFIG_HUGETLB_PAGE).  In Linux  2.4.20, the syscall numbers
	exist, but the calls fail with the error ENOSYS.
```

**[59]** `man 2 bdflush`
```text
DESCRIPTION
	Note: Since  Linux 2.6, this  system call is  deprecated and
	does nothing.   It is  likely to  disappear altogether  in a
	future  kernel release.   Nowadays,  the  task performed  by
	bdflush() is handled by the kernel pdflush thread.
```

**[60]** `man 2 create_module`
```text
DESCRIPTION
	Note: This  system call  is present  only in  kernels before
	Linux 2.6.
```

**[61]** `man 2 nfsservctl`
```text
NAME
	nfsservctl - syscall interface to kernel nfs daemon

SYNOPSIS
	#include <linux/nfsd/syscall.h>

	long nfsservctl(int cmd, struct nfsctl_arg *argp,
				 union nfsctl_res *resp);

DESCRIPTION
	Note: Since  Linux 3.1, this  system call no  longer exists.
	It  has  been  replaced  by  a set  of  files  in  the  nfsd
	filesystem; see nfsd(7).
```

**[62]** `man 2 syscalls`
```text
	perfctr(2)	2.2	Sparc; removed in 2.6.34
```

**[63]** `man 2 get_kernel_syms`
```text
GET_KERNEL_SYMS(2) -- 2016-10-08 -- Linux -- Linux Programmer's Manual

NAME
	get_kernel_syms  -  retrieve   exported  kernel  and  module
	symbols

SYNOPSIS
	#include <linux/module.h>

	int get_kernel_syms(struct kernel_sym *table);

	Note:  No declaration  of this  system call  is provided  in
	glibc headers; see NOTES.

DESCRIPTION
	Note: This  system call  is present  only in  kernels before
	Linux 2.6.
```

**[64]** `man 2 setup`
```text
SETUP(2) -- 2008-12-03 -- Linux -- Linux Programmer's Manual

NAME
	setup - setup devices and filesystems, mount root filesystem

	[...]

VERSIONS
	Since Linux 2.1.121, no such function exists anymore.
```

**[65]** `man 2 clock_settime`
```text
CLOCK_GETRES(2) -- 2016-05-09 -- Linux Programmer's Manual

NAME
        clock_getres, clock_gettime, clock_settime  - clock and time
        functions

        [...]

ERRORS
        EFAULT
                tp points outside the accessible address space.

        EINVAL
                The clk_id specified is not supported on this system.

        EPERM
                clock_settime()  does not  have permission  to set  the
                clock indicated.
```
but you can see in the source that `CLOCK_REALTIME` is the only clock with `.clock_set` and `.clock_adj` set:

**Listing 161: `kernel/time/posix-timers.c:282@c8d2bc`**
```cpp
    /*
     * Initialize everything, well, just everything in Posix clocks/timers ;)
     */
    static __init int init_posix_timers(void)
    {
            struct k_clock clock_realtime = {
                    .clock_getres   = posix_get_hrtimer_res,
                    .clock_get      = posix_clock_realtime_get,
                    .clock_set      = posix_clock_realtime_set,
                    .clock_adj      = posix_clock_realtime_adj,
                    .nsleep         = common_nsleep,
                    .nsleep_restart = hrtimer_nanosleep_restart,
                    .timer_create   = common_timer_create,
                    .timer_set      = common_timer_set,
                    .timer_get      = common_timer_get,
                    .timer_del      = common_timer_del,
            };
            // ... (other clocks omitted for brevity) ...
            posix_timers_register_clock(CLOCK_REALTIME, &clock_realtime);
            // ...
            return 0;
    }
```
and that those methods go through `settimeofday` and `adjtimex`, which are both also gated by `CAP_SYS_TIME`.

**Listing 162: `kernel/time/posix-timers.c:212@c8d2bc`**
```cpp
    /* Set clock_realtime */
    static int posix_clock_realtime_set(const clockid_t which_clock,
                                        const struct timespec *tp)
    {
            return do_sys_settimeofday(tp, NULL);
    }

    static int posix_clock_realtime_adj(const clockid_t which_clock,
                                        struct timex *t)
    {
            return do_adjtimex(t);
    }
```

**Listing 163: `security/commoncap.c:106@c8d2bc`**
```cpp
    /**
     * cap_settime - Determine whether the current process may set the system clock
     * @ts: The time to set
     * @tz: The timezone to set
     *
     * Determine whether the current process may set the system clock and timezone
     * information, returning 0 if permission granted, -ve if denied.
     */
    int cap_settime(const struct timespec64 *ts, const struct timezone *tz)
    {
            if (!capable(CAP_SYS_TIME))
                    return -EPERM;
            return 0;
    }
```

**Listing 164: `kernel/time/ntp.c:657@c8d2bc`**
```cpp
    /**
     * ntp_validate_timex - Ensures the timex is ok for use in do_adjtimex
     */
    int ntp_validate_timex(struct timex *txc)
    {
            if (txc->modes & ADJ_ADJTIME) {
                    /* singleshot must not be used with any other mode bits */
                    if (!(txc->modes & ADJ_OFFSET_SINGLESHOT))
                            return -EINVAL;
                    if (!(txc->modes & ADJ_OFFSET_READONLY) &&
                        !capable(CAP_SYS_TIME))
                            return -EPERM;
            } else {
                    /* In order to modify anything, you gotta be super-user! */
                     if (txc->modes && !capable(CAP_SYS_TIME))
                            return -EPERM;
                    /*
                     * if the quartz is off by more than 10% then
                     * something is VERY wrong!
                     */
                    if (txc->modes & ADJ_TICK &&
                        (txc->tick <  900000/USER_HZ ||
                         txc->tick > 1100000/USER_HZ))
                            return -EINVAL;
            }
            /* ... */
    }
```

**[66]** `man 3 adjtime`
```text
ADJTIME(3) -- 2016-03-15 -- Linux -- Linux Programmer's Manual

NAME
        adjtime - correct the time to synchronize the system clock

        [...]

ERRORS
        EINVAL
                The adjustment in delta is outside the permitted range.

        EPERM
                The caller does not have sufficient privilege to adjust
                the time.  Under Linux,  the CAP_SYS_TIME capability is
                required.
```

**[67]** `man 2 pciconfig_read`
```text
PCICONFIG_READ(2) -- 2016-07-17 -- Linux -- Linux Programmer's Manual

NAME
	pciconfig_read,  pciconfig_write,   pciconfig_iobase  -  pci
	device information handling
	[...]
ERRORS
	[...]
	EPERM
		User does not have  the CAP_SYS_ADMIN capability.  This
		does not apply to pciconfig_iobase().
```

**[68]** Too many too list, but see `man 2 quotactl`.

**[69]** `man 2 ustat`
```text
USTAT(2) -- 2003-08-04 -- Linux -- Linux Programmer's Manual

NAME
        ustat - get filesystem statistics

SYNOPSIS
        #include <sys/types.h>
        #include <unistd.h>    /* libc[45] */
        #include <ustat.h>     /* glibc2 */

        int ustat(dev_t dev, struct ustat *ubuf);

DESCRIPTION
        ustat() returns information about a mounted filesystem.  dev
        is a device number identifying a device containing a mounted
        filesystem.  ubuf  is a  pointer to  a ustat  structure that
        contains the following members:

            daddr_t f_tfree;      /* Total free blocks */
            ino_t   f_tinode;     /* Number of free inodes */
            char    f_fname[6];   /* Filsys name */
            char    f_fpack[6];   /* Filsys pack name */

        The  last   two  fields,   f_fname  and  f_fpack,   are  not
        implemented  and  will  always  be filled  with  null  bytes
        ('\0').
```

**[70]** `man 2 sysfs`
```text
SYSFS(2) -- 2010-06-27 -- Linux -- Linux Programmer's Manual

NAME
        sysfs - get filesystem type information

SYNOPSIS
        int sysfs(int option, const char *fsname);
        int sysfs(int option, unsigned int fs_index, char *buf);
        int sysfs(int option);

DESCRIPTION
        sysfs()  returns  information  about  the  filesystem  types
        currently present in  the kernel.  The specific  form of the
        sysfs()  call and  the information  returned depends  on the
        option in effect:
        1  Translate the filesystem identifier  string fsname into a
           filesystem type index.
        2  Translate  the  filesystem  type index  fs_index  into  a
           null-terminated   filesystem  identifier   string.
        3  Return  the total  number of  filesystem types  currently
           present in the kernel.
```

**[71]** `man 2 uselib`
```text
USELIB(2) -- 2016-03-15 -- Linux -- Linux Programmer's Manual

NAME
	uselib - load shared library

	[..]

NOTES
	[...]
	Since Linux  3.15, this system  call is available  only when
	the kernel is configured with the CONFIG_USELIB option.
```

**[72]** `man 2 sync_file_range2`
```text
SYNC_FILE_RANGE(2) -- 2014-08-19 -- Linux -- Linux Programmer's Manual

NAME
	sync_file_range - sync a file segment with disk

	[...]
NOTES
   sync_file_range2()
	Some   architectures  (e.g.,   PowerPC,  ARM)   need  64-bit
	arguments to be aligned in a suitable pair of registers.  On
	such architectures, the  call signature of sync_file_range()
	shown in the SYNOPSIS would force a register to be wasted as
	padding  between   the  fd   and  offset   arguments.
	Therefore,  these  architectures define  a different  system call
	that orders  the arguments suitably:

	    int sync_file_range2(int fd, unsigned int flags,
						off64_t offset, off64_t nbytes);
```

**[73]** `man 2 readdir`
```text
READDIR(2) -- 2013-06-21 -- Linux -- Linux Programmer's Manual

NAME
	readdir - read directory entry

SYNOPSIS
	int readdir(unsigned int fd, struct old_linux_dirent *dirp,
			  unsigned int count);

	Note: There  is no glibc  wrapper for this system  call; see
	NOTES.

DESCRIPTION
	This is  not the  function you are  interested in.   Look at
	readdir(3)  for the  POSIX conforming  C library  interface.
	This page  documents the bare kernel  system call interface,
	which is superseded by getdents(2).
```

**[74]** `man 2 kexec_file_load`
```text
NAME
	kexec_load, kexec_file_load  - load  a new kernel  for later
	execution
	[...]
ERRORS
	[...]
	EPERM
		The caller does not have the CAP_SYS_BOOT capability.
```

**[75]** `man 2 nice`
```text
NICE(2) -- 2016-03-15 -- Linux -- Linux Programmer's Manual

NAME
	nice - change process priority

	[...]
ERRORS
	EPERM
		The calling process attempted  to increase its priority
		by  supplying  a  negative  inc  but  has  insufficient
		privileges.  Under  Linux, the  CAP_SYS_NICE capability
		is   required.
```

**[76]** `man 2 perfmonctl`
```text
PERFMONCTL(2) -- 2013-02-13 -- Linux -- Linux Programmer's Manual

NAME
	perfmonctl - interface to IA-64 performance monitoring unit

	[...]

CONFORMING TO
	perfmonctl() is Linux-specific and  is available only on the
	IA-64 architecture.
```

**[77]** `man 2 syscalls`
```text
	ppc_rtas(2)	2.6.2	PowerPC only
```

**[78]** `man 2 spu_create`
```text
SPU_CREATE(2) -- 2015-12-28 -- Linux -- Linux Programmer's Manual

NAME
	spu_create - create a new spu context

SYNOPSIS
	#include <sys/types.h>
	#include <sys/spu.h>

	int spu_create(const char *pathname, int flags, mode_t mode);
	int spu_create(const char *pathname, int flags, mode_t mode,
				int neighbor_fd);

	Note: There  is no glibc  wrapper for this system  call; see
	NOTES.

DESCRIPTION
	The  spu_create() system  call is  used on  PowerPC machines
	that  implement the  Cell Broadband  Engine Architecture  in
	order  to access  Synergistic  Processor  Units (SPUs).
```

**[79]** `man 2 spu_run`
```text
SPU_RUN(2) -- 2012-08-05 -- Linux -- Linux Programmer's Manual

NAME
	spu_run - execute an SPU context

SYNOPSIS
	#include <sys/spu.h>

	int spu_run(int fd, unsigned int *npc, unsigned int *event);

	Note: There  is no glibc  wrapper for this system  call; see
	NOTES.

DESCRIPTION
	The spu_run() system  call is used on  PowerPC machines that
	implement the Cell Broadband Engine Architecture in order to
	access Synergistic Processor Units  (SPUs).
```

**[80]** `man 2 subpage_prot`
```text
SUBPAGE_PROT(2) -- 2012-07-13 -- Linux -- Linux Programmer's Manual

NAME
	subpage_prot -  define a  subpage protection for  an address
	range

	[...]

VERSIONS
	This  system call  is provided  on the  PowerPC architecture
	since Linux 2.6.25.  The system call is provided only if the
	kernel is configured  with CONFIG_PPC_64K_PAGES.
```

**[81]** `man 2 syscalls`
```text
	utrap_install(2)	2.2	Sparc only
```

**[82]** `man 2 syscalls`
```text
	kern_features(2)	3.7	Sparc64
```
This is pretty vague, so I looked at the source. It's only mentioned in an Sparc64-specific file:

**Listing 145: `arch/sparc/kernel/sys_sparc_64.c:648@c8d2bc`**
```cpp
asmlinkage long sys_kern_features(void)
{
	return KERN_FEATURE_MIXED_MODE_STACK;
}
```

**[83]** `man 2 preadv2`
```text
DESCRIPTION
	The readv() system  call reads iovcnt buffers  from the file
	associated  with the  file  descriptor fd  into the  buffers
	described by iov ("scatter input").

	The  writev()  system call  writes  iovcnt  buffers of  data
	described  by  iov to  the  file  associated with  the  file
	descriptor fd ("gather output").

	[...]

   preadv() and pwritev()
	The  preadv()  system  call combines  the  functionality  of
	readv() and pread(2).

	The  pwritev() system  call  combines  the functionality  of
	writev()  and  pwrite(2).

   preadv2() and pwritev2()
	These  system calls  are similar  to preadv()  and pwritev()
	calls, but add  a fifth argument, flags,  which modifies the
	behavior on a per-call basis.
```

**[84]** This isn't just a denial-of-service concern. If a process consumes a lot of memory, and has a better `badness` score than some other critical host-side process, the host-side process will be killed by the kernel's out-of-memory killer.

"Taming the OOM Killer" on LWN:
> The process to be killed in an out-of-memory situation is selected based on its badness score. The badness score is reflected in /proc/<pid>/oom_score. This value is determined on the basis that the system loses the minimum amount of work done, recovers a large amount of memory, doesn't kill any innocent process eating tons of memory, and kills the minimum number of processes (if possible limited to one). The badness score is computed using the original memory size of the process, its CPU time (utime + stime), the run time (uptime - start time) and its oom_adj value. The more memory the process uses, the higher the score. The longer a process is alive in the system, the smaller the score.

"gltext seems to leak memory eventually causing oom-killer to run":
> gltext is consuming large amounts of memory. Often being killed by oom-killer but eventually causing me not to be able to log into my computer disabling gltext from the list of possible screensavers caused the problem to go away.

"xscreensaver does not protect the system against its children":
> The thing is, a screensaver is **NOT** a critically important part of the system. It should die early if it is a resource hog. All you have to do is write "10" into /proc/PID/oom_adj and Bob's your uncle. Until then, Xscreensaver is failing its duties.

**[85]** (See OOM Killer discussion above)

**[86]** `man 7 cgroup_namespaces`
```text
	Cgroup namespaces virtualize the view of a process's cgroups
	(see   cgroups(7))  as   seen  via   /proc/[pid]/cgroup  and
	/proc/[pid]/mountinfo.

	Each  cgroup  namespace  has  its own  set  of  cgroup  root
	directories,  which are  the  base points  for the  relative
	locations displayed  in /proc/[pid]/cgroup.
```

**[87]** `Documentation/cgroup-v1/memory.txt@c8d2bc`
```text
Brief summary of control files.
[...]
 memory.limit_in_bytes		 # set/show limit of memory usage
```

**[88]** `Documentation/cgroup-v1/memory.txt@c8d2bc`
```text
Brief summary of control files.
[...]
 memory.kmem.limit_in_bytes      # set/show hard limit for kernel memory
```

**[89]** `man 7 cgroups`
```text
   Cgroups version 1 controllers
	Each of the  cgroups version 1 controllers is  governed by a
	kernel configuration  option (listed  below).

	cpu (since Linux 2.6.24; CONFIG_CGROUP_SCHED)
		Cgroups  can be  guaranteed  a minimum  number of  "CPU
		shares" when a  system is busy.  This does  not limit a
		cgroup's CPU usage if the CPUs are not busy.
```

**[90]** `Documentation/cgroup-v1/pids.txt@c8d2bc`
```text
						   Process Number Controller
						   =========================

Abstract
--------
The process number controller is used to allow a cgroup hierarchy to stop any
new tasks from being fork()'d or clone()'d after a certain limit is reached.

Since it is trivial to hit the task limit without hitting any kmemcg limits in
place, PIDs are a fundamental resource. As such, PID exhaustion must be
preventable in the scope of a cgroup hierarchy by allowing resource limiting of
the number of tasks in a cgroup.

Usage
-----
In order to use the `pids` controller, set the maximum number of tasks in
pids.max (this is not available in the root cgroup for obvious reasons). The
number of processes currently in the cgroup is given by pids.current.
```

for example,

**Listing 178: `forkbomb.c`**
```cpp
/* -*- compile-command: "gcc -Wall -Werror -static forkbomb.c -o forkbomb" -*- */
#include <stdio.h>
#include <unistd.h>
#include <errno.h>

int main (int argc, char  **argv)
{
	switch (fork()) {
	case -1:
		fprintf(stderr, "++ couldn't even fork once: %m\n");
		return 1;
	case 0:
		while (1) {
			switch (fork()) {
			case -1:
				break;
			case 0:
				fprintf(stderr, "++ successful fork.\n");
				break;
			default:
				break;
			}
		}
		break;
	default:
		while (1) sleep(1);
		break;
	}
	return 0;
}
```

```bash
[lizzie@empress l-c-i-500-l]$ sudo ./contained -m . -u 0 -c forkbomb
=> validating Linux version...4.7.10.201610222037-1-grsec on x86_64.
=> setting cgroups...memory...cpu...pids...blkio...done.
=> setting rlimit...done.
=> remounting everything with MS_PRIVATE...remounted.
=> making a temp directory and a bind mount there...done.
=> pivoting root...done.
=> unmounting /oldroot.0sOZgF...done.
=> trying a user namespace...writing /proc/2184/uid_map...writing /proc/2184/gid_map...done.
=> switching to uid 0 / gid 0...done.
=> dropping capabilities...bounding...inheritable...done.
=> filtering syscalls...done.
++ successful fork.
++ successful fork.
... (repeated many times) ...
C-c C-c
```

**[91]** `Documentation/cgroup-v1/blkio-controller.txt@c8d2bc`
```text
Details of cgroup files
=======================
Proportional weight policy files
--------------------------------
- blkio.weight
	- Specifies per cgroup weight. This is default weight of the group
	  on all the devices until and unless overridden by per device rule.
	  Currently allowed range of weights is from 10 to 1000.
```

**[92]** `man 7 cgroups`
```text
   Creating cgroups and moving processes
	A cgroup filesystem initially contains a single root cgroup,
	'/', which all processes belong to.  A new cgroup is created
	by creating a directory in the cgroup filesystem:

	    mkdir /sys/fs/cgroup/cpu/cg1

	A process  may be moved  to this  cgroup by writing  its PID
	into the cgroup's cgroup.procs file:

	    echo $$ > /sys/fs/cgroup/cpu/cg1/cgroup.procs
```

**[93]** `man 2 setrlimit`
```text
	The soft limit is the value that the kernel enforces for the
	corresponding resource.   The hard  limit acts as  a ceiling
	for the soft limit: an unprivileged process may set only its
	soft limit  to a value  in the range from  0 up to  the hard
	limit,  and   (irreversibly)  lower   its  hard   limit.   A
	privileged    process   (under    Linux:   one    with   the
	CAP_SYS_RESOURCE capability)  may make arbitrary  changes to
	either limit value.
```

**[94]** "Cross-Container ARP Poisoning", an LXC bug report by Jesse Hertz of NCCGroup
```text
Description:
An unprivileged LXC container can conduct an ARP spoofing attack
against another unprivileged LXC container running on the same
host. This allows man-in-the-middle attacks on another container's
traffic.

Recommendation:
Due to the complex nature of this involving the Linux bridge
interface, NCC is not aware of an easy fix. We suggest involving the
kernel networking team to allow for ARP restrictions on virtual bridge
interfaces. Using ebtables to block and control link layer traffic may
also be an effective fix.

Stéphane Graber (stgraber) wrote on 2016-02-22:	#1
Hi,
Thanks for the report. This is not exactly news to us and has been
mentioned publicly a few times.
Our usual answer to this is that if you don't trust your users, you
shouldn't grant them access to a shared bridge, instead setup a
separate bridge for them.
```

**[95]** `man 7 cgroups`
```text
   Cgroups version 1 controllers
	Each of the  cgroups version 1 controllers is  governed by a
	kernel configuration  option (listed  below).
[...]
	net_prio (since Linux 3.3; CONFIG_CGROUP_NET_PRIO)
		This  allows priorities  to be  specified, per  network
		interface, for cgroups.
```
