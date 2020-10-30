---
title: Ampron LED Displays Communication Protocol API

language_tabs: # must be one of https://git.io/vQNgJ
  - json: JSON

toc_footers:
  - <a href='https://ampron.eu/article/direction-and-amount-of-free-parking-spaces-led-signs/' target="_blank">See sample use case</a>
  - <a href="mailto:team@ampron.eu?subject=Question about API">Send Us Questions</a>

includes:
  - errors
  - howto

search: true
---

# Introduction

The purpose is to describe the configuration of Ampron LED display communication protocol. 

## AmpronLED NextGen software

AmpronLED NextGen software is designed to drive and monitor the status of Ampron LED displays. Our LED display hardware and AmpronLED software together combine the solution that we define as Smart LED Display System.

Current API defines the ways of configuring and communicating with the software. The general approach, method, syntax and semantics are described in the following chapters.

The communication protocol is defined to a level of technical detail which should assure a clear understanding of how to drive, monitor and manage the software and displays.

Following is structured in a way that at the left side you will see descriptions and at the right side are the example values.

# Communication Protocol

This chapter is intended to describe the communication concept and protocol between Third Party Communication Module (TPCM) and AmpronLED NextGen software for the operational environment. The communication protocol is closely related to the configuration of AmpronLED NextGen software.

Main Communication Process:

**PHASE1**
-    Third-Party Communication Module (TPCM) sends a control message to AmpronLED NextGen

**PHASE2**
-    AmpronLED NextGen visualises data on the Smart LED Display System (SLDS)

**PHASE3**
-    AmpronLED NextGen makes feedback messages available for parsing

## General View

The complete system is composed of the following components:

* Third Party Communication Module
* AmpronLED NextGen Software running inside of SLDS Display Units
* Communication Network

**Third-party communication module** is responsible for sending control messages and polling status messages.

**AmpronLED NextGen Software** is responsible for receiving the control messages from TPCM, creating status messages for TPCM and visualising content on SLDS Unit(s).

*Note: as for generalization purposes, only Ampron standard software with its communication format, syntax and semantics are described.*

**SLDS Display Units** are the visualization units (LED, LCD/TFT or VMS displays) which provide the visual results of communication between Third Party communication module and AmpronLED NextGen software.

**Communication Network** is considered to include all equipment and measures between TPCM and SLDS Display Units.

![Ampron LED Communication Overview]
(images/Ampron_LED_Communication_Overview_NextGen.png) 

## Method, Syntax and Semantics
### Method
Messaging between TPCM and AmpronLED NEXTGEN is performed by using **HTTP GET** method.

### Syntax
Control message format between TPCM and AmpronLED NextGen is HTTP Request:

`GET http://ipaddress:port/mlds?nameX=valueX&nameY=valueY&nameZ=valueZ`

|Message element|Description|
|--------- | ------- |
|`http://`|hypertext transfer protocol identifier, constant string|
|`ipaddress`|IP address of the SLDS running AmpronLED NextGen software|
|`:port`|communication port which is listened by AmpronLED NextGen, integer 1025…65535|
|`/mlds?`|resource request, a constant string
|`nameX=valueX`|message pairs/AreaData, predefined alphanumeric pairs
|`&`|separator, constant character

### Semantics
`nameX = valueX` - nameX is variable identifier and valueX is value of variable

There are five types of variables:

* Layouts
* AreaData
* Status responses 
* Addressing
* Numbering

Please see the next chapter for details

## Variables and Requests
### Layouts 
Layouts are configurable views of LED displays. There can be multiple layouts for every single display. However, from the communications point of view, only one layout can be defined in a single GET request.

Layouts are defined in config.json file, inside the block named “layout”.

`GET http://ipaddress:port/mlds?id=GATE_2&layout=vehiclenumber&nameZ=valueZ&inumber=XXXXX`

## AreaData 
AreaDatas are configurable sections (Areas) in every Layout. There can be multiple Areas in one Layout. From the communications point of view, if there are nameX=valueX pairs in the GET request, which are not described in the configuration file, then they will be disregarded as faulty.
If there are no pairs or fewer pairs than defined in Area configuration, the request will be handled as a request for a blank screen or blank area respectively. There are 6 types of AreaData (static text, live text, images, date&time, video and images):

