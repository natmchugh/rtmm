# How to copy, read and write Paxton fobs and cards with an RFIDler

Paxton fobs and readers are popular in the UK especially the Net2 system where the fobs look like this with a blue ring:
![Paxton Fob](https://gist.githubusercontent.com/natmchugh/18e82761dbce52fa284c87c190dc926f/raw/fob.jpg "Paxton Fob")

Paxton readers often look like this:

![Paxton Reader](https://gist.githubusercontent.com/natmchugh/18e82761dbce52fa284c87c190dc926f/raw/reader.jpg "Paxton Reader")


This guide covers how to read the data from an existing Paxton fob or card and also how to write data to a fob or card. If the original fob or card has been authorised with the reader the new fob or card will be seen by the reader as the same tag, effectively a clone. You can copy cards to fobs and fobs to cards. Hereafter both fobs and cards will be referred to as tags.

These Paxton tags use hitag2 technology and so can be copied to any hitag2 cards, fobs or other tag form factor.

## Equipment used
* An RFIDLer, available here from one of the tools authors http://rfidiot.org/
* Enamelled copper wire, I used 33swg or ~0.25 mm
* Some hitag2 tags See notes [here](/natmchugh/18e82761dbce52fa284c87c190dc926f#getting-hold-of-hitag2-tags). 
* (optional) a soldering iron but you my be able to get away without one

## Antennas
The RFIDler comes with a coil antenna that works well for reading cards and sniffing readers. It does not however work well with fobs. In order to read and write to a Paxton fob I had to wind my own antenna. This is covered further [here](/natmchugh/18e82761dbce52fa284c87c190dc926f#creating-a-diy-antenna-for-paxton-fobs).


## Connecting to your RFIDler

This is done on the command line via a serial communication program. I used minicom which is available on a mac via homebrew or on Linux via a package manager. On windows PuTTY has this functionality.

You need to find out what device the RFIDler was mounted as when you plugged it in via usb. In my case it was at `/dev/tty.usbmodem092426B340191` and I found it by looking for the most recent mounted device in /dev.

So to connect it I used the command
```
minicom -D /dev/tty.usbmodem092426B340191 -b 115200
```

## Reading a tag

Once you have connected to your RFIDLer you need to set the config to hitag2 tags
```
RFIDler> set tag hitag2
OK

*HITAG2> save
OK

HITAG2>
```
Next you can try reading the tag serial number. This is a read only number and cannot be changed but it is not used in access identification. It can be read without knowing the password for the tag and is also known as page 0.

```
HITAG2> reader
12345678
12345678
12345678
```
Hopefully you should see the tag serial number being repeated continuously. This means there is a good strong connection between the tag and antenna. If you don't see the number adjust the tag and coil position until you do.

The next step is to login to the tag by supplying the password. There is a common password for these Paxton tags.

```
HITAG2> login BDF5E846
06F907C2
```
A response like the one above means the login was successful. It shows the config bit and tag password held on page 2 of the tag. It is the same for all Paxton tags I tested. If instead you see "Login failed!" try again with a different position or no password (which will use the default blank tag password).

Once you have logged in successfully you can now read the 8 pages of data held on the tag. But first you will need to set the VTag type to hitag2 as well for reasons I'm unclear on.

The VTag is a virtual tag representation held on the RFIDler and is where you need to load data before writing it out to a new tag.

```
HITAG2> read 0
VTag not compatible!

HITAG2> set vtag hitag2
OK

*HITAG2> save
OK

HITAG2> read 0
0: 12345678

*HITAG2> read 1
1: BDF5E846

HITAG2> read 2
2: 06F907C2
…
```

Repeat the read commands all the way up to page 7.

Once you have this data you have all the info you need to clone the tag. The important pages are 4-7 these contain the data which the reader identifies for access.

## Can I Emulate a Paxton tag?
Not currently with an RFIDler. The data flow is more complicated than some other tags involving a back and forth of commands the reader could send. The chips in hitag2 tags handle these commands really well. So why not just use one of those i.e. clone to a hitag2 tag.

## Writing a Tag

Despite not being able to emulate hitag2, to write a tag you need to load your tag data into the virtual tag or VTag.
```
HITAG2> VWRITE 1 BDF5E846PAGE2DATPAGE3DATPAGE4DATPAGE5DATPAGE6DATPAGE7DAT
BDF5E846PAGE2DATPAGE3DATPAGE4DATPAGE5DATPAGE6DATPAGE7DAT

```

Where PAGE1DAT etc is the 8 hex digits you got by reading the original tag.

Once you have written to the VTag you can check the contents by issuing the vtag command

```
HITAG2> vtag
              Type: HITAG2
         Emulating: NONE
           Raw UID:
               UID:

     PWD Block (1): BDF5E846    ...F

     Key Block (2): PAGE2DAT    ..-.

  Config Block (3): PAGE3DAT

        Page 1 & 2: 0 = Read / Write
            Page 3: 0 = Read / Write
        Page 4 & 5: 0 = Read / Write
        Page 6 & 7: 0 = Read / Write
          Security: 0 = Password
              Mode: 0 = Public Mode B
        Modulation: 0 = Manchester

     PWD Block (3): GE3DAT      .=.

              Data:
                 0: 12345678
                 1: BDF5E846
                 2: PAGE2DAT
                 3: PAGE3DAT
                 4: PAGE4DAT
                 5: PAGE5DAT
                 6: PAGE6DAT
                 7: PAGE7DAT


```

Once you are happy with the data as shown in the VTAG you can then clone it onto another tag

```
CLONE <BDF5E846|4D494B52>
1: BDF5E846
2: PAGE2DAT
4: PAGE4DAT
5: PAGE5DAT
6: PAGE6DAT
7: PAGE7DAT

```

The password used to clone the tag at the end depends on where you got the tag from. A new blank hitag2 tag should have the password 4D494B52. If a tag has been set up for Paxton readers previously it will have the password BDF5E846.

The new tag should be the same as the old tags as far as the reader is concerned.

## Creating a DIY antenna for Paxton fobs

I was able to read and write genuine Paxton fobs by creating a coil antenna that allowed the fob to be placed inside. The original coil has an inductance at 374µH. 

![Original Coil](https://gist.githubusercontent.com/natmchugh/18e82761dbce52fa284c87c190dc926f/raw/original_coil.png "Original Coil")

With trial and error I created a similar inductance coil with diameter of 2.5cm roughly 140 turns.

My top tip / life hack for winding the antenna would be to use super glue to get the initial loops on and secure them at the correct height and then electrical insulation tape to protect the coil and keep it in place.

![Antenna](https://gist.githubusercontent.com/natmchugh/18e82761dbce52fa284c87c190dc926f/raw/paxton_antenna.jpg "Antenna")

To test your antenna, putting a tag in and viewing the data as a graph vs time can help in fine tuning your antenna design.

To do this there is a python wrapper for rfidler that can call it via the api and plot the results

```
cd python
python rfidler.py /dev/tty.usbmodem092426B340191 'set tag hitag2' 'uid' plot 1500
```

This sets the tag type as hitag2 and then asks the tag for its serial number before plotting the results.

![Plot](https://gist.githubusercontent.com/natmchugh/18e82761dbce52fa284c87c190dc926f/raw/plot.png "Plot")

Once you start getting good pulses that can be easily distinguished from the background noise you can attempt to use the antenna to read a fob.

## Getting hold of hitag2 tags

This is actually one of the harder steps especially with hitag2 cards in smaller quantities. A lot of what are advertised as  hitag2 cards when they arrive turn out to be a different card type such as EM4100. In the course of this research I ended up with a load of tags of many types.

Paxton fobs can be widely picked up in packs of 10 for about £30 but this is much more than a hitag2 card should cost which is less than half that.

If you want to give this guide a go I have a small quantity of genuine Paxton fobs I purchased and would be willing to sell individually for around cost price. If you would like one of these contact me via github. Also happy to clone tags for research.