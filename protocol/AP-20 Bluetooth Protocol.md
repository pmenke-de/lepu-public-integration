# 1. Overview
This document describe how the Master devices connect and communicate with AP-20 devices via BLE(Bluetooth Low Energy). We recommand you take a look at [Ble Basics](./../ble_basics/README.md).
# 2. Protocol
## 2.1 Service & Characteristic
- Only 1 service is used
  - UUID: `0000FFB0-0000-1000-8000-00805f9b34fb`
- And 2 characteristics, 1 for read and 1 for write.
  - Read Characteristic UUID: `0000FFB2-0000-1000-8000-00805f9b34fb`
  - Write Characteristic UUID: `0000FFB2-0000-1000-8000-00805f9b34fb`
## 2.2 Data pack format
|    | Head | Token | content length | content | CRC8 |
| ---- | ---- | ---- | ---- | ---- | ---- |
| **Size** (bytes) | 2 | 1 | 1 | N | 1 |
| **Description** | 0xAA 0x55 |  | N+1 |  |  |

### 2.2.1 Token
| token | description |
| ---- | ---- |
| 0x0f | oximeter token |
| 0x2d | breath token |
| 0xf0 | universal token |

### 2.2.2 Content
`Content` part contains 1 byte `Data Type` and `Message`. `message` can be null.

### 2.2.3 Data pack list
See charpter 3

