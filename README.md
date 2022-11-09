# Hive Thermostat Zigbee2MQTT


Configuration for a Hive thermostat that aims to recreate the actions found
on the hive remote

## Introduction

The climate integration in HA sends MQTT messages to a topic
that is incompatible with what is expected from Zigbee2MQTT.
This results in the thermostat receiving an unrecognised command. This
causes the thermostat to set the __occupied heating setpoint__ to
1 degrees

```angular2html
zigbee2mqtt/heating/set/system_mode  'auto' <-- Climate Integration

zigbee2mqtt/heating/set  {"system_mode": "auto"} <-- Zigbee2MQTT
```

The same problem can also be seen if you attempt to set a temperature less than 5 degrees. 
Because temperatures of less than 5 degrees
are not supported by the thermostat an error is generated and the same temperature is returned.

The end result of all this is that when you attempt to set a temperature
it always jumps back to 1 degrees


## How this gets around the problem


I have created a 'Proxy thermostat' using the MQTT HVAC integration. 
Once connected to a climate component, we can catch the original
request and re-publish the request to a new MQTT topic 
in the correct format. We can then subscribe to this in an 
automation. When the automation picks this up, we have more 
control of what actions to take, what needs to be set, when
we set this before finally publishing to the thermostat via the
MQTT publish service.

We could publish directly to the thermostat, but as I've found 
out the ordering of the attributes that we publish to the 
thermostat isn't guaranteed, so it is difficult to issue
the correct commands to recreate the actions
of the hive remote in HA

i.e. if your thermostat is '__off__' and you change the 
temperature manually, your thermostat shows '__manual__'.
I want to be able to do the same thing in HA


## Requirements

HA and Zigbee2MQTT

MQTT HVAC integration

Input number and text Helpers

Simple Thermostat. You can probably use others and adjust the 
button config accordingly.

<kbd>![alt text](https://github.com/greeeny101/Hive-Thermostat-Zigbee2MQTT/blob/main/Capture.PNG)</kbd>

## A few things This Does

* Aims to emulate the actions provided by the thermostat. In particular, schedules set in the thermostat are replicated 
in the climate component
* Maximum and Minimum temperature values set to 12 and 32 degrees respectively
* Frost prevention temperature between 5 and 16 degrees via an input number. Initial 12 degrees
* Boost temperature between 12 and 32 degrees via an input number. Initial set to 25 degrees
* Boost duration between 5 and 45 minutes via an input number. Initial set to 25 degrees
* Uses the _cool_ system mode to emulate the _boost_ function provided by the thermostat
 __Requires a mapping between the cool and  boost mode in the heating component to work__
* Prevents 1 degrees temperatures from being set in the component. Where applicable sets
the frost prevention temperature instead

Note that initial values will be reloaded when you restart the server. If you don't want this
then remove the initial value to preserve the current value following a restart


## Simple Thermostat Config

```angular2html
type: custom:simple-thermostat
entity: climate.heating_proxy
hide:
  temperature: true
  state: true
sensors:
  - entity: sensor.heating_local_temperature
    icon: mdi:home-thermometer-outline
    unit: ' °C'
  - entity: sensor.heating_running_state
    icon: mdi:fire
layout:
  mode:
    headings: false
    icons: true
    names: true
control:
  hvac:
    auto:
      name: Schedule
    cool:
      name: Boost
      icon: mdi:heat-wave
header:
  name: Heating


```
