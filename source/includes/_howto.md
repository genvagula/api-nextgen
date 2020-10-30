# How To (Features)

## Brightness control

### It is possible to change the brightness of the display in 6 different ways:

### 1. Automatic daylight

This is the default mode, brightness is changed between 50% and 100% automatically. Time of change is defined by sunset and sunrise with an accuracy of 2 weeks, example:
in the first half of May, the daytime starts later than in the second half of May and in the first half of June later than in the second half of May etc. Default sunset times are taken for local longitude and latitude.

### 2. Automatic daylight, specific location

With this mode, it is possible to define the sunrise and sunset times and also relevant brightness %, for each half months.
To use this feature, it is needed to add a specific brightness-schedule.json file to ampronled directory (with X permissions), if the file is present then it is used, if not then default schedule is used).

### 3. Manual brightness for a single screen

To enable manual brightness setting feature, define the following parameter at `"displayIP"` level:

`"sensorBrightness": true`,

To use this feature send HTTP GET command to the ampronled software:

`http://ampronledIP:port/brightness/VALUE?id=DISPLAY_ID`

VALUE can be from 1 to 100

### 4. Advanced automatic brightness management

It is possible to manipulate different types of data to manage the brightness of screens eg.open web RSS feeds, parsing XML, using MODBUS/TCP etc. This feature requires additional Ampron Monitoring Module hardware unit and Ampron Monitoring Software.

## Configuration

### Description

The main configuration is set in a `config.json` file. Other configuration items are defined in various files and are linked to the main file according to specific needs.

`AFTER CHANGES TO THE CONFIGURATION, THE CONFIGURATION MUST BE RELOADED FOR CHANGES TO TAKE EFFECT`

## Define text colour

For `"text"` and "static" areas, it is possible to define font colour also with the GET request.

`http://ampronledIP:port/mlds?id=DISPLAY_ID&layout=LAYOUT_NAME&AREA_NAME=HELLO&fontColor[AREA_NAME]=0_255_255`

where 0_255_255 is the colour of the text in RGB

## Set and change scroll-speed

Define parameter:

`"scrollSpeed": "VALUE",` VALUE - 0...8, 0-fastest and 8-slowest

if the parameter is not defined then default scroll speed is 4

## Set text background colour

It is possible to define the background colour of `"text"` and `"static"` areas text.

To set the background colour of text, define the following parameter under the relevant area:

`"backgroundColor": "0 255 255"`, whereas `"0 255 255"` is the color of text background in RGB.

It is also possible to define an alpha channel for this feature, `"backgroundColor": "0 255 255 127"`, whereas 127 is alpha channel value 0...127



