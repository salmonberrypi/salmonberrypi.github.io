# Background

I love distributed systems and distributed memory computing fabrics.  This project was initially started during the pandemic to focus on something fun, learn, and detract from the madness of the time.  It scratched an itch.  And it includes several experiments.  The more I experimented, the more the broken PCIe bus on the Raspberry Pi CM4 became a detraction in addition to the ever-changing supply chain availability and endless redesigns to work around them or cutting functionality outright. Squirrel!

Fast forward to 2024, picking this project up again but refocusing this work after some reflection.  And a couple of key tenets emerged.


## Focus on managing rather than providing (December 16, 2024)

**(Re)use commodity**. Let's review an example. Given the possibility of compactness and integration, an Ethernet switch on board is a great idea.  However, more cost-effective, scalable 3rd party products offer more flexibility if kept off-board as commodity Ethernet switching. We can always start another project if we really want to build our own Ethernet switch -- it's not critical path here. Any perceived compactness or efficiency gains don't offset the investment in jacks and cables, as these switching complexes are complicated in their own right. And maybe for some the PoE option is important.  Many chipsets are not readily available in small quantities or require NDA for documentation or other ODM agreements.  And Microchip's small Gigabit Ethernet chipsets are getting pretty long in the tooth. This isn't worth the trouble.

**Lose the PoE**.  While I love PoE for its outward neatness of feeding power to a device over an already required networking cable, this is a lot of dead weight in this design if this is meant to live in a rack.  The PoE injection and the recovery does not come for free in either $ or BOM count or board real estate. And PoE switches demand a premium. It adds quite a bit of complexity as well. And a guesstimate of 5%-10% of the power is lost and converted into heat that we then have to get rid off again. &lt;end rant&gt; Just say no, this isn't a camera on a stick, stay focused.

