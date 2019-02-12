# DeepGTAV v2 Revision

*A plugin for GTAV that transforms it into a vision-based self-driving car research environment.*

<img src="https://github.com/lyzMaster/mess/raw/master/gta-diagram-e0d755e52a552b0a6615418efc77aa1bdbd0bd5b88c3ae056e65f2168867c6c6.jpg" alt="Self-Driving Car" width="900px">

## Installation
1. 确认 *steering，throttle，brake* 在内存中与vehicle首地址的偏移量（可使用 *bin/Release* 预编译文件尝试）
2. 使用visual studio编译项目
3. 将 *bin/Release* 文件拷贝到GTAV根目录
4. Replace your saved game data in *Documents/Rockstar Games/GTA V/Profiles/* with the contents of *bin/SaveGame*
5. Download *[paths.xml](https://pan.baidu.com/s/1GlqFqVK3W7NMNhCH99VtOw)* and store it also in the GTAV installation directory. 

## 寻找vehicle首地址offset
1. 将提供的预编译文件拷贝到GTAV根目录，运行GTAV，对8000端口发出`start`指令
2. 使用 *[Cheat Engine](https://www.cheatengine.org/)* ，从GTAV根目录下的`VehicleBaseAddress.log`文件里的地址开始，到此地址后`0xfa0`位，对`[-2,2]`的`float`值进行搜索，同时进行游戏，观察地址中值的变化情况
3. 预编译文件offset： *throttle（0x8A4），brake（0x8A8），steering（0x89C）*
4. 在版本 *1.0.231.0 NON-STEAM* 运行良好

## Revision Info
1. 增加`VehicleBaseAddress.log`方便调试
2. 更改截图方式由左上角改为任意位置
3. 更改offset

## steering，throttle，brake值含义
**steering**: [-1,1]; 负数：*倒退*，正数： *向前行驶*。数值代表油门百分比</br>
**brake**: [0,1]，代表刹车的百分比</br>
**steering**: [-pi/5,pi/5]，负数：*向左转*，正数：*向右转*</br>


## Messages from the client to DeepGTAV

Messages must be always be sent in two steps: First send the JSON message length in bytes and then send the message itself, this is done so DeepGTAV knows when to stop reading a message.

### Start

This is the message that needs to be sent to start DeepGTAV, any other message sent prior to this won't make any effect. Along with this message, several fields can be set to start DeepGTAV with the desired initial conditions and requested *Data* transmission. 

When this message is sent the environment starts, the game camera is set to the front center of the vehicle and *Data* starts being sent back to the client until the client is disconnected or an *Stop* message is received

Here follows an example of the *Start* message:
```json
{"start": {
  "scenario": {
    "location": [1015.6, 736.8],
    "time": [22, null],
    "weather": "RAIN",
    "vehicle": null,
    "drivingMode": [1074528293, 15.0]
  },
  "dataset": {
    "rate": 20,
    "frame": [227, 227],
    "vehicles": true,
    "peds": false,
    "trafficSigns": null,
    "direction": [1234.8, 354.3, 0],
    "reward": [15.0, 0.5],
    "throttle": true,
    "brake": true,
    "steering": true,
    "speed": null,
    "yawRate": false,
    "drivingMode": null,
    "location": null,
    "time": false    
  }
}}
```
The scenario field specifies the desired intitial conditions for the environment. If any of its fields or itself is null or invalid the relevant configuration will be randomly set.

The dataset field specifies the data we want back from the game. If any of its fields or itself is null or invalid, the relevant *Data* field will be deactivated, except for the frame rate and dimensions, which will be set to 10 Hz and 320x160 by default.

### Config

This message allows to change at any moment during DeepGTAV's execution, the initial configuration set by the *Start* message.

Here follows an example of the *Config* message (identical to the *Start* message):
```json
{"start": {
  "scenario": {
    "location": [1015.6, 736.8],
    "time": null,
    "weather": "SUNNY",
    "vehicle": "voltic",
    "drivingMode": -1
  },
  "dataset": {
    "rate": null,
    "frame": null,
    "vehicles": false,
    "peds": true,
    "trafficSigns": null,
    "direction": null,
    "reward": null,
    "throttle": null,
    "brake": false,
    "steering": false,
    "speed": true,
    "yawRate": null,
    "drivingMode": null,
    "location": null,
    "time": true   
  }
}}
```
In this case, if any field is null or invalid, the previous configuration is kept. Otherwise the configuration is overrided.

### Commands

As simple as it seems, this message can be sent at any moment during DeepGTAV's execution to control the vehicle. Note that you will only be able to control the vehicle if *drivingMode* is set to manual.

Here follows an example of the *Commands* message:
```json
{"commands": {
  "throttle": 1.0,
  "brake": 0.0,
  "steering": -0.5
}}
```

### Stop

Stops the environment and allows the user to go back to the normal gameplay. Simply disconnecting the client produces the same effect.

Here follows an example of the *Stop* message:
```json
{"stop": {}}
```
## Messages from DeepGTAV to the client

DeepGTAV will always send the messages in two steps: First it will send the message length in bytes and then it will send the message itself, so the client can know when to stop reading it.

The messages rate and content will depend on the configuration set by the *Start* or *Config* messages, and are always sent consecutively (Frame, Data).

### Frame

This is a byte array containing the RGB values of the current GTAV screen (resized to the specified width and length). Make sure the window is not minimized, otherwise the values will be all zeroes (black).

