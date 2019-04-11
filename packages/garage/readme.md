**Timed_garage_doors**

My code on garage operation based on timers, w/o magnetic sensors  

**Description**  
This package will control your garage doors and assume their position according to how much time passes between key presses.     
It also supports different icons, depending on the status of the door.    
It will work with doors with a single switch that will cyce within modes: Open - Stop - Close etc.  
This is for two doors, if you need only one, you can remove half of the code

**Switches**
- For this to work, you need to install a momentary switch that will immitate the button press of your RF remote.
- I used two sonoff basics flashed with tasmota and hooked them up inside the garage door machine, paralel to the RF remote signal.
   - On the [tasmota FW](https://github.com/arendst/Sonoff-Tasmota/wiki) you need to use some some `pulsetime` [command](https://github.com/arendst/Sonoff-Tasmota/wiki/Commands) on the console, and set it to somewhere between 200-1000ms.
   - You also need to set  `poweronstate` to 0.
   - This code will not work with "non-self-inching" relays as the off command is handled on the tasmota (safer imo).
   - It should be easy to modify the code inside the scripts part for HA to handle the automatic OFF command (let me know)
   - The sonoffs originally provide 220v to their output. I modded the boards [like this](https://community.home-assistant.io/t/modded-sonoff-using-the-on-board-relay-as-switch-for-any-circuit/50343) so that they would provide dry contacts and replicate the button presses of the garage door switches (i think they are 12v)


**Installation on HA**  
- Copy these files inside your `config/packages/garage` folder of your HA configuration. (it's better to download as zip then copy/paste them - or make sure you select the raw version of the file if downloading individually)
- Copy the images found in `/packages/garage/extras/icons/` to your `www/images/icons/ dir`. (if it's not there create it, or change the url in the code)
- Enable [Packages on Home-Assistant](https://www.home-assistant.io/docs/configuration/packages/) by adding this code to your `onfiguration.yaml`(under `homeassistant:`)
```
homeassistant:
...
packages: !include_dir_named packages/   
...
```
- My code works with the switches named as `switch.sonoff_garage_1` and `switch.sonoff_garage_2`. Here you have the following choices:
   - Rename your switches to comply to my code (if you dont use the switches elsewhere)
   - find/replace all occurances of my names and change them to yours 
   - Create template switches named `switch.sonoff_garage_1` and have them just control your existing switches (the simplest imo). Like this:
   ```
   switch:
  - platform: template
    switches:
      sonoff_garage_1:
        value_template: "{{ states('switch.your_garage_switch') }}"
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.your_garage_switch
        turn_off:
          service: switch.turn_off
          data:
            entity_id: switch.your_garage_switch
   ### optionally, repeat for sonoff_garage_2
  - platform: template
    switches:
      sonoff_garage_2:
        value_template: "{{ states('switch.your_garage_switch_2') }}"
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.your_garage_switch_2
        turn_off:
          service: switch.turn_off
          data:
            entity_id: switch.your_garage_switch_2
   ```
- Get a timer and time the time it takes for your garage door to open or close. In my case it was exactly the same (they are sliding doors) and the code only supports this. If your doors take different time to open than to close, it won't work.
- Then go to the file `garage_variables.yaml` and change the numbers in seconds accordingly:
   ```
   variable:
     garage_up_total_time:
       value: 16 #change this according to your doors (in seconds)
     garage_up_position:
       value: 0
     garage_up_last_use:
       value: 0
     garage_up_prev_use:
       value: 0
   ```
- For the whole thing to work you need to install and initialize custom HA component `Variables` https://github.com/rogro82/hass-variables

- The notification code (inside the automations file) is set to notify when a door has been left open and to work with `html5 notification component` so if you are using something else go there and change it accordingly
   ```
   ...
       condition: []
       action:
       - data:
           message: Garage UP left Open {{ now().strftime('%H:%M')}}
           title: Home Security
         service: notify.notify_html5 #change this part according to your notification provider.
   ...
   ```
- UI: you will find my ui code inside /packages/garage/extras/ui-lovelace.yaml. You need to turn on the raw editor and paste this inside a card in your UI. For the UI to work as in my example you also need to istall these two: https://github.com/thomasloven/lovelace-fold-entity-row and https://github.com/thomasloven/lovelace-toggle-lock-entity-row
- Restart HA and with your doors closed, change the dropdown "Garage XX State" to closed.
- You are good to go (i think) :)


- Lastly, for it to work correctly you need to avoid using other sources (like rf-remotes) to control your doors. If you send a command to your door manually the code will not be able to calculate its position. The same thing will happen if for any reason your door stops its operation manually, if it is equipped with an automatic obstacle detection or if ti faces some force, it will stop and will not update its state in HA. In that case, i have included dropdown menus where you can set the realtime position of your door manuall and take it from there.

**Dependancies**:
- https://github.com/rogro82/hass-variables
- https://github.com/thomasloven/lovelace-fold-entity-row
- https://github.com/thomasloven/lovelace-toggle-lock-entity-row

**Disclaimer**: This is not professional code, i am an amateur coder that just wanted to share his project mainly to get feedback. I will not be responsible if your HA crashes or if your garage doors stay open at night, so use at your own risk and be careful if you dont know what you are doing. (I don't :p)
