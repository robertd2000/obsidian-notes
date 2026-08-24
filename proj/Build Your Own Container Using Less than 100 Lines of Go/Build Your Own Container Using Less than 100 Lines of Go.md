The problem with analogies is that they tend to turn your brain off when you hear them. Some may say that software architecture is "just like" building architecture. No, it’s not, and the fact that the analogy sounds good has arguably resulted in quite a lot of harm. In a related fashion, software containerisation is often pitched as providing the ability to move software around “just like” shipping containers move goods around. Not quite. Or at least, it is, but the analogy loses a lot of the detail.

Shipping containers and software containers do share a lot in common. Shipping containers - with their standard shape and size - enable powerful economies of scale and standardisation. And software containers promise many of the same benefits. But, this is a surface-level analogy - a goal rather than a fact.

To really understand what a container is in the world of software, we need to understand what goes into making one. And that's what this article is explains. In the process we’ll talk about containers vs containerisation, linux containers (including namespaces, cgroups and layered filesystems), then we’ll walk through some code to build a simple container from scratch, and finally talk about what this all really means.

## What is a Container, really?

I’d like to play a game. In your head, right now, tell me what a “container” is. Done? Ok. Let me see if I can guess what you might’ve said:

You might have said one or more of:

- A way to share resources
- Process Isolation
- Kind of like lightweight virtualisation
- Packaging a root filesystem and metadata together
- Kind of like a chroot jail
- Something something shipping container something
- Whatever docker does

That is quite a lot of things for one word to mean! The word “container” has started to be used for a lot of (sometimes overlapping) concepts. It is used for the analogy of containerisation, and for the technologies used to implement it. If we consider these separately, we get a clearer picture. So, let’s talk about the why of containers, and then the how. (Then we’ll come back to why, again).

## In the beginning

In the beginning, there was a program. Let's call the program run.sh, and what we’d do is we’d copy it to a remote server, and we would run it. However, running arbitrary code on remote computers is insecure and hard to manage and scale. So we invented virtual private servers and user permissions. And things were good.

But little run.sh had dependencies. It needed certain libraries to exist on the host. And it never worked quite the same remotely and locally. (Stop me if you’ve heard this tune). So we invented AMIs (Amazon Machine Images) and VMDKs (VMware images) and Vagrantfiles and so on, and things were good.

Well, they were kind-of good. The bundles were big and it was hard to ship them around effectively because they weren’t very standardised. And so, we invented caching.

![](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality\(85\)/filters:no_upscale\(\)/articles/build-a-container-golang/en/resources/fig1.jpg)

And things were good.

Caching is what makes Docker images so much more effective than vmdks or vagrantfiles. It lets us ship the deltas over some common base images rather than moving whole images around. It means we can afford to ship the entire environment from one place to another. It’s why when you `docker run whatever` it starts close to immediately even though whatever described the entirety of an operating system image. We’ll talk in more detail about how this works in (section N).

And, really, that’s what containers are about. They’re about bundling up dependencies so we can ship code around in a repeatable, secure way. But that’s a high level goal, not a definition. So let’s talk about the reality.

## Building a Container

So (for real this time!) what is a container? It would be nice if creating a container was as simple as just a create_container system call. It’s not. But honestly, it’s close.

To talk about containers at the low level, we have to talk about three things. These things are namespaces, cgroups and layered filesystems. There are other things, but these three make up the majority of the magic.

### Namespaces

Namespaces provide the isolation needed to run multiple containers on one machine while giving each what appears like it’s own environment. There are - at the time of writing - six namespaces. Each can be independently requested and amounts to giving a process (and its children) a view of a subset of the resources of the machine.

The namespaces are:

