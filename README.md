# Hive Thermostat Zigbee2MQTT


Configuration for a Hive thermostat using  that aims to recreate the actions found
on the hive remote - minus the schedule.

## Introduction



The climate integration in HA sends MQTT messages to an exdpoint
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
Once connected to a Lovelace component we can catch the original
request and re-publish the request  to a new MQTT endpoint 
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


## A few things This Does

Sets the min and max value to 12 and 32 because we don't
 really want to go below the frost prevention temp anyway.

Uses input number helpers to set defaults for
Frost temperature, Boost temperature and boost duration.

Uses an input helper text to keep track of the current system mode
It just makes things easier.

prevents temperatures of 1 degrees sent from the thermostat
from being set unless the system mode has been set to '__off__'

Republishes HA requests to thermostat in the correct order.

Uses the '__cool__' system mode as the '__boost__' function. 
(Requires the correct mapping in the  lovelace component used.
e.g. Simple Thermostat)



I've not set a schedule on the remote. It sends back extra data 
that just gets in the way. Instead, all schedules are set 
to the required frost temperature setting. The idea being the
schedule will be controlled via HA. You can still increase the
temperature when set to schedule, but it will be reset to the
frost temperature when the current time span has reached te end


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
