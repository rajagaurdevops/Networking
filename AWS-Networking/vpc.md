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

---

# Part III: Transit Gateway

## 10. Transit Gateway

### 10.1 What Is a Transit Gateway?

A Transit Gateway (TGW) is a regional AWS networking hub that connects
multiple VPCs and on-premises networks through a single gateway. Instead of
connecting networks to each other directly, each network connects once to the
Transit Gateway, which handles routing between them.

### Real-Life Analogy

A Transit Gateway is like an airport hub. Instead of every city needing a
direct flight to every other city, each city flies to the hub, and the hub
routes passengers onward.

### 10.2 Why Use a Transit Gateway?

VPC peering connects exactly two VPCs at a time. As the number of VPCs grows,
connecting every VPC to every other VPC requires a separate peering connection
for each pair. This is called a full-mesh topology and becomes difficult to
manage.

```text
VPC Peering (full mesh)              Transit Gateway (hub-and-spoke)

	 A---B                                   A     B
	 |\ /|                                    \   /
	 | X |                                      TGW
	 |/ \|                                    /   \
	 C---D                                   C     D

6 peering connections                 4 attachments
for 4 VPCs (grows fast)                (scales linearly)
```

- Peering connections grow quadratically as VPCs are added; Transit Gateway
	attachments grow linearly.
- A Transit Gateway can also connect VPN connections and AWS Direct Connect,
	not only VPCs.
- Routing is centrally controlled through Transit Gateway route tables instead
	of many separate VPC route tables.

### Main Purpose

Transit Gateway simplifies large-scale network architectures by replacing many
point-to-point connections with a single central hub.

### 10.3 How Transit Gateway Works

Transit Gateway connectivity uses three core concepts: attachments, route
tables, and association or propagation.

#### Step 1: Create the Transit Gateway

A Transit Gateway is created in a single AWS Region and acts as the central
routing hub for that region.

#### Step 2: Create Attachments

Each network that needs to connect through the Transit Gateway is added as an
attachment:

- **VPC attachment:** Connects a VPC through a subnet in each Availability Zone
	being used.
- **VPN attachment:** Connects an on-premises network over a Site-to-Site VPN.
- **Direct Connect Gateway attachment:** Connects an on-premises network over
	a dedicated connection.
- **Peering attachment:** Connects two Transit Gateways, including Transit
	Gateways in different Regions.

```text
			 VPC-A          VPC-B          VPC-C
					|              |              |
					+------ Transit Gateway ------+
												 |
									 On-Premises
									(VPN / Direct Connect)
```

#### Step 3: Association and Propagation

Each attachment is linked to a Transit Gateway route table using two separate
actions:

| Action | What it does |
| --- | --- |
| **Association** | Determines which Transit Gateway route table an attachment uses to send traffic. |
| **Propagation** | Automatically adds an attachment's routes into a Transit Gateway route table so other attachments can reach it. |

### Key Point

Association controls where an attachment looks up routes. Propagation controls
whether an attachment's own routes appear in that route table.

Using separate route tables for different attachments allows segmented,
hub-and-spoke traffic control. For example, a shared-services VPC can be made
reachable by all networks while other networks remain isolated from one
another.

#### Step 4: Update VPC Route Tables

Each attached VPC's own route table must also send traffic destined for other
networks to the Transit Gateway.

| Destination | Target |
| --- | --- |
| `10.0.0.0/16` | `local` |
| `20.0.0.0/16` (VPC-B) | Transit Gateway |
| `30.0.0.0/16` (VPC-C) | Transit Gateway |

### 10.4 Transit Gateway vs. VPC Peering

| Aspect | Comparison |
| --- | --- |
| Topology | Peering is point-to-point and commonly forms a mesh; Transit Gateway is hub-and-spoke. |
| Scaling | Peering connections multiply quickly as VPCs are added; Transit Gateway adds one attachment per network. |
| On-premises access | Peering connects VPCs only; Transit Gateway also supports VPN and Direct Connect. |
| Routing control | Peering relies on each VPC's route table; Transit Gateway offers centralized route tables that can segment traffic. |
| Transitive routing | Peering is not transitive: A-B and B-C do not automatically allow A-C. Transit Gateway supports routing between attachments when the route tables allow it. |

### 10.5 Key Concepts Summary

- **Transit Gateway:** A regional hub that interconnects VPCs, VPNs, Direct
	Connect, and other supported networks.
- **Attachment:** A connection between a network and the Transit Gateway. The
	network can be a VPC, VPN, Direct Connect connection, or another Transit
	Gateway.
- **Transit Gateway route table:** Controls how traffic is routed between
	attachments.
- **Association:** Links an attachment to the route table it uses for routing
	decisions.
- **Propagation:** Automatically populates a route table with an attachment's
	routes.
