# DeepGTAV v2 Revision

*A plugin for GTAV that transforms it into a vision-based self-driving car research environment.*

<img src="https://github.com/lyzMaster/mess/raw/master/gta-diagram-e0d755e52a552b0a6615418efc77aa1bdbd0bd5b88c3ae056e65f2168867c6c6.jpg" alt="Self-Driving Car" width="900px">

## Installation
1. ȷ�� *steering��throttle��brake* ���ڴ�����vehicle�׵�ַ��ƫ��������ʹ�� *bin/Release* Ԥ�����ļ����ԣ�
2. ʹ��visual studio������Ŀ
3. �� *bin/Release* �ļ�������GTAV��Ŀ¼
4. Replace your saved game data in *Documents/Rockstar Games/GTA V/Profiles/* with the contents of *bin/SaveGame*
5. Download *[paths.xml](https://pan.baidu.com/s/1GlqFqVK3W7NMNhCH99VtOw)* and store it also in the GTAV installation directory. 

## Ѱ��vehicle�׵�ַoffset
1. ���ṩ��Ԥ�����ļ�������GTAV��Ŀ¼������GTAV����8000�˿ڷ���`start`ָ��
2. ʹ�� *[Cheat Engine](https://www.cheatengine.org/)* ����GTAV��Ŀ¼�µ�`VehicleBaseAddress.log`�ļ���ĵ�ַ��ʼ�����˵�ַ��`0xfa0`λ����`[-2,2]`��`float`ֵ����������ͬʱ������Ϸ���۲��ַ��ֵ�ı仯���
3. Ԥ�����ļ�offset�� `throttle��0x8A4��`��`brake��0x8A8��`��`steering��0x89C��`
4. �ڰ汾 *1.0.231.0 NON-STEAM* ��������

## Revision Info
1. ����`VehicleBaseAddress.log`�������
2. ���Ľ�ͼ��ʽ�����ϽǸ�Ϊ����λ��
3. ����offset

## steering��throttle��brakeֵ����
***steering***:  **[-1,1]**�� `���������ˣ���������ǰ��ʻ`����ֵ�������Űٷֱ�</br>
***brake***:  **[0,1]**������ɲ���İٷֱ�</br>
***steering***:  **[-pi/5,pi/5]**��`����������ת������������ת`</br>


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

