# Arming and disarming

<-- [Home](README.md)

There are a few ways that you can arm and disarm an ideAlarm zone

## Using the openHAB web GUI
You can of course use the openHAB web GUI to switch the arming mode. This is not very convenient though and is probably useful only for testing.

## Programatically (E.g. using scripts)
You can make scrips in any of openHAB's scripting environments. Below are two jsr223 Jython script examples. Both of them are based on Jython scripting for openHAB 2.x.

### A simple jsr223 Jython script example:
```
from core.rules import rule
from core.triggers import when

@rule("Bed room arming", description="This is the bed room arming rule", tags=[])
@when("Item Bedroom_Switch changed to ON")
def bedRoomArming(event):

    bedRoomArming.log.info('Bedroom button pushed')
    events.sendCommand('Toggle_Z1_Armed_Home', 'ON')

```

### A little bit more advanced jsr223 jython script:

If you have a physical button device that you want to push 3 times within 10 seconds to toggle the arming mode. You may want to do like this to make things more obscure, for example if you just use a lighting switch beside the entrance door for arming and disarming. Security through obscurity In the example below, that physical button item is named 'Spider_Pig_Button'. It's also using the getItemValue function from mylib.

```
from core.rules import rule
from core.triggers import when
from core.utils import getItemValue

@rule("Spider Pig rule", description="Keeps track of how many times the Spider Pig Button has been pressed. Counter will be reset by the expire binding.", tags=[])
@when("Item Spider_Pig_Button changed to ON")
def spiderPig(event):
    qty = getItemValue('Count_Spider_Pig', 0)
    self.log.info('Spider pig was pressed. Qty='.format(qty+1))
    if qty == 2:
        events.sendCommand('Toggle_Z1_Armed_Away', 'ON')
    events.postUpdate('Count_Spider_Pig', str(qty+1))
```

Here is how the definition of the 2 items might look like:
```
Number Count_Spider_Pig  {expire="10s,command=0"}
Switch Spider_Pig_Button "Spider pig button" <switch> {channel="zwave:device:XXXXXXX:nodeNN:sensor_binary1"}
```

## MQTT Alarm Control Panel
(Untested) You can easily integrate an Alarm Control Panel that communicates using MQTT. For example the MQTT Alarm Control Panel for Home Assistant project might work well for integration with ideAlarm.

### PREREQUISITS:

You'd need to set up a MQTT server. Unless you already have one you'd probably want to set up a MQTT message broker such as Mosquitto on the same server as the one that runs openHAB.

openHAB MQTT Binding should be installed and configured.

Add two openHAB items like in the example below:
```
String Alarm_CPanel_State   "Alarm CPanel state [%s]" {channel="mqtt:topic:alarmpanel:XXXXXX"}
String Alarm_CPanel_Command "Alarm CPanel command [%s]" {channel="mqtt:topic:alarmpanel:XXXXX"}
```

The Alarm Control Panel will send an update to the `Alarm_CPanel_Command` item with a payload to notify ideAlarm to arm or disarm. The MQTT service will use the `Alarm_CPanel_State` item with the payload to notify the Alarm Control Panel of the current ideAlarm state, which will update the interface accordingly.

When the prerequisites are met, it's time to make a Custom Helper Function in `custom.py` that will synchronize your newly created items `Alarm_CPanel_State` and `Alarm_CPanel_Command` with an ideAlarm zone. See [Event helpers](Eventhelpers.md). There's a template custom script that we will use as a starter. Note that this is a python library script so you'd have to reload the Jython rule binding before any changes that you make to the file gets effective.

Change the function `onArmingModeChange` so that it updates the `Alarm_CPanel_State` item when there is a change of arming mode for a zone. Below is an example:

```
def onArmingModeChange(zone, newArmingMode, oldArmingMode):
    '''
    This function will be called when there is a change of arming mode for a zone.
    oldArmingMode is None when initializing.
    '''
    if zone.zoneNumber == 1:
        if oldArmingMode is not None:
            pass

        if newArmingMode == ARMINGMODE['DISARMED']:
            events.sendCommand('Alarm_CPanel_State', 'disarmed')
        elif newArmingMode == ARMINGMODE['ARMED_HOME']:
            events.sendCommand('Alarm_CPanel_State', 'armed_home')
        elif newArmingMode == ARMINGMODE['ARMED_AWAY']:
            events.sendCommand('Alarm_CPanel_State', 'armed_away')

        log.debug(u"onArmingModeChange: [{}] ---> [{}]".format(zone.name.decode('utf8'), kw(ARMINGMODE, newArmingMode)))

```

For syncing the new arming modes that comes from the Alarm Control Panel we create a regular jython script. Below is an example of that:

```
from core.rules import rule
from core.triggers import when
from core.utils import getItemValue
from logging import DEBUG, INFO, WARNING, ERROR
from community.idealarm import ideAlarm, ARMINGMODE

@rule("Sync APC commands", description="MQTT Alarm Control Panel synchronization", tags=[])
@when("Item Alarm_CPanel_Command changed")
def SyncACPCommands(event):

    if event.state == 'ARM_HOME' and ideAlarm.getZoneStatus('MY_ZONE_NAME') != ARMINGMODE['ARMED_HOME']:
        ideAlarm.alarmZones[ideAlarm.getZoneIndex('MY_ZONE_NAME')].setArmingMode(ARMINGMODE['ARMED_HOME'])
    elif event.state == 'ARM_AWAY' and ideAlarm.getZoneStatus('MY_ZONE_NAME') != ARMINGMODE['ARMED_AWAY']:
        ideAlarm.alarmZones[ideAlarm.getZoneIndex('MY_ZONE_NAME')].setArmingMode(ARMINGMODE['ARMED_AWAY'])
    elif event.state == 'DISARM' and ideAlarm.getZoneStatus('MY_ZONE_NAME') != ARMINGMODE['DISARMED']:
        ideAlarm.alarmZones[ideAlarm.getZoneIndex('MY_ZONE_NAME')].setArmingMode(ARMINGMODE['DISARMED'])

```

<-- [Home](README.md)