# 3. Command
## 3.1 Device Info
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Device Info | Send | 0xf0 | 0x81 | null | |
| | Receive | 0xf0 | 0x01 | DeviceInfo |
```c++
typedef struct
{
  short software_version; //2 byte BCD encode. i.e. 0x21,0x03, the version is v2.1.0.3
  unsigned char hardware_version; // 1 byte
  char name[n]; //n bytes, ASCII encode. i.e. "AP-20", the bytes are "0x41,0x50,0x2D,0x32,0x30"
} DeviceInfo
```
```
example:
SEND: 0xAA 55 F0 02 81 19
RECEIVE: 0xAA 55 F0 0A 01 21 03 13 41 50 2D 32 30 62
```
## 3.2 Device SN
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Device SN | Send | 0xf0 | 0x82 | null | |
| | Receive | 0xf0 | 0x02 | SN |
```c++
typedef struct
{
  char sn[n]; //n bytes, ASCII encode. i.e. the SN is "28ABZD", the bytes are "0x32 0x38 0x41 0x42 0x5A 0x44"
} SN
```
```
example:
SEND: 0xAA 55 F0 02 82 FB
RECEIVE: 0xAA 55 F0 08 02 32 38 41 42 5A 44 2E
```
## 3.3 Battery
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Battery | Send | 0xf0 | 0x83 | null | |
| | Receive | 0xf0 | 0x03 | Battery |
```c++
typedef struct
{
  char battery; //value: 0-3; 0: battery low; 3: battery full
} Battery
```
```
example:
SEND: 0xAA 55 F0 02 83 A5
RECEIVE: 0xAA 55 F0 03 03 {Battery} {CRC}
```
## 3.4 Get Backlight
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Get Backlight | Send | 0xf0 | 0x84 | null | |
| | Receive | 0xf0 | 0x04 | Backlight |
```c++
typedef struct
{
  char backlight; //value: 0-5; 0: dark; 5: light
} Backlight
```
```
example:
SEND: 0xAA 55 F0 02 84 26
RECEIVE: 0xAA 55 F0 03 04 {Backlight} {CRC}
```
## 3.5 Set Backlight
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Set Backlight | Send | 0xf0 | 0x85 | Backlight | |
| | Receive | 0xf0 | 0x05 | Result |
```c++
typedef struct
{
  char backlight; //value: 0-5; 0: dark; 5: light
} Backlight
```
```c++
typedef struct
{
  char result; // 0x00: failed; 0x01: success
} Result
```
```
example:
SEND: 0xAA 55 F0 03 85 {Backlight} {CRC}
RECEIVE: 0xAA 55 F0 03 05 {Result} {CRC}
```
## 3.6 Set Time
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Set Time | Send | 0x0f | 0x87 | Time | |
| | Receive | 0x0f | 0x07 | Result |
```c++
typedef struct
{
  unsigned short year;  // big endian
  unsigned char month;
  unsigned char day;
  unsigned char hour;
  unsigned char minute;
  unsigned char second;
} Time
```
```c++
typedef struct
{
  char result; // 0x00: failed; 0x01: success
} Result
```
```
example:
SEND: 0xAA 55 0F 09 87 07 E0 02 0E 09 0F 03 A5 //set time to 2016/02/14 09:15:03
RECEIVE: 0xAA 55 0F 03 07 01 9A
```
## 3.7 Get Oxi alert Config
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Get Oxi alert Config | Send | 0x0f | 0x91 | ConfigType | |
| | Receive | 0x0f | 0x11 | ConfigParam |
```c++
typedef struct
{
  char type;
} ConfigType
```
```c++
typedef struct
{
  char type;
  char value;
} ConfigParam
```
| ConfigType | Config | ConfigParam |
| ---- | ---- | ---- |
| 0x01 | Alert switch | 0x00: off; 0x01: on |
| 0x02 | Spo2 low threshold | 85%-99% |
| 0x03 | pulse rate low threshold | 30-99 bpm |
| 0x04 | pulse rate high threshold | 100-250 bpm |
| 0x05 | pulse beep | 0x00: off; 0x01: on |
```
example:
SEND: 0xAA 55 0F 03 91 02 11  // get spo2 low alert threshold setting
RECEIVE: 0xAA 55 0F 04 11 02 56 21  // spo2 low alert is 86%
```
## 3.8 Set Oxi alert Config
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Set Oxi alert Config | Send | 0x0f | 0x92 | ConfigParam | |
| | Receive | 0x0f | 0x12 | Result |
```c++
typedef struct
{
  char type;
  char value;
} ConfigParam
```
```c++
typedef struct
{
  char result; // 0x00: failed; 0x01: success
} Result
```
| ConfigType | Config | ConfigParam |
| ---- | ---- | ---- |
| 0x01 | Alert switch | 0x00: off; 0x01: on |
| 0x02 | Spo2 low threshold | 85%-99% |
| 0x03 | pulse rate low threshold | 30-99 bpm |
| 0x04 | pulse rate high threshold | 100-250 bpm |
| 0x05 | pulse beep | 0x00: off; 0x01: on |
```
example:
SEND: 0xAA 55 0F 04 92 01 01 AA  // turn on oxy alert
RECEIVE: 0xAA 55 0F 04 12 01 01 C8   // success
```
## 3.9 Enable Oxi Param Notify
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Enable oxy param data | Send | 0x0f | 0x84 | Switch | |
| | Receive | 0x0f | 0x04 | Frequency |
```c++
typedef struct
{
  char switch; //0x00: turn off oxi param notify; 0x01 turn on.
} Switch
```
```c++
typedef struct
{
  char frequency; // oxi param data pack frequenct
} Frequency
```
## 3.10 Enable Oxi Wave Notify
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Enable oxy wave data | Send | 0x0f | 0x85 | Switch | |
| | Receive | 0x0f | 0x05 | Frequency |
```c++
typedef struct
{
  char switch; //0x00: turn off oxi wave notify; 0x01 turn on.
} Switch
```
```c++
typedef struct
{
  char frequency; // oxi wave data pack frequenct
} Frequency
```
## 3.11 Enable Respiration Param Notify
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Enable Respiration param data | Send | 0x2d | 0x84 | Switch | |
| | Receive | 0x2d | 0x04 | Frequency |
```c++
typedef struct
{
  char switch; //0x00: turn off Respiration param notify; 0x01 turn on.
} Switch
```
```c++
typedef struct
{
  char frequency; // Respiration param data pack frequenct
} Frequency
```
```
example:
SEND: 0xAA 55 2D 03 84 01 97
RECEIVE：0xAA 55 2D 03 04 01 B8
```
## 3.12 Enable Respiration Wave Notify
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Enable Respiration wave data | Send | 0x2d | 0x83 | Switch | |
| | Receive | 0x2d | 0x03 | Frequency |
```c++
typedef struct
{
  char switch; //0x00: turn off Respiration wave notify; 0x01 turn on.
} Switch
```
```c++
typedef struct
{
  char frequency; // breRespirationthe wave data pack frequenct
} Frequency
```
```
example:
SEND: 0xAA 55 2D 03 83 01 F9
RECEIVE：0xAA 55 2D 03 03 32 8A
```
## 3.13 Oxi Param Notify data pack
OxiParam data is 1Hz, means one data pack will be received every second.
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| oxi param data | Receive | 0x0f | 0x01 | OxiParam | |
```c++
typedef struct
{
  unsigned char spo2;  //spo2, value:0%-100%. 0% means invalid data
  unsigned short pr; //pulse rate, little endian, 0-511 bpm. 0 means invalid data
  unsigned char pi; //10 times of pi. pi value 0-25.5; 0 means invalid.
  char staus1;  // bit1: probe off; bit 3: probe error; bit7-6: 00 adult mode, 01 baby mode
  char status2; //bit 5: oxi wave notify switch; bit6-7: battery level, 0-3
} OxiParam
```
## 3.14 Oxi Wave Notify data pack
OxiWave data is 50Hz, means 50 samples will be received every second. One data pack contains 5 samples.
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| oxi wave data | Receive | 0x0f | 0x02 | OxiWave | |
```c++
typedef struct
{
  unsigned char sample0;  // bit 7: 1 pulse detected; bit 0-6: wave data,0-127
  unsigned char sample1;
  unsigned char sample2;
  unsigned char sample3;
  unsigned char sample4;
} OxiWave
```
## 3.15 Respiration Param Notify data pack
RespirationParam data is 1Hz, means one data pack will be received every second.
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Respiration param data | Receive | 0x2d | 0x02 | RespirationParam | |
```c++
typedef struct
{
  unsigned char rr; //respiration rate
  char flag;  // bit0: 0 respiration signal normal, 1 abnormal.
} RespirationParam
```
## 3.16 Respiration Wave Notify data pack
RespirationWave data is 50Hz, means 50 samples will be received every second. One data pack contains 1 samples.
| Command | Send/Receive | Token | Data Type | Message | note |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Respiration wave data | Receive | 0x2d | 0x01 | RespirationWave | |
```c++
typedef struct
{
  unsigned short flow;  // Respiration flow wave, 0-4095, little endian.
  unsigned short snore;  // snore wave, 0-4095, little endian.
} RespirationWave
```