Wip

### XMCOSY+ Outdoor String Lights (will move this soon)
Source: [thehelpfulidiot.com](https://thehelpfulidiot.com/integrating-xmcosy-outdoor-string-lights-with-home-assistant-locally)
```
{
  dps: {
    '20': false, #on/off
    '21': 'white', #scene
    '22': 255, #brightness
    '23': 165, #color temp
    '24': '23eb230159ff38', #lights hex code AND HSV
    '101': 'ffff0502808080ffff00' #a string storing the customizable two colors for scene8 
  }
}
```
Create LocalTuya entities:

* light.colored_string_lights – the main light entity
* sensor.custom_colors – a sensor displaying the hex code for the two custom colors used for scene8
* sensor.string_light_color – brightness 
* sensor.string_light_color_value – hex and hsv color/brightness string
* sensor.string_lights_scene – the current scene

Templates
```
  - platform: template
    lights:
      string_lights:
        level_template: >
          {% if state_attr('light.colored_string_lights','color_mode') == 'color_temp' %}
            {{ states('sensor.string_light_color')|int }}
          {% elif state_attr('light.colored_string_lights','color_mode') == 'hs' %}
            {{ states('sensor.string_light_color_value')[12:]|int(base=16) }}
          {% else %}
            255
          {% endif %}
        value_template: "{{ states('light.colored_string_lights') }}"
        temperature_template: >
          {% if state_attr('light.colored_string_lights','color_mode') == 'color_temp' %}
            {{ state_attr('light.colored_string_lights','color_temp') }}
          {% elif state_attr('light.colored_string_lights','color_mode') == 'hs' %}
            {{ color_temp|int(default=0) }}
          {% endif %}
        color_template: >
          {% if state_attr('light.colored_string_lights','color_mode') == 'hs' %}
            {{ state_attr('light.colored_string_lights','hs_color') }}
          {% endif %}
        effect_list_template: "{{['White','Color','Strobe','Fade','Pulse','Blink','Chase','Christmas','America','Pick 2','Multi']}}"
        effect_template: >
          {% if states('sensor.string_lights_scene') == 'scene1' %}
            {{ "Strobe" }}
          {% elif states('sensor.string_lights_scene') == 'scene2' %}
            {{ "Fade" }}
          {% elif states('sensor.string_lights_scene') == 'scene3' %}
            {{ "Pulse" }}
          {% elif states('sensor.string_lights_scene') == 'scene4' %}
            {{ "Blink" }}
          {% elif states('sensor.string_lights_scene') == 'scene5' %}
            {{ "Chase" }}
          {% elif states('sensor.string_lights_scene') == 'scene6' %}
            {{ "Christmas" }}
          {% elif states('sensor.string_lights_scene') == 'scene7' %}
            {{ "America" }}
          {% elif states('sensor.string_lights_scene') == 'scene8' %}
            {{ "Pick 2" }}
          {% elif states('sensor.string_lights_scene') == 'scene9' %}
            {{ "Multi" }}
          {% elif states('sensor.string_lights_scene') == 'white' %}
            {{ "White" }}
          {% elif states('sensor.string_lights_scene') == 'colour' %}
            {{ "Color" }}
          {% endif %}
        min_mireds_template: "{{ state_attr('light.colored_string_lights','min_mireds') }}"
        max_mireds_template: "{{ state_attr('light.colored_string_lights','max_mireds') }}"
        turn_on:
          service: light.turn_on
          target:
            entity_id: light.colored_string_lights
        turn_off:
          service: light.turn_off
          target:
            entity_id: light.colored_string_lights
        set_level:
          - service: script.brightness_with_white_or_color
            data:
              brightness: '{{ brightness|int }}'
        set_color:
          - service: script.check_color_mode
            data:
              h: '{{ hs[0] }}'
              s: '{{ hs[1] }}'
        set_temperature:
          - service: script.check_white_mode
            data:
              color_temp: '{{color_temp|int }}'
        set_effect:
          - service: localtuya.set_dp
            data:
              device_id: eb3f15638dccc2fe59wdgj
              dp: 21
              value: >
                {% if effect == 'Strobe' %}
                  {{ "scene1" }}
                {% elif effect == 'Fade' %}
                  {{ "scene2" }}
                {% elif effect == 'Pulse' %}
                  {{ "scene3" }}
                {% elif effect == 'Blink' %}
                  {{ "scene4" }}
                {% elif effect == 'Chase' %}
                  {{ "scene5" }}
                {% elif effect == 'Christmas' %}
                  {{ "scene6" }}
                {% elif effect == 'America' %}
                  {{ "scene7" }}
                {% elif effect == 'Pick 2' %}
                  {{ "scene8" }}
                {% elif effect == 'Multi' %}
                  {{ "scene9" }}
                {% elif effect == 'White' %}
                  {{ "white" }}
                {% elif effect == 'Color' %}
                  {{ "colour" }}
                {% endif %}
```
```
alias: check white mode
sequence:
  - if:
      - condition: not
        conditions:
          - condition: state
            entity_id: sensor.string_lights_scene
            state: white
    then:
      - service: localtuya.set_dp
        data:
          device_id: eb3f15638dccc2fe59wdgj
          dp: 21
          value: white
  - service: light.turn_on
    data:
      color_temp: '{{ color_temp }}'
    entity_id: light.colored_string_lights
mode: single
```
```
alias: check color mode
sequence:
  - if:
      - condition: not
        conditions:
          - condition: state
            entity_id: sensor.string_lights_scene
            state: colour
    then:
      - service: localtuya.set_dp
        data:
          device_id: eb3f15638dccc2fe59wdgj
          dp: 21
          value: colour
      - wait_for_trigger:
          - platform: state
            entity_id:
              - sensor.string_lights_scene
            to: colour
        timeout: '1'
  - service: light.turn_on
    data:
      hs_color:
        - '{{ h }}'
        - '{{ s }}'
    entity_id: light.colored_string_lights
```
```
alias: brightness with white or color
sequence:
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {{ state_attr('light.colored_string_lights','color_mode') ==
              'color_temp' }}
        sequence:
          - service: localtuya.set_dp
            data:
              device_id: eb3f15638dccc2fe59wdgj
              dp: 22
              value: '{{brightness|int}}'
    default:
      - service: rest_command.color_converter
        data:
          brightness: '{{brightness|int}}'
      - service: localtuya.set_dp
        data:
          device_id: eb3f15638dccc2fe59wdgj
          dp: 24
          value: >
            {{ states('sensor.color_output') }}{{
            states('sensor.string_light_color_value')[6:12] }}{{ "%0x" | format(
            brightness | int ) }}
mode: single
```
```
  change_color_not_brightness:
    url: http://nodered:1880/color-converter
    method: POST
    payload: '{"hsv":{"hue":{{ h }},"saturation":{{ s }},"brightness":{{ (state_attr(''light.string_lights'',''brightness'')/2.55)|int}}}}'
    content_type:  'application/json; charset=utf-8'
    verify_ssl: false
```
Automations
```
alias: Change string light colors
description: ''
trigger:
  - platform: state
    entity_id: input_select.color_1
    id: color1
  - platform: state
    entity_id: input_select.color_2
    id: color2
condition: []
action:
  - choose:
      - conditions:
          - condition: trigger
            id: color1
        sequence:
          - service: localtuya.set_dp
            data:
              device_id: eb3f15638dccc2fe59wdgj
              dp: 101
              value: >
                {% if states('input_select.color_1') == 'Black' %}
                  {% set new1 = '000000' %}
                {% elif states('input_select.color_1') == 'Silver' %}
                  {% set new1 = 'c0c0c0' %}
                {% elif states('input_select.color_1') == 'Gray' %}
                  {% set new1 = '808080' %}
                {% elif states('input_select.color_1') == 'White' %}
                  {% set new1 = 'ffffff' %}
                {% elif states('input_select.color_1') == 'Maroon' %}
                  {% set new1 = '800000' %}
                {% elif states('input_select.color_1') == 'Red' %}
                  {% set new1 = 'ff0000' %}
                {% elif states('input_select.color_1') == 'Purple' %}
                  {% set new1 = '800080' %}
                {% elif states('input_select.color_1') == 'Fuchsia' %}
                  {% set new1 = 'ff00ff' %}
                {% elif states('input_select.color_1') == 'Green' %}
                  {% set new1 = '008000' %}
                {% elif states('input_select.color_1') == 'Lime' %}
                  {% set new1 = '00ff00' %}
                {% elif states('input_select.color_1') == 'Olive' %}
                  {% set new1 = '808000' %}
                {% elif states('input_select.color_1') == 'Yellow' %}
                  {% set new1 = 'ffff00' %}
                {% elif states('input_select.color_1') == 'Navy' %}
                  {% set new1 = '000080' %}
                {% elif states('input_select.color_1') == 'Blue' %}
                  {% set new1 = '0000ff' %}
                {% elif states('input_select.color_1') == 'Teal' %}
                  {% set new1 = '008080' %}
                {% elif states('input_select.color_1') == 'Aqua' %}
                  {% set new1 = '00ffff' %}
                {% endif %}

                {{ 'ffff0502' }}{{new1}}{{
                states('sensor.custom_colors').split('ffff0502')[1][6:] }}
      - conditions:
          - condition: trigger
            id: color2
        sequence:
          - service: localtuya.set_dp
            data:
              device_id: eb3f15638dccc2fe59wdgj
              dp: 101
              value: >
                {% if states('input_select.color_2') == 'Black' %}
                  {% set new2 = '000000' %}
                {% elif states('input_select.color_2') == 'Silver' %}
                  {% set new2 = 'c0c0c0' %}
                {% elif states('input_select.color_2') == 'Gray' %}
                  {% set new2 = '808080' %}
                {% elif states('input_select.color_2') == 'White' %}
                  {% set new2 = 'ffffff' %}
                {% elif states('input_select.color_2') == 'Maroon' %}
                  {% set new2 = '800000' %}
                {% elif states('input_select.color_2') == 'Red' %}
                  {% set new2 = 'ff0000' %}
                {% elif states('input_select.color_2') == 'Purple' %}
                  {% set new2 = '800080' %}
                {% elif states('input_select.color_2') == 'Fuchsia' %}
                  {% set new2 = 'ff00ff' %}
                {% elif states('input_select.color_2') == 'Green' %}
                  {% set new2 = '008000' %}
                {% elif states('input_select.color_2') == 'Lime' %}
                  {% set new2 = '00ff00' %}
                {% elif states('input_select.color_2') == 'Olive' %}
                  {% set new2 = '808000' %}
                {% elif states('input_select.color_2') == 'Yellow' %}
                  {% set new2 = 'ffff00' %}
                {% elif states('input_select.color_2') == 'Navy' %}
                  {% set new2 = '000080' %}
                {% elif states('input_select.color_2') == 'Blue' %}
                  {% set new2 = '0000ff' %}
                {% elif states('input_select.color_2') == 'Teal' %}
                  {% set new2 = '008080' %}
                {% elif states('input_select.color_2') == 'Aqua' %}
                  {% set new2 = '00ffff' %}
                {% endif %}

                {{ 'ffff0502' }}{{
                states('sensor.custom_colors').split('ffff0502')[1][:6] }}{{
                new2 }}
    default: []
mode: single
```
```
- sensor:
      - name: "Custom Color 2"
        state: >
          {% if states('sensor.custom_colors').split('ffff0502')[1][6:] == '000000' %}
            {{ 'Black' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == 'c0c0c0' %}
            {{ 'Silver' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '808080' %}
            {{ 'Gray' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == 'ffffff' %}
            {{ 'White' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '800000' %}
            {{ 'Maroon' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == 'ff0000' %}
            {{ 'Red' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '800080' %}
            {{ 'Purple' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == 'ff00ff' %}
            {{ 'Fuchsia' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '008000' %}
            {{ 'Green' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '00ff00' %}
            {{ 'Lime' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '808000' %}
            {{ 'Olive' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == 'ffff00' %}
            {{ 'Yellow' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '000080' %}
            {{ 'Navy' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '0000ff' %}
            {{ 'Blue' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '008080' %}
            {{ 'Teal' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][6:] == '00ffff' %}
            {{ 'Aqua' }}
          {% else %}
            {{ states('sensor.custom_colors').split('ffff0502')[1][6:] }}
          {% endif %}
- sensor:
      - name: "Custom Color 1"
        state: >
          {% if states('sensor.custom_colors').split('ffff0502')[1][:6] == '000000' %}
            {{ 'Black' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == 'c0c0c0' %}
            {{ 'Silver' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '808080' %}
            {{ 'Gray' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == 'ffffff' %}
            {{ 'White' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '800000' %}
            {{ 'Maroon' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == 'ff0000' %}
            {{ 'Red' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '800080' %}
            {{ 'Purple' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == 'ff00ff' %}
            {{ 'Fuchsia' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '008000' %}
            {{ 'Green' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '00ff00' %}
            {{ 'Lime' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '808000' %}
            {{ 'Olive' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == 'ffff00' %}
            {{ 'Yellow' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '000080' %}
            {{ 'Navy' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '0000ff' %}
            {{ 'Blue' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '008080' %}
            {{ 'Teal' }}
          {% elif states('sensor.custom_colors').split('ffff0502')[1][:6] == '00ffff' %}
            {{ 'Aqua' }}
          {% else %}
            {{ states('sensor.custom_colors').split('ffff0502')[1][:6] }}
          {% endif %}
```

