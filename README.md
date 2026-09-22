# Wired-wireless-networks-subnet-mask-IP-addressing-
Troubleshot a ping failure on a switched LAN by identifying an undersized subnet mask and recalculating the correct addressing to cover all connected hosts.

# Subnetting Troubleshooting Lab (Packet Tracer: Diagnosing a Failed Ping Across a Misconfigured Subnet)

`Cisco Packet Tracer` · `Subnetting` · `CIDR` · `Network Troubleshooting` · `IPv4 Addressing`

## Overview
This lab was a **subnetting and connectivity troubleshooting exercise** in Cisco Packet Tracer. A single switch (`2960-24TT`) connects seven end devices, a mix of PCs and a laptop, all addressed in what was supposed to be one small subnet. Two of the devices could not be pinged from the rest of the network, and the task was to figure out why and work out the correct addressing scheme by hand.

Rather than just guessing at a fix, I wanted to actually walk through the subnet math: confirm how many usable hosts the current mask supports, compare that against how many devices are actually on the network, and use that to justify a new subnet mask instead of just picking one that "feels" right.

## Objective
Given a flat topology where some hosts can't reach each other, determine whether the subnet mask in use is large enough for the number of devices on the network, and calculate a corrected subnet mask (and addressing scheme) that actually accommodates every host.

## Environment
- **Switch:** `2960-24TT` (`Switch0`), Fa0/1 through Fa0/7 connected to each end device
- **End devices (7 total):**
  | Device | IP Address |
  |--------|-----------|
  | PC2 | 200.168.1.1 |
  | Laptop0 | 200.168.1.2 |
  | PC0 | 200.168.1.3 |
  | PC1 | 200.168.1.4 |
  | PC4 | 200.168.1.6 |
  | "C5" (PC-PT) | 200.168.1.9 |
  | PC3 | 200.168.2.5 |
- **Symptom:** several hosts were unable to ping PC5 and PC3
- **Lab platform:** Cisco Packet Tracer, Realtime mode

## Tools I Used

| Tool | What It Does | Why I Used It |
|------|--------------|----------------|
| **Cisco Packet Tracer** | Network simulation | Built and observed the topology, ran pings between hosts to confirm which connections were failing |
| **Subnetting by hand** | Binary/CIDR math | Worked out host counts, subnet boundaries, and a corrected mask instead of trusting the existing configuration |

## What I Did

### Confirming the Symptom
1. Used the topology's ping tests to confirm that several PCs could not reach PC5 and PC3, while pings between the rest of the hosts worked fine.
2. Started from the assumption that the problem was addressing related rather than a physical/Layer 2 issue, since the switch showed all links up and every device had an IP assigned.

### Checking the Subnet Mask Against the Host Count
1. The subnet mask in use was `255.255.255.248`, a /29.
2. Converted that to binary to confirm the host bits: `11111111.11111111.11111111.11111000`, which leaves 3 host bits.
3. Ran the host count formula for those 3 bits: 2³ = 8, minus the network and broadcast addresses, leaving **6 usable hosts**.
4. Counted the actual devices on the network: **7**. A /29 only has room for 6 hosts, so as soon as a 7th device was added, the subnet mask was one host short of what the network actually needed.

### Recalculating the Subnet
1. Since 6 usable hosts wasn't enough, I moved up to the next block size and tested 4 host bits instead of 3: 2⁴ = 16, minus network and broadcast, leaving **14 usable hosts**, comfortably more than the 7 devices on the network, with room to grow.
2. That corresponds to a subnet mask of `255.255.255.240`, written in binary as `11111111.11111111.11111111.11110000`.
3. Wrote the corrected network out in CIDR notation: `200.168.1.0/28`.

