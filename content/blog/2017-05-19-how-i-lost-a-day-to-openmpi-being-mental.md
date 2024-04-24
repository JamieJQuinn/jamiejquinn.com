---
title: "How I lost a Day to OpenMPI Being Mental"
date: "2017-05-19 13:28:58 +0100"
tags:
- Code
tags:
- openmpi
- code
- hpc
- mpi
- fortran
image: openmpi-being-mental/cover.jpg
cover_author: Charles Deluvio
cover_link: https://unsplash.com/@charlesdeluvio?utm_medium=referral&amp;utm_campaign=photographer-credit
---

So at Glasgow Uni we have this little cluster for the maths department which happens to including about ten machines set up to work with torque (a job scheduling system). I discovered that these machines hadn't had _anything_ run on them for literally months, what a waste of resources! To rectify this atrocity I decided to try and run my MPI enabled code on _all ten machines_.

### Problem One
Turns out two of the machines have eight cores and the other eight have twelve cores. I can't mix and match with core number (AFAIK) so I guess I'll just run the code on _all eight machines_.

### Problem Two
The vendor supplied fortran compiler and openmpi library are pretty old and pretty useless. They produce binaries that run about 3x slower than current versions so I guess I'll have to build the latest versions myself.

### Problem Three
I run the code on a single machine. Works great, not so great performance but I can perhaps sort that out later. I run the code on multiple machines and __instant__ crash.

```
ERROR: received unexpected process identifier
```

Google doesn't help, the only result is someone trying to run openmpi on one machine with multiple IP addresses on a single interface. So I spend _six hours_ looking through the verbose MPI output to discover that upon MPI setting itself up, each MPI process contacts every other process.

Contacting another process on the same machine? Absolutely fine, nae bother.

Contacting another machine over the network? Not fine, definitely bother.

Why? It was contacting the other machines through the interface `virbr0` which happens to have the _same IP address_ on _every single machine_. So the conversation between a machine _and itself_ was going something like this:

Machine A: Hey, 192.168.1.1, you're running this MPI thing?\\
Machine A: Do I know you?\\
Machine A: Apparently not.\\
Machine A: Well, let's decide to produce a very confusing error message and hang.\\
Machine A: Agreed.

Machine B: Me too.

After I found this out simply passing `--mca btl_tcp_if_exclude virbr0,lo` to `mpirun` suppressed any contact through that odd interface and loopback and it worked perfectly.

### Problem Four
It crashes on more than four machines.

### Problem Five
It's about 7 times slower than running on a single one of the newer machines in the cluster.

### Lesson Learned:
Sometimes clusters are better left unused.
