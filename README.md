# Background

This project was initially started during the pandemic to focus on something fun, learn, and detract from the madness of the time.  And it includes several experiments.  The more I experimented, the more the broken PCIe bus on the Raspberry Pi CM4 became a detraction in addition to the ever-changing supply chain availability and endless redesigns to work around them. Squirrel!

Fast forward to 2024, picking this project up again but refocusing this work after some reflection.  And a couple of key tenets emerged.

## Focus on managing rather than providing

An Ethernet switch on board seems like a great idea.  However, there are more cost effective, more scalable 3rd party offers that offer more flexibility if kept off-board. 

A terminal server is not really optional, out-of-band access to console is necessary.

Monitoring operational state of nodes is important.  Being able to soft reset or power cycle a node is important.

Manipulating the boot state and boot devices is important.  [New boot modes](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#http-boot) help.
