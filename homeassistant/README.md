# Home Assistant Guides, Templates, Tips and Tricks!
This is a work in progress, check back for more!

## Navigation
* [Apps](https://github.com/TechHutTV/homelab/tree/main/apps) - List of all the apps and services.
* [__Home Assistant__](https://github.com/TechHutTV/homelab/tree/main/homeassistant) - Smart home services and automation.
  - [Dashboard](https://github.com/TechHutTV/homelab/tree/main/homeassistant/dashboard)
  - [LocalTuya](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya)
  - [Zigbee](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee)
* [Media Server](https://github.com/TechHutTV/homelab/tree/main/media) - Plex, Jellyfin, *arr stack, and more.
* [Server Monitoring](https://github.com/TechHutTV/homelab/tree/main/monitoring) - Graphs and Visualizations for Unriad, Proxmox, and more.
* [Surveillance System](https://github.com/TechHutTV/homelab/tree/main/surveillance) - Frigate NVR Solution with Coral TPU.
* [Storage](https://github.com/TechHutTV/homelab/tree/main/storage) - Current Storage and Backup Solution.
* [Proxy Managment](https://github.com/TechHutTV/homelab/tree/main/proxy) - NGINX Proxy Manager, DDNS with Cloudflare, Local Domains, and more.

To do
* [https://github.com/custom-cards/upcoming-media-card](https://github.com/custom-cards/upcoming-media-card)
* [https://community.home-assistant.io/t/tautulli-rest-end-point-and-custom-button-card/559371](https://community.home-assistant.io/t/tautulli-rest-end-point-and-custom-button-card/559371)

## Bubble Cards 
I'm starting to utilze Bubble Cards for my dashboard. I'll add more information soon, but first my custom card for shades with set_postition varibles on the sub-buttons.

```
type: custom:bubble-card
card_type: cover
entity: cover.bedroom_window_shade
name: Bedroom
icon_open: mdi:roller-shade
icon_close: mdi:roller-shade-closed
show_state: false
show_last_changed: false
show_attribute: true
attribute: current_position
card_layout: normal
hold_action: {}
sub_button:
  - entity: cover.bedroom_window_shade
    name: Close
    show_icon: false
    show_name: true
    tap_action:
      action: call-service
      target:
        entity_id: cover.bedroom_window_shade
      data:
        position: 2
      service: cover.set_cover_position
    show_last_changed: false
    show_attribute: false
    show_state: false
  - entity: cover.bedroom_window_shade
    name: "15"
    show_icon: false
    show_name: true
    tap_action:
      action: call-service
      target:
        entity_id: cover.bedroom_window_shade
      data:
        position: 15
      service: cover.set_cover_position
  - entity: cover.bedroom_window_shade
    name: "50"
    show_icon: false
    show_name: true
    tap_action:
      action: call-service
      target:
        entity_id: cover.bedroom_window_shade
      data:
        position: 50
      service: cover.set_cover_position
  - entity: cover.bedroom_window_shade
    name: "75"
    show_icon: false
    show_name: true
    tap_action:
      action: call-service
      target:
        entity_id: cover.bedroom_window_shade
      data:
        position: 75
      service: cover.set_cover_position
grid_options:
  columns: 12
  rows: 2
styles: |2-
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
scrolling_effect: false
```

## LocalTuya in Home Assistant
When I first tired integrating XMCOSY String Lights with LocalTuya I followed a few different [guides](https://thehelpfulidiot.com/integrating-xmcosy-outdoor-string-lights-with-home-assistant-locally), but had no luck. The main issue ended up being LocalTuya itself. I found that using [this fork](https://github.com/xZetsubou/localtuya) works great and it's more up to date. This quick guide assumes you have the Tuya app setup with your devices added. Also, you'll need a [Tuya developer account](https://platform.tuya.com/) to get this setup.

#### Install and Setup LocalTuya Fork
1. Open HACS and navigate to Integrations Section.
2. Open the Overflow Menu (⋮) in the top right corner and click on Custom repositories.
3. Paste ```https://github.com/xZetsubou/localtuya``` and select Integration from the category dropdown then click ADD.
4. Now the integration should be added search in for it and install it, then restart Home Assistant.

#### Credentials from Tuya IoT Platform
1. Sign in and create a new project slecting Smart Home for both Industry and Development Method. Next, on the API options add 'Device Log Query' as you may need it to see important information about your devices.
2. Within the project overview page select 'Devices' and then 'Link App Account'. Use the sanner in the settings of your Tuya app to scan the QR code. Once you athenticate your devices should appear under 'All Devices'. If you see Read under 'Device Permission' you need to change this so it's listed as Controllable.
3. Now let's collect our IDs and Keys to connect LocalTuya. Under the project overview tab you will see 'Authorization Key'. You will need to save both the Client ID and Client Secret. Next go back to Devices and then 'Link App Account' and save the UID as this will go under User ID in the next step.
4. Within Home Assisant nagivate to the newly added LocalTuya integration. Click on 'Configure' and then 'Manage Cloud API account'. Paste in all the details from your Tuya project dashboard. Click submit and you should be good to go.

#### Adding Tuya devices in Home Assistant
1. Click on 'Configure' again, but this time select 'Add new device' and select the device you'd like to add. Manual configuration is a pain, but if you have your devices on a different Vlan this may be required.
2. Next, you'll get a 'Configure device connectivity' option. In most cases the pre-filled options will be perfect and there will not be anything you need to change here.
#### Manual condfigureation tips
If you need to manually connect to a device an it's important to know where to find you local ids and values. These can be found within your project on the Tuya IoT Platform. For steps within the API Explorer copy the Device ID found under All Devices.
* local_key: API Explorer > Query Device Details in Bulk > Input Device > See string after "local_key"
* entity ids: API Explorer > Query Properties > Input Device > See number after "dp_id"
* min/max values: Devices > All Devices > Debug Device > Device Debugging > Standard Instuction Set

### XMCOSY String Lights with LocalTuya

#### Finding Raw Data Values for Scenes
You can use the Tuya app to create custom or access pre-exisiting scenes. Open the developer portal, go to devices and click on "debug device". Open then device logs page and select the 'scene' logs from the dropdown menu on the top and click search. This should provide a overview of all scenes that were activated.

```
# XMcosy RGBCW String Lights, 24 bulbs, 64ft Acrylic Holiday Lights
ASMJMgkAAABkAABkAAAAAGQAPGQAAAAAZAC7VAAAAABkAOpXAAAAAGQBCGMAAAAA: Christmas
AVATMhkAAABkALxkAAAAAGQALU4AAAAAZAAAZAAAAABkAGQ8AAAAAA==: Christmas eve
ASkJMk8AAABkAN1LAAAAAGQAvVMAAAAAZADYXQAAAAA: Fathers Day
AQEDMjIAAABkAUVkAAAAAGQBIjkAAAAAZAEeZAAAAABkATpfAAAAAGQAAWQAAAAAZAAnWgAAAAA=: Flower harbor
ASoJMgoAAABkAAFkAAAAAGQAiGQAAAAAZAC7ZAAAAAA: Football Day
AScJMiQAAABkAJVbAAAAAGQAtGQAAAAAZACRYQAAAAA: Forest Day
ASUKMjIAAABkAApfAAAAAGQBBmQAAAAAZADvXAAAAABkALAsAAAAAGQAfFQAAAAA: Halloween
ASgKMgcAAABkAU49AAAAAGQBF1UAAAAAZAEMOwAAAAA: Mothers Day
ASYUMjIAAABkADxVAAAAAGQBD2QAAAAAZADNQwAAAABkABZgAAAAAA: Thanksgiving
ASQMMjIAAABkARJkAAAAAGQA71cAAAAAZAFBUwAAAABkAVJkAAAAAA: Valentines
```
__Wish List__: I feel like these could be possible with python scripts if I figure out how to decode that raw scene string to something a little more human friendly. I've reached out to XMCOSY, but I'm waiting on them to get back to me.
- [ ] Per-bulb control with a custom card
- [ ] Ability to save current configuration as a scene directly in HA
- [ ] Mode option in HA (pulse, fade, etc.)

## Other Devices
One of my switches wasn't pulling the actions or entities, [this blueprint](https://community.home-assistant.io/t/zha-aqara-wireless-mini-switch/255540) seems to fix it.
