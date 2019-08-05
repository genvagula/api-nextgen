---
title: Ampron LED Displays Communication Protocol API

language_tabs: # must be one of https://git.io/vQNgJ
  - json: JSON

toc_footers:
  - <a href='https://www.ampron.eu/products' target="_blank">See our products</a>
  - <a href='https://github.com/AmpronLED/api/issues' target="_blank">Submit an issue</a>
  - <a href="mailto:team@ampron.eu?subject=Question about API">Send Us Questions</a>

includes:
  - errors

search: true
---

# Introduction

The purpose is to describe the configuration of Ampron LED display communication protocol. 

## AmpronLED software

AmpronLED software is designed to drive and monitor the status of Ampron LED displays. Our LED display hardware and AmpronLED software together combine the solution that we define as Smart LED Display System.

Current API defines the ways of configuring and communicating with the software. The general approach, method, syntax and semantics are described in the following chapters.

The communication protocol is defined to a level of technical detail which should assure a clear understanding of how to drive, monitor and manage the software and displays.

Following is a structured in a way that at the left side you will see descriptions and at the right side are the example values.

# Communication Protocol

This chapter is intended to describe the communication concept and protocol between Third Party Communication Module (TPCM) and AmpronLED software for the operational environment. The communication protocol is closely related to the configuration of AmpronLED software.

Main Communication Process:

**PHASE1**
-    Third-Party Communication Module (TPCM) sends a control message to AmpronLED

**PHASE2**
-    AmpronLED sends data to the Smart LED Display System (SLDS)

**PHASE3**
-    SLDS reports back to AmpronLED with the status message

## General View

The complete system is composed of the following components:

* Third Party Communication Module
* AmpronLED Software
* SLDS Display Units
* Communication Network

**Third-party communication module** is responsible for sending control messages and receiving status messages.

**AmpronLED Software** is responsible for receiving the control messages from TPCM, sending out status messages to TPCM and communicating with SLDS Unit(s).

*Note: as for generalization purposes, only Ampron standard software with its communication format, syntax and semantics are described.*

**SLDS Display Units** are the visualization units (LED displays) which provide the visual results of communication between Third Party communication module and AmpronLED software.

**Communication Network** is considered to include all equipment and measures between TPCM, AmpronLED and SLDS Display Units.

![Ampron LED Communication Overview]
(images/Ampron_LED_Communication_Overview.png)
## Method, Syntax and Semantics
### Method
Messaging between TPCM and AmpronLED is performed by using **HTTP GET** method in both directions.
### Syntax
Control message format between TPCM and AmpronLED is HTTP Request:

`GET http://ipaddress:port/mlds?nameX=valueX&nameY=valueY&nameZ=valueZ`

|Message element|Description|
|--------- | ------- |
|`http://`|hypertext transfer protocol identifier, constant string|
|`ipaddress`|IP address of the server where AmpronLED Software or TPCM is installed|
|`:port`|communication port which is listened by TPCM or AmpronLED, integer 1025…65535|
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
AreaDatas are configurable sections (Areas) in every Layout. There can be multiple Areas in one Layout. From the communications point of view, if there are nameX=valueX pairs in the GET request, which are not described in the configuration file, then the request will be disregarded as faulty.
If there are no pairs or fewer pairs than defined in Area configuration, the request will be handled as a request for a blank screen or blank area respectively. There are 3 types of AreaData (static text, live text and images):

`GET http://ipaddress:port/mlds?id=GATE_2&layout=vehiclenumber&kiosk=21&plate=123ABC&inumber=XXXXX`

## Status Responses 
### Success
AmpronLED software confirms the successful control message execution by responding with HTTP GET.

`GET http://ipaddress:port/mlds?confirm=1&inumber=XXXXX`

### Failed
AmpronLED software responds with unsuccessful control message execution by responding with HTTP GET.

`GET http://ipaddress:port/mlds?confirm=0&inumber=XXXXX`

### Fault
AmpronLED software responds to the faulty command by responding with HTTP GET.


`GET http://ipaddress:port/mlds?confirm=2&inumber=XXXXX`

## Addressing 
There can be 99 MLDS units operated by one AmpronLED instance, usually, multiple instances are used:

MLDS unit identifier -> variable nameX is “id”, value is string max 255 chars

