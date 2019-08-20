# IKEA Tradfri COAP Docs

How can you communicate to your ikea tradfri gateway/hub through coap-client. Tested with Tradfri hub version 1.3.0014 and 1.8.0025.

## Table of Contents
   * [IKEA Tradfri COAP Docs](#ikea-tradfri-coap-docs)
      * [Table of Contents](#table-of-contents)
      * [Install coap-client](#install-coap-client)
      * [Authenticate](#authenticate)
      * [The URL](#the-url)
      * [Get a list of all devices](#get-a-list-of-all-devices)
      * [Get info about a specific device](#get-info-about-a-specific-device)
      * [Get a list of all groups](#get-a-list-of-all-groups)
      * [Get info about a specific group](#get-info-about-a-specific-group)
      * [Get a list of all scenes](#get-a-list-of-all-scenes)
      * [Get info about a specific scene](#get-info-about-a-specific-scene)
      * [Bulbs](#bulbs)
         * [Your first bulb](#your-first-bulb)
         * [Payload](#payload)
         * [Colors](#colors)
            * [Cold / Warm Bulbs](#cold--warm-bulbs)
            * [RGB Bulbs](#rgb-bulbs)
         * [More Colors](#more-colors)
      * [Plug](#plug)
         * [Your first plug](#your-first-plug)
         * [Payload](#payload-1)
      * [Blind](#blind)
         * [Your first blind](#your-first-blind)
         * [Payload](#payload-2)
      * [Endpoints](#endpoints)
         * [Global](#global)
         * [Gateway](#gateway)
      * [Codes](#codes)
         * [Devices](#devices)
         * [General parameters](#general-parameters)
         * [Group parameters](#group-parameters)
         * [Scene parameters](#scene-parameters)
         * [Sensor parameters](#sensor-parameters)
         * [Bulb parameters](#bulb-parameters)
         * [Plug parameters](#plug-parameters)
         * [Blind parameters](#blind-parameters)
      * [License](#license)

## Install coap-client
Before you can talk to you Ikea gateway/hub you need to install the coap-client:
1. Download the `install-coap-client.sh` from github.
2. Chmod the file so you can run it.
3. Run the file as root.

## Authenticate
First we need to create a preshared key. This key can then be used to authenticate yourself:
Please note: this key will expire if you don't use it in 6 weeks from activation. Every time you use this key the time will be extended accordingly.

```
coap-client -m post -u "Client_identity" -k "$TF_GATEWAYCODE" -e '{"9090":"$TF_USERNAME"}' "coaps://$TF_GATEWAYIP:5684/15011/9063"
```

* $TF_USERNAME: Can be a random name as long as you use it in the other requests you want to make
* $TF_GATEWAYCODE: Is the security code at the bottom of your Gateway/HUB. It should be 16 characters long.
* $TF_GATEWAYIP: Is the IP of your Gateway/HUB

This will then respond something like this:
```
{
  "9091": "$TF_PRESHARED_KEY", // Preshared Key
  "9029": "1.3.0014" // Gateway Firmware version
}
```

## The URL
To control a bulb you need to know it's URL. This is very easy. The URL's all begin with `coaps://$TF_GATEWAYIP:5684/15001`

The bulbs will have addresses beginning at `65537` for the first bulb, `65538` for the second, and so on. So, to control the first bulb, you would use the following url:
```
coaps://$TF_GATEWAYIP:5684/15001/65537
```

## Get a list of all devices
To get a complete list of all devices connected to your hub use the following command:
```
coap-client -m get -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" "coaps://$TF_GATEWAYIP:5684/15001"
```

## Get info about a specific device
To check the current status of a connected device and to know which device is linked to a specified id, use this command:
```
coap-client -m get -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" "coaps://$TF_GATEWAYIP:5684/15001/$TF_DEVICEID"
```

> `$TF_DEVICEID` is an id from the list generated by the previous command. id's are 5 digits long.

## Get a list of all groups
To get a complete list of all groups use the following command:
```
coap-client -m get -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" "coaps://$TF_GATEWAYIP:5684/15004"
```

## Get info about a specific group
To check the current status of a group and to know which devices are linked to the specified id, use this command:
```
coap-client -m get -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" "coaps://$TF_GATEWAYIP:5684/15001/$TF_GROUPID"
```

> `$TF_GROUPID` is an id from the list generated by the previous command. id's are 6 digits long.

## Get a list of all scenes
To get a complete list of all scenes use the following command:
```
coap-client -m get -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" "coaps://$TF_GATEWAYIP:5684/15005/$TF_GROUPID"
```

> `$TF_GROUPID` is an id from the list generated by the `Get a list of all groups` command. id's are 6 digits long.

## Get info about a specific scene
To get the current configuration of a scene and to know which devices are linked to the specified id, use this command:
```
coap-client -m get -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" "coaps://$TF_GATEWAYIP:5684/15001/$TF_GROUPID/$TF_SCENEID"
```

> `$TF_SCENEID` is an id from the list generated by the previous command. id's are 6 digits long.

> `$TF_GROUPID` is an id from the list generated by the `Get a list of all groups` command. id's are 6 digits long.

## Bulbs

### Your first bulb
To set the brightness of your first bulb to 50% use the following command:
```
coap-client -m put -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" -e '{ "3311": [{ "5851": 127 }] }' "coaps://$TF_GATEWAYIP:5684/15001/$TF_DEVICEID"
```

### Payload
Here is an example payload for coap-client with explanation what each field does:
```
{
  "3311": [
    {
      "5850": 1, // on / off
      "5851": 254, // dimmer (1 to 254)
      "5706": "f1e0b5", // color in HEX (Don't use in combination with: color X and/or color Y)
      "5709": 65535, // color X (Only use in combination with color Y)
      "5710": 65535, // color Y (Only use in combination with color X)
      "5712": 10 // transition time (fade time)
    }
  ]
}
```

### Colors
The following colors where taken from the IKEA Android app (these could be used in field `"5706"`):

Please note: If you are using another HEX value then these the lamp will default to the Warm Glow color.

#### Cold / Warm Bulbs
```
'f5faf6': 'White',
'f1e0b5': 'Warm',
'efd275': 'Glow'
```

#### RGB Bulbs

```
'4a418a': 'Blue',
'6c83ba': 'Light Blue',
'8f2686': 'Saturated Purple',
'a9d62b': 'Lime',
'c984bb': 'Light Purple',
'd6e44b': 'Yellow',
'd9337c': 'Saturated Pink',
'da5d41': 'Dark Peach',
'dc4b31': 'Saturated Red',
'dcf0f8': 'Cold sky',
'e491af': 'Pink',
'e57345': 'Peach',
'e78834': 'Warm Amber',
'e8bedd': 'Light Pink',
'eaf6fb': 'Cool daylight',
'ebb63e': 'Candlelight',
'efd275': 'Warm glow',
'f1e0b5': 'Warm white',
'f2eccf': 'Sunrise',
'f5faf6': 'Cool white'
```

### More Colors
Ikea RGB bulbs can produce more colors then the list above.

They can produce all colors in the xyY color space.

To understand how this color space works take a look at the diagram below:

![xyY Color Space](https://user-images.githubusercontent.com/7496187/48645383-d4192380-e9e5-11e8-8466-5de1248720ca.png)

To create your own color you need define two values (x and y) from 0 to 65535 with this command:
```
coap-client -m put -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" -e '{ "3311": ["5709": 65535, "5710": 65535] }' "coaps://$TF_GATEWAYIP:5684/15001/$TF_DEVICEID"
```

## Plug
The ikea plug was introduced around oktober 2018.

Also this device can be controlled with the same api.

### Your first plug
To turn on a plug use the following command:
```
coap-client -m put -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" -e '{ "3312": [{ "5850": 1 }] }' "coaps://$TF_GATEWAYIP:5684/15001/$TF_DEVICEID"
```

### Payload
Here is an example payload for coap-client with explanation what each field does:
```
{
  "3312": [
    {
      "5850": 1, // on / off
      "5851": 254 // dimmer (0 to 254) (As of writing there arn't any dimmable plugs. Value's above 0 will simply turn on the plug)
    }
  ]
}
```

## Blind
The ikea blinds where introduced around august 2019.

Also this device can be controlled with the same api.

### Your first blind
To change position of a blind use the following command:
```
coap-client -m put -u "$TF_USERNAME" -k "$TF_PRESHARED_KEY" -e '{ "15015": [{ "5536": 0.0 }] }' "coaps://$TF_GATEWAYIP:5684/15001/$TF_DEVICEID"
```

### Payload
Here is an example payload for coap-client with explanation what each field does:
```
{
  "15015": [
    {
      "5536": 1, // position
    }
  ]
}
```

## Endpoints

### Global
| URL                              | Description            |
|----------------------------------|------------------------|
| coaps://$TF_GATEWAYIP:5684/15001 | Devices endpoint       |
| coaps://$TF_GATEWAYIP:5684/15004 | Groups endpoint        |
| coaps://$TF_GATEWAYIP:5684/15005 | Scenes endpoint        |
| coaps://$TF_GATEWAYIP:5684/15006 | Notifications endpoint |
| coaps://$TF_GATEWAYIP:5684/15010 | Smart Tasks endpoint   |

### Gateway
| URL                                    | Description                     |
|----------------------------------------|---------------------------------|
| coaps://$TF_GATEWAYIP:5684/15011/9030  | Reboot gateway endpoint         |
| coaps://$TF_GATEWAYIP:5684/15011/9031  | Reset gateway endpoint          |
| coaps://$TF_GATEWAYIP:5684/15011/9034  | UpdateFirmware gateway endpoint |
| coaps://$TF_GATEWAYIP:5684/15011/15012 | Gateway details endpoint        |

## Codes

### Devices
| Code  | Description   | Type  |
|-------|---------------|-------|
| 3300  | Motion Sensor | Array |
| 3311  | Light/Bulb    | Array |
| 3312  | Plug          | Array |
| 15015 | Blind         | Array |

### General parameters
| Code | Description   | Type   |
|------|---------------|--------|
| 9001 | Name          | String |
| 9002 | Creation date | Int    |
| 9003 | Instance ID   | Int    |

### Group parameters
| Code | Description             | Type    |
|------|-------------------------|---------|
| 5712 | Transition Time         | Float   |
| 5850 | On/Off                  | Boolean |
| 5851 | Brightness              | Int     |
| 9018 | Accessory Link (Remote) | Array   |
| 9039 | Scene ID                | Int     |

### Scene parameters
| Code  | Description         | Type    |
|-------|---------------------|---------|
| 9058  | Scene Active State  | Boolean |
| 9057  | Device Index ID     | Int     |
| 9068  | Is Scene Predefined | Boolean |
| 15013 | Light Settings      | Array   |

### Sensor parameters
| Code | Description                         | Type    |
|------|-------------------------------------|---------|
| 5601 | Minimum Measured Value              | Float   |
| 5602 | Maximum Measured Value              | Float   |
| 5603 | Minimum Range Value                 | Float   |
| 5604 | Maximum Range Value                 | Float   |
| 5605 | Reset Minimum/Maximum Measure Value | Boolean |
| 5700 | Sensor Value                        | Float   |

### Bulb parameters
| Code | Description       | Type       |
|------|-------------------|------------|
| 5706 | Color HEX String  | HEX String |
| 5707 | Hue               | Int        |
| 5708 | Saturation        | Int        |
| 5709 | colorX            | Int        |
| 5710 | colorY            | Int        |
| 5711 | Color Temperature | Int        |
| 5712 | Transition Time   | Float      |
| 5850 | On/Off            | Boolean    |
| 5851 | Brightness        | Int        |

### Plug parameters
| Code | Description | Type    |
|------|-------------|---------|
| 5850 | On/Off      | Boolean |
| 5851 | Brightness  | Int     |

### Blind parameters
| Code | Description | Type  |
|------|-------------|-------|
| 5536 | Position    | Float |

## License

MIT
