# FastThumbsProtocol
The dumbest protocol possible. 

## Synopsis

Trackmania is a fun game, but sometimes, you really want to be playing something other than trackmania while playing trackmania. 

Openplanet offers promise, but it's unfortunately limited - there's no serverless way to communicate between individuals through it. 

Want to make a chess plugin? A competitive snake plugin? A private messaging app that's not really encrypted, but is really weird? 

Move those thumbs, and automate those inputs: The future...is now.

## The Protocol

FTP uses three communication signals for messaging: 

1. Steering input 
2. Brake input
3. Gas input

The steering input value we can read spans from -126 to 126, so - even though this *should* be an 8-bit int we can use, we only get seven bits out of it. 
Then, we'll use the gas input for the last bit, and brake for a clock sync. 

It's unclear what the performance of the plugin will be like in-practice, so we use a concept of hold-and-pulse to facilitate data transfer even in poor conditions. 

Essentially, given a sync interval of 3, click into and hold gas and steer for two frames, and then on the third *also* trigger the clock sync. 

### Bit Layout

Individual packets should be read like this in little-endian notation: 

```
0 0 0 0 0 0 0 0 
| - - - - - | | 
steering      gas

```

These will be collected into "frames" of eight data packets, with a header packet with structure: 

```
1 1 1 1 0 1 0 
| magic | seqnum

```

Thus, an example frame may look like: 

```
1 1 1 1 0 0 1 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 | 1 0 0 1 0 0 1 0 |

```

### From Packets to Donuts

Packets shall be sent in donuts, which can be up to eight frames long. 

The header of a donut will contain: 
* The magic sequence `0 0 0 0`
* The number of frames in the donut. 

Each data packet inside the donut shall be the checksum of the message in the corresponding frame, calculated by the numeric sum of all the bits in the frame (including the header/checksum packet).

### Boxes of Donuts

If you need to send multiple donuts of data, you can package them into a box. 

Boxes are special, because they have both a heading and tailing packet. 

The header looks like: 

```
1 0 0 0 0 0 0 0 | [donut 1] | [donut 2] | ... | 0 0 0 0 0 0 0 1
```

If you send a full box of donuts, you have the option to upgrade to a baker's dozen for the same price as a dozen - replace the header packet with a heart, and then you can use the tailing packet as another data packet. <3 

```
1 1 0 0 0 0 1 1 | [exactly thirtteen donuts]

```
