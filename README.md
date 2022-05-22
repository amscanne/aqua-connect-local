# Home Assistant configuration for the Aqua Connect Local Gateway

Note that my local gateway has an address of `192.168.3.90`, you'll need to replace that below. I am not sure to the extent the rest of the `KeyId`s are all standardized, etc.

```yaml
sensor:
  - platform: rest
    name: pool_panel
    resource: http://192.168.3.90/WNewSt.htm
    scan_interval: 5
    value_template: "{{ value | regex_findall(' *(.*[^ ]) *xxx') | join(' ') }}"
  - platform: template
    sensors:
      pool_temperature:
        friendly_name: "Water Temperature"
        attribute_templates:
          minute_last_updated: >-
            {% set panel = states('sensor.pool_panel') %}
            {% if panel is search('(Pool|Spa) Temp') %} {{ now().minute }} {% else %} {{ state_attr('sensor.pool_temperature', 'minute_last_updated') }} {% endif %}
        availability_template: >-
          {% if now().minute - state_attr('sensor.pool_temperature', 'minute_last_updated') < 10 %} true {% else %} false {% endif %}
        value_template: >-
          {% set panel = states('sensor.pool_panel') %}
          {% if panel is search('(Pool|Spa) Temp') %} {{ panel | regex_findall_index(' Temp +(\d+)') }} {% else %} {{ states('sensor.pool_temperature') }} {% endif %}
        device_class: temperature
        unit_of_measurement: "F"
      air_temperature:
        friendly_name: "Air temperature"
        value_template: >-
          {% set panel = states('sensor.pool_panel') %}
          {% if panel is search('Air Temp') %} {{ panel | regex_findall_index('Air Temp +(\d+)') }} {% else %} {{ states('sensor.air_temperature') }} {% endif %}
        device_class: temperature
        unit_of_measurement: "F"
switch:
  - platform: template
    switches:
      pool_spa:
        friendly_name: "Spa"
        value_template: "{{ states('sensor.pool_panel') is search(' +[5EUe][^ ]*$') }}"
        turn_on:
          service: "rest_command.pool_panel"
          data:
            KeyId: "07"
        turn_off:
          service: "rest_command.pool_panel"
          data:
            KeyId: "06"
      pool_filter:
        friendly_name: "Filter"
        value_template: "{{ states('sensor.pool_panel') is search(' +.[5EUe][^ ]*$') }}"
        turn_on:
          service: "rest_command.pool_panel"
          data:
            KeyId: "08"
        turn_off:
          service: "rest_command.pool_panel"
          data:
            KeyId: "08"
      pool_lights:
        friendly_name: "Lights"
        value_template: "{{ states('sensor.pool_panel') is search(' +..[STUV][^ ]*$') }}"
        turn_on:
          service: "rest_command.pool_panel"
          data:
            KeyId: "09"
        turn_off:
          service: "rest_command.pool_panel"
          data:
            KeyId: "09"
      pool_heater:
        friendly_name: "Heater"
        value_template: "{{ states('sensor.pool_panel') is search(' +...[STUV][^ ]*$') }}"
        turn_on:
          service: "rest_command.pool_panel"
          data:
            KeyId: "13"
        turn_off:
          service: "rest_command.pool_panel"
          data:
            KeyId: "13"
rest_command:
  pool_panel:
    url: http://192.168.3.90/WNewSt.htm
    method: POST
    payload: "KeyId={{KeyId}}"
```