`GET http://ipaddress:port/mlds?id=GATE_2&layout=vehiclenumber&kiosk=21&plate=123ABC&inumber=XXXXX`

## Numbering 
The numbering is done by TPCM (Third Party Communication Module), AmpronLED answers with the string provided by TPCM:

Instructions sequence number -> variable `nameX` is `inumber`, value is string of reasonable length


`GET http://ipaddress:port/mlds?id=GATE_2&layout=vehiclenumber&kiosk=21&plate=123ABC&inumber=XXXXX`

## Encoding of URL Query Parameters
Characters from the unreserved set are represented as-is without translation, other characters are converted to bytes according to UTF-8, and then percent-encoded.
See <a href='https://en.wikipedia.org/wiki/Percent-encoding' target="_blank">Wikipedia Percent Encoding</a>

# Configuration

The main configuration is set in a `config.json` file. Other configuration items are defined in various files and are linked to the main file according to specific needs.
After changes to configuration, files server needs to be restarted.

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

            "confirmIp": "127.0.0.1",
            "confirmPort": "9098",
            
            "displayHeight": "160",
            "displayWidth": "96",
            
            "layout": {
                "vehicle": {
                    "numberplate": {
                        "coordinates": "0 0 100 29",
                        "type": "text",
                "align": "center",
                        "fontSize": "24",
                        "fontColor": "255 255 255"
                    },
                    "infotext": {
                        "coordinates": "0 30 100 69",
                        "type": "static",
                        "align": "center",
                        "fontSize": "24",
                        "fontColor": "255 255 255",
                        "params": {
                            "21": "Some static text"
                        }
                    },
                    "arrow": {
                        "coordinates": "0 70 100 100",
                        "type": "image",
                        "params": {
                            "S": "fwd.bmp",
                            "RE": "left.bmp"
                        }
                    }
                },
                "message": {
                    "all": {
                        "coordinates": "0 0 100 100",
                        "type": "image",
                        "params": {
                            "WAIT": "pleasewait.bmp",
                            "CLOSED": "closed.bmp"
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
- Feedback server IP and port
- Display height
- Display width
- Display layout
- Display areas
- Content-type
- Text-based content alignment
- Text-based content font size
- Text-based content font colour
- Picture based content parameters

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

The first thing to do for adding new display is defined following value in `config.json` file:

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

`controllerType`          -> Display height in pixels

## Feedback Server IP and port

```json

"confirmIp": "127.0.0.1",
"confirmPort": "9092",
```

Define the following value in `config.json` file:

`confirmIp`          -> In the form of standard IPv4 address
`confirmPort`        -> Numerical value within the range 1025->65535

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

`type`           -> Possible values - `static`, `text` or `image`

### Static Text Variables

```json
"align": "center",
           "fontSize": "24",
           "fontColor": "255 255 255",
           "params": {
                        "TEXTID": "text"
                     }
```

If the chosen area type was chosen `static` then define section `texts` and add following variables:

`align` -> text alginment. Possible values `center` or `left`

`fontSize` -> font size in pixels, must be smaller than area height

`fontColor` -> font color in RGB

`params` -> parameters subgroup. Fixed value.

`TEXTID` -> String type value, up to 255 characters

`text` -> Text to be displayed, up to 255 characters

### Picture Parameters

```json
"params": {
            "PICTUREID": "filename1.gif",
          }
```

Add following values:

`PICTUREID`    -> String type unique value, up to 255 characters
`filename.gif`    -> String type value, up to 255 characters fiename with the filename extension.

All files must be located in the catalogue `images/filename`, relative to `ampronled`

### Freetext Parameters

```json
"align": "center",
"fontSize": "24",
"fontColor": "255 255 255"

```

If the chosen area type was chosen `text` then define following variables:

`align`     -> text alginment. Possible values `center` or `left`. Valid only when text fits into area.
`fontSize`  -> font size in pixels, must be smaller than area height
`fontColor` -> font color in RGB


## Monitoring and BITE

Our information system has built-in monitoring that monitors communications to all displays that are added to the configuration file. It also monitors the successful forwarding of the information packages and has the feedback loop to a feedback IP address.
Information system monitors the last sent package success. If the feedback was the same then information is not sent again. After server restart, all active displays receive the command to empty the content of the screen.

## Logging

AmpronLED software logs all actions with timestamps. Logs are kept by default one month. Length may be set different according to specific needs.