These are UNIX machines. A **terminal server** is not optional; out-of-band console access is mandatory.  Some projects like [freetserv](https://freetserv.github.io) can be used as inspiration here, although they might benefit from modernization, given that it is almost a decade old.

Monitoring the operational state of nodes is essential.  Being able to **remotely soft reset or power cycle a node** is critical.  Understanding what state the node is in to make sure decisions is a prerequisite.

Manipulating the **boot state and boot devices** is essential. Raspberry Pi firmware has matured and expanded, and [new boot modes](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#http-boot) help.

Storage can't be an afterthought or blended in with general-purpose compute: **node specialization** is needed, e.g. dual disks for Ceph nodes.w

The unit of a node should be **independent of traditional Pi or compute module form factor**. It should be defined by control interfaces and not waste real estate on a specific form factor.  We can print mounts rather than waste PCB on them. More cost-effective and flexible. We can split the CM4/5 footprint in half and only use what we need.

Should quantities always be 2<sup>n</sup> numbers to make scale out and gain **straight forward modularity**?  Not 48. Not 5, 6, or 7. Why? Trade ultimate efficiency for allowing combinations to work better.  12/22/2024: OTOH, this could be very wasteful in scenarios were odd or prime numbers are great for resiliency. This needs more thought.


## An Idea

The dimensions of a [Raspberry Pi Compute module](https://datasheets.raspberrypi.com/cm5/cm5-datasheet.pdf) (CM) are 40mm W x 55mm H. We can create board(s) to host the CM and provide the needed interfaces to connect various umbillicals to, such as Ethernet, serial console, control signals, etc.

Both the "left" and "right" board to board connector on the bottom of the CM actually have very different purposes. Ethernet and GPIO and SD (for a CM-Lite) are on the "left". HDMI, USB, PCIe, etc are all on the "right".  

We don't need both for all use cases. A basic compute node, for example, can live without one side of the board connectors. This also removes alignment issues between both board to board connectors if assembled by hand outside the tolerances. Per [GS-20-0793](https://cdn.amphenol-cs.com/media/wysiwyg/files/documentation/gs-20-0793.pdf), 0.33mm and 0.192mm are the maximum permissible misalignment distances in X (length) and Y (width) of the Bergstak 0.4 BTB connector. Both board to board connectors can effectively be aligned independently by adjusting the location of the board, provided the mounting holes provide adequate clearance (this can probably be bounded, but I haven't done the calculation yet).

If we use different stacked mating heights (e.g. 1.5mm vs 4mm), we can gain board real estate without increasing the footprint unnecessarily over what is needed for networking (user-side) and power (substrate-side).  The main board effectively has a cut out that allows for the aux board to be stacked below it using the longer stacked mating height.

For cooling, we can place a heat sink or fan or both on this board stack to remove heat. Or we can place a heat exchanger on this board stack.

<p align=center><img src="https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/horizontal.png?raw=true"  width="400" alt="horizontal view of CM board on substrate"></p>

If we use a heat exchanger (like the readily available 40mm x 40mm x 12mm variety), we can optimize this design further when we place two of these board stacks back to back around a heat exchanger. A single CM is unlikely to produce enough heat to overwhelm the heat exchanger. And we can accomodate taller components on the board in this arrangement if we offset them to one side.  And irrespective of what we do on the underside for umbillicals, this design will work.

<p align=center><img src="https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/vertical.png?raw=true" width="700" alt="vertical view of single CM and heat exchanger as well as two CMs sandwiching a heat exchanger"></p>

## First Draft (December 21, 2024)

A few days later this is the result:

<p align=center><img src="https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/first%20draft.png?raw=true" width="500" alt="first draft board design"></p>

It is by no means done. The board is more than half routed. But this is sufficient to test the idea and we can build on this further.  The PSU is done, connectors chosen, pass through for the board stack sized and validated (~0.9mm clearance between the main and the mezzanine board below it; anything not a collision is good enough here).  This board represents a connected, fully remote manageable device.

Before I go any further I'm going to try to do the mezzanine board design to see if I can do a quick and dirty validation of the same concept.  If so, I'll spend time on finishing routing the board(s).

## Evolution? Evolution. (December 22, 2024)

In an attempt to smoke this design out, I built a mezzanine template and mocked up various configurations on it.  M.2, multiple M.2, USB, you name it.

<p align=center><img src="https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/mezz-template.png?raw=true" width="500" alt="first draft board design"></p>

It's definitely doable.  But it has a few lessons are learned.


### space constraints

The Hirose/Amphenol low profile board to board connector is being reused.  This gives us two board stacking options and resulting clearances (per the CM datasheets).

|stacked height|clearance|
|---|---|
|1.5mm|0mm|
|4.0mm|2.5mm|

This is sufficient to mate the mezzanine, but as anticipated there is no room on the underside of the main board or on the "top" of the mezzanine where the connector passes through to the main board.  There's roughly 0.9mm clearance between the boards, assuming a PCB of 1.6mm, which is sufficient to prevent electrical contact and reasonably space mechanically.

However, it is pretty wasteful in the end.  Nothing can be placed under the CM on the main board other than traces and vias (unless you carefully navigate around the underside of the CM perhaps, but that's are really fugly hack).  And nothing can be places on the "top" of the mezzanine.

Further, we have to be very careful with what protrudes through the main or the mezzanine board as not to impact the other board. Even if we don't use THT parts and restrict to SMD only, there are parts with locating pins out of plastic or metal or both that quickly invade this gap.  For things like power connectors, THT might be beneficial for routing, plane connection, and stability of the connector and represents a compromise.  It can be solved by cutting away clearance in the main board for avoiding interference but that's also fugly hack #2.

In turn, we're severely limited to what we can do on these boards as-is.


### power constraints

The second CM connector does not pass through power, only ground.  This means we either have to have pins with bottom entry sockets on the mainboard or external jumpers or even both to connect the required power. Fugly hack #3.  Let's design for passing power from board to board from the start.


### signal constraints

While the high speed signals are all on this "second" connector, nothing else is.  That's ok for almost all the scenarios I can conceive of and really only one falls out: you couldn't expose the onboard USB2 connector for the CM this way unless you did away with negotiated CC1/CC2.  Not catastrophic.


### alignment constraints

Disconnecting the alignment issues with separate boards and increased menchanical tolerances to enable slight misalignments does work. But maybe a jig for aligning the non-floating connectors and introducing a separate board to board connector is more efficient for expansion?  

## Exploration

I had sticker shock looking at high speed capable mated connectors for this purposes until I found [Samtec LTH/LSH](https://www.samtec.com/products/matedsetinfo?male=LTH-050-01-G-D-A-K-TR&female=LSH-050-01-G-D-A-K-TR), which is not cheap but more reasonable and readily available. You might think that because they top out at 100 pins that this recreates the problem, but they have board alignment guides that reduce issues.  And there are a few connectors alternatives, here's an example from Samtec:

| Series | Notes |
|---|---|
| [Samtec ERM8/ERF8](https://www.samtec.com/products/matedsetinfo?male=ERM8-100-09.0-L-DV-K&female=ERF8-100-05.0-L-DV-K) | 0.8mm pitch, up to 200 pins, 7mm to 18mm stack height; about 2x the BOM cost of the LTH/LSH in 200 pin version; in effect it is about the same BOM cost since we only need 1x connector.  But they also come in smaller versions with board alignment pins. |
| Samtec ERM6/ERF6 | similar, smaller pitch (0.635mm), up to 120 pins, 5mm stack height |
| Samtec ERM5/ERF5 | similar, even smaller pitch (0.5mm), up to 150 pins, 7 to 12mm stack height|

<p>
Let's be clear.  We need to good reasons to go this way since creating such an interposer creates BOM cost and that has to be understood well and at least answer these questions:

 What do we gain?  Lose?
 Should the entire format change?
 
 ## Rev C

 Simplifying the board by focusing on core use case, addressing some of the concerns above, cleaning up connectors.

### Board Design

Going back to refining what started with [An Idea](#an-idea) above, I took a look at what the panels for JLCPCB look like, and this is the calculation I came up with to understand the consequences of my decisions.

|                       | L/D      | W        |
| --------------------- | -------- | -------- |
| board dimensions      | 118      | 44       |
|                       |          |          |
| panel max size        | 475      | 475      |
|                       |          |          |
| edge width            | 0        | 0        |
|                       |          |          |
| effective panel size  | 475      | 475      |
|                       |          |          |
| max boards            | 4.025424 | 10.79545 |
| rounded boards        | 4        | 10       |
|                       |          |          |
| optimal board size    | 118.75   | 47.5     |
| waste                 | 0.75     | 3.5      |
|                       |          |          |
| 19" rack              | 1100     | 450      |
| plumbing              | 40       | 0        |
| effective board size  | 158.75   | 47.5     |
| boards                | 6.929134 | 9.473684 |
| rounded boards        | 6        | 9        |
| optimal assembly size | 183.3333 | 50       |
| waste                 | 24.58333 | 2.5      |

I enlarged the board slightly to 118x44mm in the redesign for Rev C.This seems like a reasonable compromise.

Much on the board has been refined, although the concept remains. This included going through a bit of work in removing all THT components to prevent any interference issues between the main board and the mezzanine.  This was mostly successful, allthough a cutout where the Ethernet jack lives is needed as there doesn't seem to be a pure SMD with magnetics connector with LEDs built in for me to use; I don't really want to place more LEDs on the board.  Interestingly, the SMD Ethernet jack and the ESD diodes are now trivial and clean to route, whereas before it was a bit of a mess.

### LEDs

All the LEDs have their own MOSFET drivers and a MOSFET switch to turn on or off all LEDs except one.  I'm sourcing all [0603 chip LEDs from W&uuml;rth Elektronik](https://www.digikey.com/en/products/filter/led-indication-discrete/105?s=N4IgjCBcpgbFoDGUBmBDANgZwKYBoQB7KAbRAGYBOAJgBZ4BdAgBwBcoQBlVgJwEsAdgHMQAXwLUADNQAcCEMkjps%2BIqQqVyYcuRBMQbDt37CxBALS15i5bgIA3Aakx21kMrXKTKAVj0t2SBAzEHNqayheAFdVYncQPwZxUMoIyGjY9Qgk5PhoED4AEw4tVICjXkERAlYAT2YcDjQsZFFRIA).

These are the candidates:

| color  | selected | PN (DigiKey)                                                                                             | cost (single) | mcd | Vf [V] | If[mA] | R @ 5V, 50% It |
| ------ | -------- | -------------------------------------------------------------------------------------------------------- | ------------- | --- | ------ | ------ | -------------- |
| Red    | X        | [150060SS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060SS75020/10468226) | 0.33          | 70  | 1.9    | 20     | 160R           |
| Red    |          | [150060RS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060RS75020/10468240) | 0.33          | 100 | 2      | 20     | 150R           |
| Green  | X        | [150060VS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060VS75020/10468264) | 0.33          | 50  | 2      | 20     | 150R           |
| Green  |          | [150060GS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060GS75020/10468246) | 0.33          | 700 | 3.2    | 20     | 91R            |
| Amber  | X        | [150060AS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060AS75020/10468247) | 0.33          | 140 | 2      | 20     | 150R           |
| Yellow | X        | [150060YS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060YS75020/10468235) | 0.33          | 130 | 2      | 20     | 150R           |
| Blue   | X        | [150060BS75020](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060BS75020/10468228) | 0.33          | 150 | 3.2    | 20     | 91R            |

and

| color | selected | PN (DigiKey)                                                                                                                                                                                                                                                                                                                                                    | cost (single) | mcd | Vf [V] | It [mA] | R @ 5V, 50% It |
| ----- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | --- | ------ | ------- | -------------- |
| White | X        | [150060WS75000](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/150060WS75000/25589270?s=N4IgjCBcpgbFoDGUBmBDANgZwKYBoQB7KAbRAGYBOAJgBZ4BdAgBwBcoQBlVgJwEsAdgHMQAXwLkA7AhDJI6bPiKkQtamAAMADg0gmINh279hYgtQ3UtMuQtwFikMlXJhy5PS3aQuvQSPEQAFpaG1RMexAANwFwxQcVWnINSgBWTwNvEDNg6jDIXgBXJUcydIZAoMp8opKVCArA%2BGgQPgATDldqryM-UwJWAE9mHA40LGRRUSA) | 0.22          | 250 | 2.8    | 20      | 110R           |

|color|function|
|---|---|
|red |CM power|
|green|CM activity|
|green|+12V rail ok|
|green|+5V rail ok|
|green|+3V3 rail ok|
|green|PSU enable|
|white|attention/ID|

### current state of the board

<p align=center><img src="https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/revcbare.png?raw=true" width="200" alt="rev c evolution bare"><img src="https://github.com/salmonberrypi/salmonberrypi.github.io/blob/main/revcsmd.png?raw=true" width="200" alt="rev c evolution with SMD components"></p>

To be continued..

--

&copy; 2024 Christian Kuhtz