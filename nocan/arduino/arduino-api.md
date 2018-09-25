## Use

To use the Nocan library in the Arduino API, you should follow the steps outlined in the [NoCAN installation tutorial](http://omzlo.com/articles/installing-nocan#2-3-configuring-arduino)

## API

To use the API, include <nocan.h> in your Arduino sketch.

### Nocan.open

#### Synopsis
```cpp
NocanNodeId Nocan.open()
```

#### Description
This function performs 3 main tasks:

- initialize the I2C interface in master mode, 
- initialize the CAN bus driver (STM32F042)
- request a node id to be attributed to the node (based on a unique 8-byte serial number).

#### Return value
This function returns either:

- a value greater than 0 representing the node id attributed to the node.
- 0 if no node id was assigned to the node
- a value less than 0 in case of a hardware initialization issue

### Nocan.close

#### Synopsis
```cpp

int8_t Nocan.close()
```

#### Description
This function currently does nothing.

### Nocan.lookupChannel

#### Synopsis
```cpp

int8_t Nocan.lookupChannel(const char *channel, NocanChannelId *channel_id) 
```

#### Description

This function looks up the value of `channel_id` corresponding to the string `channel`.

#### Return value

It return 0 in case of success and a negative value in case of failure.

The value of of `channel_id` is updated.

### Nocan.registerChannel

#### Synopsis
```cpp

int8_t Nocan.registerChannel(const char *channel, NocanChannelId *channel_id) 
```

#### Description

This functions registers the channel named `channel` and obtains the corresponding `channel_id`.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

The value of `channel_id` is updated.

### Nocan.unregisterChannel 

#### Synopsis
```cpp

int8_t Nocan.unregisterChannel(uint8_t channel_id) 
```

#### Description

This function unregisters the channel identified by `channel_id`.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.subscribeChannel

#### Synopsis
```cpp

int8_t Nocan.subscribeChannel(NocanChannelId channel_id) 
```

#### Description

This function subscribes the node to the channel identified by `channel_id`.

A node can subscribe to a maximum of 12 channels simultaneously. 

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.lookupAndSubscribeChannel(const char *channel) 

#### Synopsis
```cpp

int8_t Nocan.lookupAndSubscribeChannel(const char *channel, NocanChannelId *channel_id) 
```

#### Description

This function simply a convenience function that combines the following 2 functions in one: `lookupChannel` and `subscribeChannel`.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.unsubscribeChannel

#### Synopsis
```cpp

int8_t Nocan.unsubscribeChannel(NocanChannelId channel_id)
```

#### Description

This function is the opposite of `Nocan.subscribeChannel`: it signifies that the node will stop receiving messages from the channel `channel_id`.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.publishMessage

#### Synopsis
```cpp

int8_t Nocan.publishMessage(NocanChannelID cid, const char *str) 

int8_t Nocan.publishMessage(NocanMessage &msg) 
```

#### Description

The first form of this function publishes a string `str` to the channel identified by `cid`, where `str` is a standard 0-terminated C string.

The alternate form of the function uses the following structure instead. 

```cpp
typedef struct {
    uint8_t node_id;
    uint16_t channel_id;
    uint8_t data_len;
    uint8_t data[64];
} NocanMessage
```
It enables to publish a message `data` of length `data_len` on the channel identified by `channel_id`. 
`data_len` should be less than or equal to 64.
The value of `node_id` is automatically set by the function, based on the value obtained through `Nocan.open()`.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.processMessage

#### Synopsis
```cpp

int8_t Nocan.receiveMessage(NocanMessage *msg) 
```

#### Description
This function allows receiving Nocan messages of up to 64 bytes.

The received message has the following structure:

```cpp
typedef struct {
    uint8_t node_id;
    uint16_t channel_id;
    uint8_t data_len;
    uint8_t data[64];
} NocanMessage
```

Where:

* `node_id` is the address of the node (see `Nocan.open()`).
* `channel_id` is the channel identifier of the received message.
* `data_len` is a number between 0 and 64 describing the length of the message.
* `data` is the actual message.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.getUniqueDeviceIdentifier

#### Synopsis
```cpp
int8_t Nocan.getUniqueDeviceIdentifier(uint8_t *dest)
```
#### Description

This function populates `dest` with the 8-byte serial number of the device.
`dest` must point to an array of at least 8 bytes.

#### Return value

It returns 0 in case of success and a negative value in case of failure.

### Nocan.receivePending

#### Synopsis
```cpp
bool Nocan.receivePending() 
```

#### Description

This function returns 1 if an incoming message is pending, and 0 otherwise.

If a message is pending, it can be read with `Nocan.receiveMessage()`.

#### Return value

1 if a message is pending, 0 otherwise.

### Nocan.transmitPending
```cpp
bool Nocan.transmitPending()
```
#### Description

This function returns 1 if the transmit buffer is full, and 0 otherwise.

#### Return value

1 if a message is pending, 0 otherwise.

### Nocan.led
```cpp
void Nocan.led(bool on)
```

#### Description

This function controls the network led (green) of the CANZERO.
When `on` is true, the led is turned on. 
When `on` is false, the led is turned off. 

Note that this led should not be confused with the user defined led on the CANZERO (orange).

#### Return value

None.

## Examples

### DS18B20 temperature sensor

The code below shows a full sketch of an Omzlo One application that reads data from a DS18B20 temperature sensor and broadcasts it to the channel called "temperature". 

```cpp
#include <OneWire.h>
#include <DallasTemperature.h>
#include <nocan.h>

#define ONE_WIRE_BUS 2

OneWire oneWire(ONE_WIRE_BUS); 
DallasTemperature sensors(&oneWire);
NocanChannelId tid;

void setup() {
  // put your setup code here, to run once:
  sensors.begin();
  for (;;) 
  {
     if (Nocan.open()>=0)
        break;
     delay(1000);
  }
  Nocan.registerChannel("temperature",tid);
}

void loop() {
  // put your main code here, to run repeatedly:
  float temp;
  NocanMessage msg;

  sensors.requestTemperatures();    // this takes some time
  
  temp = sensors.getTempCByIndex(0);
  dtostrf(temp, 6, 1, msg.data);    // transform temp into string
  
  msg.data_len = 6;
  msg.channel_id = tid;
  Nocan.publishMessage(msg);        // publish to the network

  delay(1000); 
}
```

### LCD display

Once the above sketch for the DS18B20 is running on an Omzlo One, you could add a second Omzlo One node with an LCD shield. It's very simple to publish the temperature from the previous example on the LCD by simply using the following sketch.

```cpp
#include <LiquidCrystal.h>
#include <nocan.h>

// initialize the library with the numbers of the interface pins
LiquidCrystal lcd(7, 6, 5, 4, 3, 2);
NocanChannelId cid;

// Initialization
void setup() 
{
  lcd.begin(16, 2);              
  lcd.print("OMZLO ONE SAYS:");  
  lcd.setCursor(0, 1);
  lcd.print("Initializing....");
  
  for (;;)
  {
    if (Nocan.open() < 0)
      delay(1000);
    else
      break;
  }

  Nocan.registerChannel("temperature",cid);
  Nocan.subscribeChannel(cid);
  
  lcd.setCursor(0, 1);           // set the cursor to column 0, line 1
  lcd.print("Waiting.........");
}

// main loop
void loop() 
{
  int i;
  NocanMessage msg;
  int8_t status = Nocan.receiveMessage(&msg);
  
  if (status==0)
  {
      if (msg.data_len>16) msg.data_len=16;    
      while (msg.data_len<16) msg.data[msg.data_len++]=' ';
      msg.data[msg.data_len]=0;
      
      lcd.setCursor(0, 1);
      lcd.print((const char *)msg.data);
  }
}
```