`GET http://ipaddress:port/mlds?id=GATE_2&layout=vehiclenumber&kiosk=21&plate=123ABC&inumber=XXXXX`

## Status Responses 
### Command Status
AmpronLED NextGen software confirms synchronously with OK if the HTTP GET request is received and does not contain forbidden elements.
AmpronLED NextGen software confirms the last control message status by visualising it at:

`http://ipaddress:port/info`

### Configuration Status 
AmpronLED NextGen software confirms the last configuration check result by visualising it at:

`http://ipaddress:port/info`

### Network configuration
AmpronLED NextGen software current network configuration is visible at:

`http://ipaddress:port/getethernetconfig`

## Addressing 
There can be 1 SLDS unit operated by one AmpronLED NextGen instance, however multiple id-s can be present in config.json:

MLDS unit identifier -> variable nameX is “id”, value is string max 255 chars

`GET http://ipaddress:port/mlds?id=GATE_2&layout=vehiclenumber&kiosk=21&plate=123ABC&inumber=XXXXX`


## Encoding of URL Query Parameters
Characters from the unreserved set are represented as-is without translation, other characters are converted to bytes according to UTF-8, and then percent-encoded.
See <a href='https://en.wikipedia.org/wiki/Percent-encoding' target="_blank">Wikipedia Percent Encoding</a>

# Configuration

## Main
The main configuration is set in a `config.json` file. Other configuration items are defined in various files and are linked to the main file according to specific needs using referencing.

## Uploading configuration
After changes to configuration, configuration files must: (a) be uploaded to SLDS via SFTP or (b)zipped and made available to be reached at desired URL. To download configuration files to SLDS, relevant HTTP GET command must be given:

`http://ipaddress:port/load_config?url=http://URL/conf.zip

## Activation of new configuration

Configuration is checked every 3000 seconds and activated automatically, to force check use HTTP GET command:

`http://ipaddress:port/reload_config`

After configuration change, check the status at:

`http://ipaddress:port/info`

If errors are detected in configuration files, they are disregarded and relevant information is shown on the above link.

## Managing brightness manually

LED Screen brightness can be managed manually by sending HTTP GET request with desired value 0...100:

`http://ampronledIP:port/brightness/VALUE`
 
NOTE! To disable internal brightness scheduler to avoid it from overriding the manual value define in config.json:

"sensorBrightness": true,

## Changing Network configuration

To change network configuration and enable/disable DHCP use commands (NB! remember, this should be used only if one id is defined in config.json):

`http://OLDipaddress:port/setethernetconfig?dhcp=true&ip=NEWipaddress&netmask=NEWnetmaskip&gateway=NEWgatewayip&dns=NEWdnsip` 

to check status:

`http://NEWipaddress:port/getethernetconfig`


## Configurable Items

```json
{
    "port": "9199",

    "displays": {
        "GATE_OUT2": {
            
        "displayName": "anyname",
        "displayIp": "192.168.100.100",
            "displayPort": "9551",
            "controllerType": "T430",
			"infoScreenTtl": "15"
            
            
            "displayHeight": "160",
            "displayWidth": "96",
            
            "layout": {
                "vehicle": {
                    "numberplate": {
                        "coordinates": "0 0 95 29",
                        "type": "text",
						"textFromStart":"true",
						"textLiveUpdate":"false",
						"headToTail":"5",
						"scrollSpeed": "4",
						"font": "font1.ttf",
						"align": "center",
                        "fontSize": "24",
                        "fontColor": "255 255 255"
                    },
                    "infotext": {
                        "coordinates": "0 30 95 69",
                        "type": "static",
                        "align": "center",
						"font": "font2.ttf",
                        "fontSize": "24",
                        "fontColor": "255 255 255",
                        "params": {
                            "21": "Some static text"
                        }
                    },
                    "arrow": {
                        "coordinates": "0 70 95 100",
                        "type": "image",
						"ttl": "300"
						"defaultValue": "This is default text"
                        "params": {
                            "S": "fwd.png",
                            "RE": "left.png"
                        }
                    },
                    "commercial": {
                        "coordinates": "0 101 95 120",
                        "type": "video",
                        "params": {
                            "VID1": "video1.mp4",
                            "VID2": "video2.mp4"
                        }
                    },
                    "clock": {
                        "coordinates": "0 121 95 140",
                        "type": "date",
                        "dateFormat": "yyyy-MM-dd",
						"dateOffset": "+3"
                        }
                    }
                },
                "message": {
                    "all": {
                        "coordinates": "0 0 95 100",
                        "type": "image",
                        "params": {
                            "WAIT": "pleasewait.png",
                            "CLOSED": "closed.png"
                        }
                    }
                }
            }
        }
    }
}

```
- Server listening port
- Display group name
- Display ID
- Display IP and port
- Display meta name
- Display controller type
- Display height
- Display width
- Display layout
- Display areas
- Content-type
- Text-based content alignment
- Text-based content font size
- Text-based content font colour
- Picture based content parameters
- Video based content parameters
- Date based content parameters

