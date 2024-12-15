## Navigation
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant)
  - [__Dashboard__](https://github.com/TechHutTV/homelab/tree/main/homeassistant/dashboard)
  - [LocalTuya](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya)
  - [Zigbee](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee)

Custom card compents used; [bubble card](https://github.com/Clooos/Bubble-Card), [mini graph card](https://github.com/kalkih/mini-graph-card).

## Pi-Hole

If you need to setup your Pi-Hole Integration check out [the docs](https://www.home-assistant.io/integrations/pi_hole/).

### Bubble Card with Disable Ads Timer
This is my config using the [bubble card](https://github.com/Clooos/Bubble-Card).
```
type: custom:bubble-card
card_type: button
entity: switch.pi_hole
button_type: state
sub_button:
  - entity: switch.pi_hole
    tap_action:
      action: call-service
      service: pi_hole.disable
      service_data:
        entity_id: switch.pi_hole
        duration: "00:05:00"
      target:
        entity_id: switch.pi_hole
    name: 5m
    show_name: true
    icon: mdi:block-helper
  - entity: switch.pi_hole
    tap_action:
      action: call-service
      service: pi_hole.disable
      service_data:
        entity_id: switch.pi_hole
        duration: "00:30:00"
      target:
        entity_id: switch.pi_hole
    name: 30m
    show_name: true
    icon: mdi:block-helper
  - entity: switch.pi_hole
    tap_action:
      action: call-service
      service: pi_hole.disable
      service_data:
        entity_id: switch.pi_hole
        duration: "00:01:00"
      target:
        entity_id: switch.pi_hole
    name: 60m
    show_name: true
    icon: mdi:block-helper
card_layout: large
scrolling_effect: false
force_icon: false
show_name: false
show_state: true
show_last_changed: true
tap_action:
  action: more-info
  service: ""
  target:
    entity_id: switch.pi_hole
styles: |2
    .is-on .bubble-button-card-container {
      background-color: #0000;
      border-radius: 10px;
  }
    .bubble-icon-container {
      opacity: 1 !important;
      background-color: ${state === 'on' ? 'green' : 'darkred'} !important;
  }
    .bubble-sub-button-1 {
      background: 00000;
    }
      .bubble-sub-button-2 {
      background: 00000;
    }
      .bubble-sub-button-3 {
      background: 00000;
    }
      .bubble-sub-button-4 {
      background: 00000;
    }
button_action:
  tap_action:
    action: toggle

```
### Mini Graph Card for Metrics
This uses the [mini graph card](https://github.com/kalkih/mini-graph-card).
```
type: horizontal-stack
cards:
  - type: custom:mini-graph-card
    name: Queries
    entities:
      - sensor.pi_hole_dns_queries_today
    hours_to_show: 3
    points_per_hour: 10
    update_interval: 10
    aggregate_func: max
    line_width: 8
    line_color: blue
    font_size: 70
  - type: custom:mini-graph-card
    name: Blocked
    entities:
      - sensor.pi_hole_ads_blocked_today
    hours_to_show: 3
    points_per_hour: 10
    update_interval: 10
    aggregate_func: max
    line_width: 8
    line_color: red
    font_size: 70
  - type: custom:mini-graph-card
    name: Difference
    entities:
      - sensor.pi_hole_ads_percentage_blocked_today
    hours_to_show: 3
    points_per_hour: 10
    update_interval: 10
    aggregate_func: max
    line_color: orange
    line_width: 8
    font_size: 70

```
