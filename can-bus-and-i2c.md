# CAN Bus

**summary:** this is a protocol, something used to let different devices on our rover talk to each other. important features are that it uses differential signalling (giving robustness) and has a multi-master architecture (providing flexibility). it is a broadcast protocol, meaning that all devices on the bus receive all messages, and it is up to the devices to decide if they should act on the message or not.

- control area network
- two-wire protocol, that uses differential signalling
- wires are called CAN H and CAN L, for high and low voltage
- multi-master
- broadcast protocol
- linear bus topology
- includes error detection and acknowledgement system
- half-duplex

## the can frame
a frame is one message sent on the bus. looking at its composition is instructive to understand how the bus works.

![CAN Frame](https://upload.wikimedia.org/wikipedia/commons/thumb/5/54/CAN-bus-frame-with-stuff-bit-and-correct-CRC.png/880px-CAN-bus-frame-with-stuff-bit-and-correct-CRC.png)

- **start bit:** this is a dominant 0, to signal the start of a frame
- **arbitration field:** this is the identifier of the message, and is used to determine which message has priority. a higher priority (more important) means a lower number. this is a 11 bit number, but can be extended to 29 bits (see extended frame format)
    - **RTR (request to transmit) bit:** this is a dominant 0, and is used to indicate if the message is a data frame (0) or a request frame (1). a remote frame is used to request data from a device, and is not used in our application
- **control field:** this is a 6 bit field, and contains the data length code (DLC). this is the number of bytes in the data field
    - **identifier extension (IDE) bit:** this is a recessive 1, and is used to indicate if the frame is a standard frame (0) or an extended frame (1). a standard frame uses 11 bit identifiers, and an extended frame uses 29 bit identifiers
    - **r0 bit:** this is a reserved bit, and is not used. it may be used in the future
    - **data length code (DLC):** this is the number of bytes in the data field (0-8)
- **data field:** this is the data being sent, and is 0-8 bytes long. 0 in request frames, since they don't contain data
- **crc field:** this is a 15 bit cyclic redundancy check (CRC), and is used to check for errors in the frame. the standard does not specify a way to calculate the CRC.
- **acknowledgement (ack) bit:** used to conform if the receiver received the message correctly. the sender sends a recessive 1; if the receiver received the message correctly, it sends a dominant 0 (override)
    - **ack delimiter (ack del):** this is a recessive 1, and is used to signal the end of the ack bit
- **end of frame (eof):** 7 sequential recessive 1s, used to signal the end of the frame

## other features
- **bus arbitration:** if two devices try to send a message at the same time, the device with the lower identifier/higher priority wins. this happens because by default the bus is pulled to dominant bits. so if there is a collision, the bus will default to the dominant bit. since each device (including the transmitter) is watching the bus, the transmitter shall see that stop transmitting.
- **error detection:**
  - the crc field is used to detect errors in the frame. if the crc is incorrect, the frame is discarded. the crc is calculated using the data field, the control field, and the identifier field.
  - a transmitter also monitors its own transmission. if a data bit is written onto the bus and its opposite is read, an error is generated.
- **multi-master:** any device can send a message at any time, and the bus will handle it. this is because of the bus arbitration feature.
- **differential signalling:** the bus uses differential signalling, which means that the signal is sent on two wires, and the difference between the two signals is used to determine the value. this is used to provide robustness, since noise will affect both wires equally, and the difference will remain the same.
- **bit stuffing:** if there are 5 sequential bits of the same value, a bit of the opposite value is inserted. this is used to provide clock synchronization, and to prevent long sequences of the same value, which could cause the bus to lose synchronization.
    - note that if bit stuffing results in 6 sequential bits of the same value, they will also be stuffed recursively.
    - bit stuffing does not apply after the last CRC bit.
- **types of frames**
    - **error frames:** if a device detects an error in a frame, it will send an error frame. this is 6 consecutive dominant bits, causing all other nodes to raise error frames as well. then the transmitter will resend its message (see error handling)
    - **overload frame:** if a device needs to send a message, but the bus is busy, it will send an overload frame. this is 6 consecutive recessive bits, causing all other nodes to raise error frames as well. then the transmitter will resend its message
    - note that overload frames are sent between frames, and error frames are sent during frames
- **termination:** the bus is terminated at each end with resistors
- **synchronous:** CAN bus is synchronous in the sense that all nodes must be synchronized to the same clock. however, the bus is asynchronous in the sense that there is no clock signal sent on the bus. instead, the clock is generated by each node, and is synchronized to the bus using the bit stuffing feature.
- **filter:** to filter out a message, we use a mask. the mask is a 0 if we want to check the bit, and a 1 if we don't care about the bit. we then OR the mask with the identifier.
- **uneven power:** in the event that our devices have different power sources, we must provide a CAN_GND (common ground) line, since CAN H and CAN L must be referenced to the same ground. moreover, the difference between CAN H and CAN L must be between 2.5V and 3.5V
- **bit timing:** the nominal bit timing is the time it takes to send one bit (duration from start of sync segment to end of phase 2). the bit timing is determined by the bit rate, which is the number of bits sent per second.
  - synchronization segment: time of rising edge, used to sync clocks
  - propagation segment: time of rising edge to start of phase 1, used to compensate for physical delays
  - phase 1 and phase 2: phase buffer segments, used for resynchronization
  - the bit is sampled exactly when phase 1 ends and phase 2 starts
  - one bit is divided in to 8 time quanta (tq)
  - sync is 1 tq, rest are programmable.

- **error handling**:
  - each node starts in *error active mode* (normal mode). in this mode, if errors are detected, it will transmit active error flags.
  - each node has a receive error counter (REC) and a transmit error counter (TEC). if there is an error while receiving a message, REC is incremented. if there is an error while transmitting a message, TEC is incremented.
  - increments by 8 on transmit error, and 1 on receive error. decrements by 1 on successful transmission.
  - if REC > 127 or TEC > 127, the node goes into *error passive mode*. this means 16 consecutive frames were corrupted. in this mode the node will transmit passive error flags.
  - if REC > 255 or TEC > 255, the node goes into *bus off mode*. this means 256 consecutive frames were corrupted. in this mode the node will not transmit anything, and will not acknowledge any messages.
  - a passive error flag comprises 6 consecutive recessive bits, so it does not disrupt traffic.
  - an active error flag comprises 6 consecutive dominant bits followed by 8 recessive bits, so it disrupts traffic.
  ![error handling graph](https://cdn.shopify.com/s/files/1/0579/8032/1980/files/CAN-error-handling-states-error-active-passive-bus-off.svg)

## implementation
- typically, to implement you would use a transceiver. this converts the TTL signals from the microcontroller to the differential signals used on the bus.
- additionally, the transceiver will provide protection against voltage spikes, and will provide a way to detect if the bus is busy or not. it also often handles the termination
- often when using Arduino, the transceiver is connected to the SPI pins, and the microcontroller communicates with the transceiver using SPI. note that Arduino does not have any support for CAN, so you will need to use a library, which "simulates" CAN bus using bit-banging.

## sources
- TI Precision Labs videos for much of the concepts: https://www.ti.com/video/series/ti-precision-labs-can-lin-sbc.html
- specifications:
  - bosch: http://esd.cs.ucr.edu/webres/can20.pdf
  - FTDI: https://ftdichip.com/wp-content/uploads/2020/08/TN_156-What-is-CAN.pdf
- CSS electronics for features of the bus, at a high-level: https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial
- mask and filter: https://www.cnblogs.com/shangdawei/p/4716860.html
- different power sources:
  - https://electronics.stackexchange.com/questions/518827/can-bus-nodes-powered-by-different-potential-sources
  - https://electronics.stackexchange.com/questions/476129/can-over-12v-dc-possible
- bit-level error handling: http://www.can-wiki.info/doku.php?id=can_faq:can_faq_erors
- bit-timing: http://www.bittiming.can-wiki.info/
- [FURTHER READING] high-level error handling: https://www.csselectronics.com/pages/can-bus-errors-intro-tutorial

# I2C
- inter-integrated circuit
- two-wire protocol, uses TTL signalling
- wires are called SDA and SCL, for serial data and serial clock
- master-slave
- synchronous
- half-duplex
- linear bus topology
- two pull-up resistors are used to pull the lines high

## I2C frame
![I2C Frame](https://www.circuitbasics.com/wp-content/uploads/2016/01/Introduction-to-I2C-Message-Frame-and-Bit-2.png)

- the master pulls the SDA line low to signal the start of a frame
- the master then sends the address of the slave it wants to communicate with
- if the slave pulls low to acknowledge the address, the master sends the data, with a similar acknowledgement system
- towards the end of a frame (either data or address), the master sends one final high clock pulse, and the slave pulls the SDA line high to acknowledge the end of the frame
- each bit is sent during a high clock pulse
- if the address is longer than 7 bits, we right-shift the address to accommodate the extra bits

## sources
- NTS Press video for concepts: https://www.youtube.com/watch?v=7CgNF78pYQM
- TI Precision labs: https://www.ti.com/video/series/ti-precision-labs-i2c.html