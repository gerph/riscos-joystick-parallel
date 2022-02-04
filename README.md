# Parallel Port Joystick

This repository provides a simple driver for a 9-pin joystick that can
be connected to a parallel port. I built this for the A5000 so that I
could play Elite with the same joystick I'd used on the BBC.

The module provides only the 8bit interface, but as a switched joystick,
this is all that is necessary.

## Building the driver

The driver builds automatically on pushes using the RISC OS Build service.

To manually build the module, push the code to the server using the `riscos-build-online` tool. For example:

```
curl -s -L -o riscos-build-online https://github.com/gerph/robuild-client/releases/download/v0.05/riscos-build-online && chmod +x riscos-build-online

zip -q9r /tmp/source-archive.zip * .robuild.yaml

./riscos-build-online -i /tmp/source-archive.zip -a off -t 60 -o archive
```

This will produce a binary called `archive,a91` which is a RISC OS zip archive containing the module.

## Constructing the "Interface"

This is not really an interface, just a one-to-one relationship for the
connectors.

You require a 9-pin joystick like this :

```
1 2 3 4 5
 6 7 8 9
```

and a printer lead. The printer lead must have the printer end removed so
that all the wires are accessable.

You may need to open the computer end to note the colours of the leads so
that you can connect the correct wire to each hole.

At the computer end the first wire on my lead is brown, then red, orange,
pink. The wires are labelled (with my colours) :

| Pin | Colour          | Name           | Direction
| --- | --------------- | -------------- | ---------
| 1   | Brown           | Strobe         | I/O
| 2   | Red             | D0             | I/O
| 3   | Orange          | D1             | I/O
| 4   | Pink            | D2             | I/O
| 5   | Yellow          | D3             | I/O
| 6   | Dark Green      | D4             | I/O
| 7   | Cyan            | D5             | I/O
| 8   | Blue            | D6             | I/O
| 9   | Blue (Stripe)   | D7             | I/O
| 10  | Purple          | Ack            | I
| 11  | Grey            | Busy           | I
| 12  | White           | Paper Error    | I
| 13  | Black           | Select         | I
| 14  | Brown (Stripe)  | Auto feed      | O
| 15  | Red (Stripe)    | Error          | I
| 16  | Orange (Stripe) | Reset          | O
| 17  | Green (Stripe)  | Select In      | O
| 18  | Black (Stripe)  | Ground         | N/A

Directions relative to computer.

Connect the following :

| Computer  | 9-pin
| --------- | ------
|    2      |   2
|    3      |   3
|    4      |   4
|    5      |   5
|    16     |   7
|    6      |   9

So, if you have the same lead as me the lead should read (left to right) :

* Nothing, Red, Orange, Pink, Yellow
* Nothing, Orange (Stripe), Nothing, Green

And connect 12, 13, and 15 to 14 in the Computer lead. The reason for this is
to allow some simple testing of the parallel port to see if the Joystick is
connected or not. What should happen is that when the AutoFeed line is
toggled, so should the PaperError, Select, and Error return lines be toggled.

I have used bits D0-D4 to mark up, down, left, right and fire, and used Reset
to provide the power for the joystick. Therefore, reset must always be kept
high. As the parallel port retains settings and the joystick never clears set
bits after each read the joystick should be cleared. To be exact, the value
read will be that of the directions moved /since the last clear/ and it is
possible that all directions could be set. To alleviate this the read code
might be placed on an interrupt, but I'm not bothered, as most games read the
joystick regularly.