### Spotting the Second Issue
1. While working through the addressing, I noticed PC5 ("C5") is addressed as `200.168.1.9`. Under the *original* /29 mask, the valid host range for `200.168.1.0/29` is only `.1` through `.6`. `.9` falls outside that block entirely, in the next /29 block over. That alone explains why hosts on the first block couldn't reach it, independent of the host count problem.
2. PC3 is addressed as `200.168.2.5`, a different third octet entirely from the rest of the `200.168.1.x` devices. That's not a subnet mask problem at all; it's a completely separate network, and without a router in between to route traffic across it, no amount of remasking `200.168.1.0` will make PC3 reachable. Moving to a /28 fixes the host count shortfall and pulls `.9` back inside the valid range, but PC3 needs its address corrected (or a routed path added) to be reachable at all.

## What's in This Repo

```
subnetting-troubleshooting-lab/
├── README.md                        # This file
└── screenshots/
    ├── 01-topology-overview.png      # Full topology with IPs and ping failures
    ├── 02-subnet-math-worksheet.png  # /29 host-count calculation
    ├── 03-corrected-subnet-math.png  # /28 recalculation
    └── 04-ping-verification.png      # Pings after correction
```

## Skills I Picked Up
- **Checking host count against a mask before assuming a config is correct.** A /29 looks reasonable at a glance, but it only takes one extra device to outgrow it, and the network won't announce that on its own; you have to do the math.
- **Reading a subnet boundary correctly**, and recognizing that an address like `.9` can look "close enough" to a `.1` through `.6` range while actually sitting in a completely different block.
- **Separating a masking problem from a routing problem.** Not every unreachable host is fixed by a bigger subnet. `200.168.2.5` needed its own correction, not a wider mask on a different network.
- **Working the binary out by hand** instead of jumping straight to a subnet calculator, which made it much easier to explain *why* /28 was the right next size instead of just stating the answer.

## How This Applies in the Real World
Outgrowing a subnet is one of the most common real-world causes of "random" connectivity issues. A network gets sized correctly on day one, then a device or two gets added later and nobody revisits the mask. The fix isn't guesswork; it's going back to the host count, confirming the mask actually covers it, and picking the smallest block size that still leaves headroom.

The second issue here is just as realistic: assuming everything is a masking problem when one host is simply misconfigured onto the wrong network entirely. Part of troubleshooting is confirming that every address you're looking at actually belongs where it's supposed to before you start changing the subnet.

## Where I'm Coming From
I'm making the jump into cybersecurity from a background in **healthcare**. It's a different field on paper, but a lot of the muscle memory carries over: following procedures carefully, protecting sensitive information, staying calm and methodical when something isn't working the way it's supposed to. I'm currently studying for **CompTIA Security+** and building labs like this one to get real hands-on reps in, since that's what I'm missing on paper right now compared to my experience.

## What I Want to Learn Next
- Adding a router to the topology and practicing inter-VLAN or inter-subnet routing so a host like PC3, on a genuinely different network, can be reached the correct way
- Practicing subnetting in both directions: given a required host count, finding the mask, and given a mask, finding the valid host range, until it's fast without a calculator
- Building out VLSM (variable-length subnet masking) examples where different segments of the same network need different-sized subnets
- Getting faster at spotting addressing mistakes just by scanning a topology, instead of needing to work every host out by hand

## Limitations & What I'd Do Differently in Production
- **This was a single flat network with no router**, so the PC3 issue could only be fixed by re-addressing it, not by routing. A production network would more likely use a router or Layer 3 switch to connect genuinely separate subnets on purpose.
- **The corrected /28 leaves room for growth but isn't documented anywhere outside this lab.** In a real environment, subnet allocations should be tracked in an IP address management (IPAM) sheet or tool so the next person doesn't have to reverse-engineer the math from scratch.
- **No DHCP was involved**; every address was static. A real network this size would likely use DHCP with a reservation for anything that needs a fixed address, which reduces the chance of a device getting manually assigned an address outside the intended range in the first place.

## References
- [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer)
- [Subnetting Practice: CIDR Basics](https://www.comptia.org/certifications/security)
- [CompTIA Security+ (SY0-701) Exam Objectives](https://www.comptia.org/certifications/security)
