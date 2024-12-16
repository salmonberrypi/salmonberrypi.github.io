# Background

I love distributed systems and distributed memory computing fabrics.  This project was initially started during the pandemic to focus on something fun, learn, and detract from the madness of the time.  It scratched an itch.  ANd it includes several experiments.  The more I experimented, the more the broken PCIe bus on the Raspberry Pi CM4 became a detraction in addition to the ever-changing supply chain availability and endless redesigns to work around them or cutting functionality outright. Squirrel!

Fast forward to 2024, picking this project up again but refocusing this work after some reflection.  And a couple of key tenets emerged.

## Focus on managing rather than providing

**(Re)use commodity**. Let's review an example. Given the possibility of compactness and integration, an Ethernet switch on board is a great idea.  However, more cost-effective, scalable 3rd party offers offer more flexibility if kept off-board as commodity Ethernet switching. Any perceived compactness or efficiency gains don't offset the investment in jacks and cables, as these switching complexes are complicated in their own right. And maybe for some the PoE option is important.  Many chipsets are not readily available in small quantities or require NDA for documentation or other ODM agreements.  And Microchip's small Gigabit Ethernet chipsets are getting pretty long in the tooth. This isn't worth the trouble.

These are UNIX machines. A **terminal server** is not optional; out-of-band console access is mandatory.  Some projects like [freetserv](https://freetserv.github.io) can be used as inspiration here, although they might benefit from modernization, given that it is almost a decade old.

Monitoring the operational state of nodes is essential.  Being able to **remotely soft reset or power cycle a node** is critical.

Manipulating the **boot state and boot devices** is essential. Raspberry Pi firmware has matured and expanded, and [new boot modes](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#http-boot) help.

Storage can't be an afterthought or blended in with general-purpose compute: **node specialization** is needed, e.g. dual disks for Ceph nodes.

The unit of a node should be **independent of traditional Pi or compute module form factor**. It should be defined by control interfaces and not waste real estate on a specific form factor.  We can print mounts rather than waste PCB on them. More cost-effective and flexible. We can split the CM4/5 footprint in half and only use what we need.

Quantities should always be 2<sup>n</sup> numbers to make scale out and gain **straight forward modularity**.  Not 48. Not 5, 6, or 7. Why? Trade ultimate efficiency for allowing combinations to work better.

## An Idea

The dimensions of a Raspberry Pi Compute module (CM) are 40mm W x 55mm H. We can create board(s) to host the CM. 

We can place a heat exchanger on the stack to remove heat. We can place two stacks back to back around a heat exchanger to do this with two at the saem time.

![horizontal layout](https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/horizontal.png?raw=true | width=100)
![vertical layout](https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/vertical.png?raw=true)

--

&copy; 2024 Christian Kuhtz