## Listening Port

```json

"port":  "9090",

```

Define the following value in `config.json` file:

`port`        -> Numerical value within the range of from 1025 up to 65535

## Displays Group
 
```json
"displays":
```
`displays` Displays subgroup. Fixed value.

## Display ID
 
```json
"DISPLAYID"
```

The first thing to do for adding new display is defining following value in `config.json` file:

`Display ID` - String type unique value, up to 255 characters

Replace default DISPLAYID with your own unique display ID name.

## Display IP and Port

```json

"displayIp":    "192.168.200.200",
"displayPort":  "10000",
```
Set the following value in `config.json` file:

`“displayIP“`          -> In the form of standard IPv4 address
`“displayPort“`         -> Numerical value according to the specification

## Display Metaname

```json

"displayName":  "anyname",
```

Define the following value in `config.json` file:

`displayName`        -> String type value, up to 255 characters

## Display Controller Type

```json
"controllerType": "G20",
```

Define the following value in `config.json` file:

`controllerType`          -> controller model

## Show IP on boot

```json
"infoScreenTtl": "15"
```

Define the following value in `config.json` file:

`infoScreenTtl`          -> define value in seconds

## Display Size

```json
"displayHeight":    "160",
"displayWidth":     "96",
```

Define the following values in `config.json` file:

`displayHeight`     -> Display height in pixels

`displayWidth`          -> Display width in pixels

## Display Layout Management

```json
    "layout": {
        "layoutName": {
            "areaID": {
                    
                            }
                    }
                }      
                            
```

Define the following value in `config.json` file:

`layoutName`    -> String type value, up to 255 characters

`areaID`        -> String type of unique value, up to 255 characters

Consider the values that make sense in your overall system configuration.


## Area Coordinates

```json
"coordinates": "0 0 100 100",
```

Define the following values in `config.json` file:

`coordinates`    -> Area coordinates in pixels (x y X Y)

<aside class="warning">
Must physically fit into display area and no overlap with other areas.
</aside>

*x,y - square area upper left corner.*

*X,Y - square area lower right corner.*

## Area Types

```json
"type": "static",
```

Define the following values in `config.json` file:

`type`           -> Possible values - `static`, `text`, `video`, `date` or `image`

### Static Text Variables

```json
"align": "center",
           "fontSize": "24",
           "fontColor": "255 255 255",
		   "ttl": "300",
		   "defaultValue": "This is default text",
           "params": {
                        "TEXTID": "text"
                     }
```

If the chosen area type was chosen `static` then define section `texts` and add following variables:

`align` -> text alginment. Possible values `center` or `left`
`fontSize` -> font size in pixels, must be smaller than area height
`fontColor` -> font color in RGB
`ttl` -> time in seconds after which content is erased or replaced by "defaultValue" 
`defaultValue` -> after expiration of ttl, the area content can be define as default text or image (picture1.png)
`params` -> parameters subgroup. Fixed value
`TEXTID` -> String type value, up to 255 characters
`text` -> Text to be displayed, up to 255 characters

For "text" and "static" areas it is possible to define font color also with the GET request: 

http://ampronledIP:port/mlds?id=DISPLAY_ID&layout=LAYOUT_NAME&AREA_NAME=HELLO&fontColor[AREA_NAME]=0_255_255 , where 0_255_255 is the color of text in RGB

It is possible to define background color of "text" and "static" areas text. To set the background color of text, define following parameter under the relevant area:

"backgroundColor": "0 255 255",    whereas "0 255 255" is the color of text background in RGB

it is also possible to define alpha channel for this feature, 

