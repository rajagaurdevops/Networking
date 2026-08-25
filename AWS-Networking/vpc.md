 # AWS VPC

## Networking Reference Guide

### DevOps Learning Notes

## Topics Covered

- VPC fundamentals
- Subnets
- Route tables
- Internet and NAT gateways
- VPC peering

---

# Part I: VPC Fundamentals

## 1. What Is a VPC?

An AWS VPC (Virtual Private Cloud) is a logically isolated virtual network in
AWS. It provides a networking environment where resources such as EC2
instances, load balancers, and databases can communicate using private IP
addresses.

### Real-Life Analogy

A VPC is similar to a company's private office network. You decide the IP
ranges, subnets, routing, and network access rules.

## 2. VPC CIDR Block

A VPC is created with a CIDR block that defines its IPv4 address range.

```text
VPC CIDR: 10.0.0.0/16
```

- `/16`: The first 16 bits identify the network.
- The VPC contains 65,536 IPv4 addresses in the CIDR range.
- Subnets are created from the VPC CIDR and must fit inside it.

## 3. Subnets

A subnet is a smaller IP network inside a VPC. Subnets are commonly used to
separate workloads and network tiers.

```text
VPC: 10.0.0.0/16
|
+-- Public Subnet:   10.0.2.0/24
|
+-- Private Subnet:  10.0.1.0/24
|
+-- Database Subnet: 10.0.3.0/24
```

| Subnet | Typical purpose |
| --- | --- |
| Public subnet (`10.0.2.0/24`) | Has a route to an Internet Gateway and hosts internet-facing resources. |
| Private subnet (`10.0.1.0/24`) | Has no direct route to an Internet Gateway and hosts internal application tiers. |
| Database subnet (`10.0.3.0/24`) | Remains private and isolated from direct internet access. |

## 4. Public vs. Private Subnets

A subnet is not public merely because it has a public-IP setting. Its routing
is the key factor.

- **Public subnet:** Its route table contains a default route such as
	`0.0.0.0/0 -> Internet Gateway`.
- **Private subnet:** It does not have a direct default route to the Internet
	Gateway.
- A private subnet can still have outbound internet access through a NAT
	Gateway.

## 5. Internet Gateway (IGW)

An Internet Gateway is an AWS networking component that connects a VPC to the
internet.

### Public Route Table

```text
0.0.0.0/0 -> Internet Gateway -> Internet
```

### Key Point

An Internet Gateway alone does not make a subnet public. The subnet must use a
route table containing a route to the Internet Gateway.

## 6. Route Tables

A route table is a set of rules that determines where network traffic should
be sent.

| Destination | Target |
| --- | --- |
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | Internet Gateway |

- The `local` route allows communication inside the VPC CIDR.
- `0.0.0.0/0` matches any IPv4 destination and is commonly used as the
	default route.

## 7. NAT Gateway

A NAT Gateway allows resources in private subnets to access the internet
outbound while keeping those resources without public IP addresses.

- A public NAT Gateway is placed in a public subnet.
- The NAT Gateway uses a public IPv4 address, typically an Elastic IP.
- Return traffic is translated back to the private instance by the NAT
	service.
- A NAT Gateway does not make a private EC2 instance directly reachable from
	the internet.

## 8. Elastic IP and NAT Gateway

An Elastic IP is a static public IPv4 address allocated to your AWS account. A
public NAT Gateway uses an Elastic IP so that outbound internet traffic has a
public source address.

---

# Part II: VPC Peering

## 9. VPC Peering

### 9.1 What Is VPC Peering?

VPC peering is a private network connection between two AWS VPCs. It allows
resources in one VPC to communicate with resources in another VPC using
private IP addresses.

```text
			VPC-A                          VPC-B

	 10.0.0.0/16                    20.0.0.0/16

			EC2                             EC2

	 10.0.1.10                       20.0.1.10

			|                                |
			+--------- VPC Peering ----------+
```

### 9.2 Why Do We Need VPC Peering?

By default, two separate VPCs cannot communicate directly.

```text
VPC-A   X   VPC-B
			 no direct communication by default
```

Example scenario:

- VPC-A contains an application server.
- VPC-B contains a database.
- The application needs to access the database.

### Main Purpose

VPC peering allows private communication between two separate VPCs.

### 9.3 How VPC Peering Works

There are three main steps.

#### Step 1: Create a Peering Connection

```text
VPC-A
	|
	v
VPC Peering
	|
	v
VPC-B
```

#### Step 2: Add Routes

| VPC | Route to add |
| --- | --- |
| VPC-A | Destination: `20.0.0.0/16` -> Target: VPC peering connection |
| VPC-B | Destination: `10.0.0.0/16` -> Target: VPC peering connection |

#### Step 3: Allow Traffic

Security groups and network ACLs must allow the required traffic between the
VPCs.