- **PID**: The pid namespace gives a process and its children their own view of a subset of the processes in the system. Think of it as a mapping table. When a process in a pid namespace asks the kernel for a list of processes, the kernel looks in the mapping table. If the process exists in the table the mapped ID is used instead of the real ID. If it doesn’t exist in the mapping table, the kernel pretends it doesn’t exist at all. The pid namespace makes the first process created within it pid 1 (by mapping whatever its host ID is to 1), giving the appearance of an isolated process tree in the container.
- **MNT**: In a way, this one is the most important. The mount namespace gives the process’s contained within it their own mount table. This means they can mount and unmount directories without affecting other namespaces (including the host namespace). More importantly, in combination with the pivot_root syscall - as we’ll see - it allows a process to have its own filesystem. This is how we can have a process think it’s running on ubuntu, or busybox, or alpine — by swapping out the filesystem the container sees.
- **NET**: The network namespace gives the processes that use it their own network stack. In general only the main network namespace (the one that the processes that start when you start your computer use) will actually have any real physical network cards attached. But we can create virtual ethernet pairs — linked ethernet cards where one end can be placed in one network namespace and one in another creating a virtual link between the network namespaces. Kind of like having multiple ip stacks talking to each other on one host. With a bit of routing magic this allows each container to talk to the real world while isolating each to its own network stack.
- **UTS**: The UTS namespace gives its processes their own view of the system’s hostname and domain name. After entering a UTS namespace, setting the hostname or the domain name will not affect other processes.
- **IPC**: The IPC Namespace isolates various inter-process communication mechanisms such as message queues. See the [Namespace](http://man7.org/linux/man-%20pages/man7/namespaces.7.html) docs for more details.
- **USER**: The user namespace was the most recently added, and is the likely the most powerful from a security perspective. The user namespace maps the uids a process sees to a different set of uids (and gids) on the host. This is extremely useful. Using a user namespace we can map the container's root user ID (i.e. 0) to an arbitrary (and unprivileged) uid on the host. This means we can let a container think it has root access - we can even actually give it root-like permissions on container-specific resources - without actually giving it any privileges in the root namespace. The container is free to run processes as uid 0 - which normally would be synonymous with having root permissions - but the kernel is actually mapping that uid under the covers to an unprivileged real uid. Most container systems don't map any uid in the container to uid 0 in the calling namespace: in other words there simply isn't a uid in the container that has real root permissions.

Most container technologies place a user’s process into all of the above namespaces and initialise the namespaces to provide a standard environments. This amounts to, for example, creating an initial internet card in the isolated network namespace of the container with connectivity to a real network on the host.

### CGroups

Cgroups could honestly be a whole article of their own (and I reserve the right to write one!). I'm going to address them fairly briefly here because there's not a lot you can't find directly in the documentation once you understand the concepts.

Fundamentally cgroups collect a set of process or task ids together and apply limits to them. Where namespaces isolate a process, cgroups enforce fair (or unfair - it's up to you, go crazy) resource sharing between processes.

Cgroups are exposed by the kernel as a special file system you can mount. You add a process or thread to a cgroup by simply adding process ids to a tasks file, and then read and configure various values by essentially editing files in that directory.

### Layered Filesystems

Namespaces and CGroups are the isolation and resource sharing sides of containerisation. They’re the big metal sides and the security guard at the dock. Layered Filesystems are how we can efficiently move whole machine images around: they're why the ship floats instead of sinks.

At a basic level, layered filesystems amount to optimising the call to create a copy of the root filesystem for each container. There are numerous ways of doing this. [Btrfs](https://btrfs.wiki.kernel.org/) uses copy on write at the filesystem layer. [Aufs](https://en.wikipedia.org/wiki/Aufs) uses “union mounts”. Since there are so many ways to achieve this step, this article will just use something horribly simple: we’ll really do a copy. It’s slow, but it works.

### Building the Container

**Step One: Setting up the skeleton**

Let’s just get the rough skeleton in place first. Assuming you have the latest version of the [golang programming language SDK](https://golang.org/doc/install) installed, then open an editor, and copy in the following listing.

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	switch os.Args[1] {
	case "run":
		parent()
	case "child":
		child()
	default:
		panic("wat should I do")
	}
}

func parent() {
	cmd := exec.Command("/proc/self/exe", append([]string{"child"}, os.Args[2:]...)...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	}
}

func child() { 
	cmd := exec.Command(os.Args[2], os.Args[3:]...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	}
}

func must(err error) {
	if err != nil {
		panic(err)
	}
}
```

So what does this do? Well we start in main.go and we read the first argument. If it’s ‘run’ then we run the parent() method, if it’s child() we run the child method. The parent method runs `/proc/self/exe` which is a special file containing an in-memory image of the current executable. In other words, we re-run ourselves, but passing child as the first argument.

What is this craziness? Well, right now, not much. It just lets us execute another program that executes a user-requested program (supplied in `os.Args[2:]` ). With this simple scaffolding, though, we can create a container.

**Step Two: Adding namespaces**

To add some namespaces to our program, we just need to add a single line. On line. On the second line of the parent() method, just add this line to tell go to pass some extra flags when it runs the child process.

```go
cmd.SysProcAttr = &syscall.SysProcAttr{
	Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
}
```

If you run your program now, your program will be running inside the UTS, PID and MNT namespaces!

**Step Three: The Root Filesystem**

Currently your process is in an isolated set of namespaces (feel free to experiment with adding the other namespaces to your Cloneflags above at this point). But the filesystem looks the same as the host. This is because you’re in a mount namespace, but the initial mounts are inherited from the creating namespace.

Let’s change that. We need the following four simple lines to swap into a root filesystem. Place them right at the start of the `child()` function.

```go
must(syscall.Mount("rootfs", "rootfs", "", syscall.MS_BIND, ""))
	must(os.MkdirAll("rootfs/oldrootfs", 0700))
	must(syscall.PivotRoot("rootfs", "rootfs/oldrootfs"))
	must(os.Chdir("/"))
```

The final two lines are the important bit, they tell the OS to move the current directory at `/` to `rootfs/oldrootfs` , and to swap the new rootfs directory to `/` . After the `pivotroot` call is complete, the / directory in the container will refer to the rootfs. (The bind mount call is needed to satisfy some requirements of the `pivotroot` command — the OS requires that `pivotroot` be used to swap two filesystems that are not part of the same tree, which bind mounting the rootfs to itself achieves. Yes, it’s pretty silly).

**Step Four: Initialising the world of the container**

At this point you have a process running in a set of isolated namespaces, with a root filesystem of your choosing. We’ve skipped setting up cgroups, although this is pretty simple, and we’ve skipped the root filesystem management that lets you efficiently download and cache the root filesystem images we `pivotroot`-ed into.

We’ve also skipped the container setup. What you have here is a fresh container in isolated namespaces. We have set the mount namespace by pivoting to the rootfs, but the other namespaces have their default contents. In a real container we’d need to configure the ‘world’ for the container before running the user process. So, for example, we’d set up networking, swap to the correct uid before running the process, set up any other limits we want (such as dropping capabilities and setting rlimits) and so on. This might well nudge us over 100 lines.

**Step Five: Putting it Together**

So here it is, a super super simple container, in (way) less than 100 lines of go. Obviously this is intentionally simple. If you use it in production, you are crazy and, more importantly, on your own. But I think seeing something simple and hacky gives a really useful picture of what’s going on. So let’s look through Listing A.

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	switch os.Args[1] {
	case "run":
		parent()
	case "child":
		child()
	default:
		panic("wat should I do")
	}
}

func parent() {
	cmd := exec.Command("/proc/self/exe", append([]string{"child"}, os.Args[2:]...)...)
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	}
}

func child() {
	must(syscall.Mount("rootfs", "rootfs", "", syscall.MS_BIND, ""))
	must(os.MkdirAll("rootfs/oldrootfs", 0700))
	must(syscall.PivotRoot("rootfs", "rootfs/oldrootfs"))
	must(os.Chdir("/"))

	cmd := exec.Command(os.Args[2], os.Args[3:]...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	}
}

func must(err error) {
	if err != nil {
		panic(err)
	}
}
```

## So, what does it mean?

Here’s where I’m going to be a bit controversial. To me, a container is a fantastic way to ship things around and run code cheaply with a good deal of isolation, but that isn’t the end of the conversation. Containers are a technology, not a user experience.

As a user I don’t want to push around containers into production any more than a shopper using amazon.com wants to actually phone the docks to organise shipment of their goods. Containers are a fantastic technology to build on top of, but we shouldn’t be distracted by an ability to move machine images around from the need to build really great developer experiences.

Platforms as a Service (PaaS) built on top of containers, such as [Cloud Foundry](https://www.cloudfoundry.org/), start with a user experience based on _code_ rather than containers. For most developers, what they want to do is to push their code and have it run. Behind the scenes, Cloud Foundry - and most other PaaSes - take that code and create a containerised image which is scaled and managed. In the case of Cloud Foundry this uses a buildpack, but you can skip this step and push a Docker image created from a Dockerfile too.

With a PaaS, all of the advantages of containers are still present - consistent environments, efficient resource management etc - but by controlling the user experience a PaaS can both offer a simpler user experience for a developer and perform a few extra tricks like patching the root file system when there are security vulnerabilities. What’s more, platforms provide things such as databases and message queues as _services_ you can bind to your apps, removing the need to think of everything as containers.

So, we have examined what containers are. Now, what shall we do with them?