"backgroundColor": "0 255 255 127",  whereas 127 is alpha channel value 0...127

### Freetext Parameters

```json
"align": "center",
"fontSize": "24",
"fontColor": "255 255 255"
"textFromStart":"true",
"textLiveUpdate":"false",
"headToTail":"5",
"scrollSpeed": "4",
"font": "font1.ttf",

```

If the chosen area type was chosen `text` then define following variables:

`align`     -> text alginment. Possible values `center`, `right` or `left`. Valid only when text fits into area.
`fontSize`  -> font size in pixels, must be smaller than area height
`fontColor` -> font color in RGB
`textFromStart` -> start scrolling text from beginning on update
`textLiveUpdate` -> update text on the fly
`headToTail` -> add spaces between scrolling text end and start
`scrollSpeed` -> manage scrolling speed, values 0 to 8, default 4
`font` -> assign specific font to the area, relevant .ttf file must be located in `fonts/filename`, relative to `ampronled`

For "text" and "static" areas it is possible to define font color also with the GET request: 

http://ampronledIP:port/mlds?id=DISPLAY_ID&layout=LAYOUT_NAME&AREA_NAME=HELLO&fontColor[AREA_NAME]=0_255_255 , where 0_255_255 is the color of text in RGB

It is possible to define background color of "text" and "static" areas text. To set the background color of text, define following parameter under the relevant area:

"backgroundColor": "0 255 255",    whereas "0 255 255" is the color of text background in RGB

it is also possible to define alpha channel for this feature, 

"backgroundColor": "0 255 255 127",  whereas 127 is alpha channel value 0...127

### Picture Parameters

```json
"params": {
            "PICTUREID": "filename1.gif",
          }
```

Add following values:

`PICTUREID`    -> String type unique value, up to 255 characters
`filename.gif`    -> String type value, up to 255 characters filename with the filename extension.

All files must be located in the catalogue `images/filename`, relative to `ampronled`

### Video Parameters

```json
"params": {
            "VIDEOID": "filename1.mp4",
          }
```

If the chosen area type was chosen `video` then define following variables:

`VIDEOID`    -> String type unique value, up to 255 characters
`filename1.mp4`    -> String type value, up to 255 characters filename with the filename extension.

All files must be located in the catalogue `images/filename`, relative to `ampronled`

### Date and Time Parameters

```json

"fontSize": "24",
"fontColor": "255 255 255",
"font": "font1.ttf",
"dateOffset": "+3"
"dateFormat": "yyyy-MM-dd",


```

If the chosen area type was chosen `date` then define following variables:

`fontSize`  -> font size in pixels, must be smaller than area height
`fontColor` -> font color in RGB
`font` -> assign specific font to the area, relevant .ttf file must be located in `fonts/filename`, relative to `ampronled`
`dateOffset` -> offset in hours from GMT+0

`dateFormat` -> see time formatting options:

				Symbol   Meaning                Presentation       Example
				------   -------                ------------       -------
				G        era designator         (Text)             AD
				y        year                   (Number)           1996
				M        month in year          (Text & Number)    July & 07
				L        standalone month       (Text & Number)    July & 07
				d        day in month           (Number)           10
				c        standalone day         (Number)           10
				h        hour in am/pm (1~12)   (Number)           12
				H        hour in day (0~23)     (Number)           0
				m        minute in hour         (Number)           30
				s        second in minute       (Number)           55
				S        fractional second      (Number)           978
				E        day of week            (Text)             Tuesday
				D        day in year            (Number)           189
				a        am/pm marker           (Text)             PM
				k        hour in day (1~24)     (Number)           24
				K        hour in am/pm (0~11)   (Number)           0
				z        time zone              (Text)             Pacific Standard Time
				Z        time zone (RFC 822)    (Number)           -0800
				v        time zone (generic)    (Text)             Pacific Time
				Q        quarter                (Text)             Q3
				'        escape for text        (Delimiter)        'Date='
				''       single quote           (Literal)          'o''clock' 


## Monitoring and BITE

Status information of SLDS can be checked http://ipaddress:port/info and http://ipaddress:port/status

It is possible to check screenshot of current view on the screen(this will currently not show video area content): http://ipaddress:port/screenshot

## Logging

Logging should be implemented by TPCM via checking `http://ipaddress:port/info` page