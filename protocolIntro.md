Disclaimer this is mainly for my own educational purposes, for any corrections just push a request

## What is Can

CAN Bus is the underlying protocol for vehicles produced as of 2008 on. The protocol manages communication between systems within the vehicle such as sensors and ECU. The system is a multi-master system and thus requires two devices "nodes" to be connected by a two wire twisted pair bus of 120 ohms. The data is transmitted with lossless bitwise arbitration to allow priority bits to be sent out over the network for priority systems. In order to implement this it uses "dominant" bits and "recessive" bits where dominant is actually 0 and recessive is a 1, thus allowing for dominant bits to always trump recessive during transmission.

The physical layer of the system uses differential signaling to create a fault tolerant system, essentially instead of just logical on and off voltages. The determination of a on or off bit is from the voltage difference of the static and outstanding voltages.

## Can Frame
There are two formats for CAN 2.0 that is A and B; where B is the extended frame format which supports a 29 bit length identifier instead of the standard 11 bit identifier. Which is just a 11 bit base identifier from standard A and an additional 18 bit extension, thus allowing for CAN 2.0 B systems to support CAN 2.0 A frames.

Four Frame Types:

  * Data frame: a frame containing node data for transmission
  * Remote frame: a frame requesting the transmission of a specific identifier

   Frame indicated by the RTR bit and the DLC contains the length of the message being requested not the one being sent.

  * Error frame: a frame transmitted by any node detecting an error

   Consists of two fiels where the first field is error flags ( 6- 12 dominant/recessive bits)
   Second field is Error delimiter ( 8 bits recessive )

  * Overload frame: a frame to inject a delay between data and/or remote frame


Wikipedia's frame layout is great for this:

| **Field Name**         | **Length (bits)**        | **Purpose**   |
| ---------------------- | ------------------------- | ---------------------------------------------------------------- |
| Start of Frame | 1       | Denotes Start of transmission   |
| Identifier | 11      | Unique identifier, also represents message priority |
| Remote Transmission Request         | 1   | Dominant bit for data frames, recessive for Rrequest frames |
| Identifier extension bit | 1 | Dominant for base frame format |
| Reserved bit | 1 | Must be dominant, but accepted as dominant or recessive |
| Data length Code | 4 | Number of bytes of data (0-8 bytes) |
| Data Field | 0-64 | Data to be transmitted length displayed in DLC above |
| CRC | 15 | [https://en.wikipedia.org/wiki/Cyclic_redundancy_check](Cyclic Redundancy Check) |
| CRC Delimiter | 1 | must be recessive |
| ACK Slot | 1 | Trasmitter sends recessive and receiver can assert a dominant |
| ACK Delimiter | 1 | Must be recessive |
| EOF | 7 | must be all recessive |

For the extended frame see Wikipedia, it just an additional Identifier of 18 bits and moves around some locations.

This 8 bytes of data, is the reason why there is no room for encryption and any device can see the frames allowing for MiTM attacks. As well as send frames pretending to be any device on the system.

More information can be found: [https://en.wikipedia.org/wiki/CAN_bus](Wikipedia Can Bus)

## OBD-II

On Board Diagnostic systems, a standard required on all vehicles. Allows for simply display of critical Diagnostic data, such as when your check engine light is on, with OBD-II you can prove the correct ID to determine the error code. All of the ID's for obd are standard besides a few vehicle/manafacture specific ID's as additional. This allows for a standard method of receiving car data, as CAN systems are manafacture/vehicle specific and have no real standard for ID protocols. Thus requiring reversing to identify ID's. A standard OBD Diagnostic check goes such as this:

1. Request data from specific PID through the port
2. Controller on the network recognizes it is responsible for that PID, reports the value back
3. Scan tool reads response and displays

But OBD-II is written on top of CAN not a protocol on its own, and thus the port allows for arbitrary CAN id messaged to be written to the network.

To determine the protocol used by the OBD-II port you can look at the pin layout on the port.

* J1850 VPW --The connector should have metallic contacts in pins 2, 4, 5, and 16, but not 10.
* ISO 9141-2/KWP2000 --The connector should have metallic contacts in pins 4, 5, 7, 15, and 16.
* J1850 PWM --The connector should have metallic contacts in pins 2, 4, 5, 10, and 16.
* CAN --The connector should have metallic contacts in pins 4, 5, 6, 14 and 16.

For information regarding sending info to OBD-II see [https://en.wikipedia.org/wiki/OBD-II_PIDs](Wikipedia OBD II PIDS).

One example of a request would be to check RPM: (Credit to: [http://www.digitalbond.com/blog/2015/04/13/attacking-canbus-part-1/](Cory Theun))

1. Diagnostic tool sends a message on ID 0x7DF, the standard OBDII query ID. The message contains the number of additional data
   bytes, the “Mode”, and the “PID” of the requested information.
   For querying the RPM the Mode is 1 and the PID is 0x0C. The message would look like:
   0x7DF [8] 02 01 0C 00 00 00 00 00

2. The modules in a vehicle respond to these diagnostics messages. The ID for the response depends on the module but the IDs start
   at 0x8 higher than the standard query ID. Thus, the ECU responding to an RPM request will respond on ID 0x7E8. The message
   contains the number of data bytes, the mode + 0x40, the PID, and the value.
   For responding to RPM the Mode is 0x41, the PID is 0x0C. That message may look like:
   0x7E8 [8] 03 41 0C 09 C4 00 00 00
