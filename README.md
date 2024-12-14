# Background

This project was initially started during the pandemic to focus on something fun, learn, and detract from the madness of the time.  And it includes several experiments.  The more I experimented, the more the broken PCIe bus on the Raspberry Pi CM4 became a detraction in addition to the ever-changing supply chain availability and endless redesigns to work around them. Squirrel!

Fast forward to 2024, picking this project up again but refocusing this work after some reflection.  And a couple of key tenets emerged.

## Focus on managing rather than providing

Given the possibility of compactness and integration, an Ethernet switch on board is a great idea.  However, there are more cost-effective, scalable 3rd party offers that offer more flexibility if kept off-board. Any perceived compactness or efficiency gains don't offset the investment in jacks and cables, as these switching complexes are complicated in their own right.  And many chipsets from Asia are not readily available in small quantities either or require NDA for documentation or other ODM agreements.

These are UNIX machines. A terminal server is not optional; out-of-band console access is mandatory.  There are projects like [freetserv](https://freetserv.github.io) that can be used as inspiration here, although they might benefit from modernization, given that it is almost a decade old.

Monitoring the operational state of nodes is important.  Being able to soft reset or power cycle a node is important.

Manipulating the boot state and boot devices is important.  [New boot modes](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#http-boot) help.
