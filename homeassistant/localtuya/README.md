## Navigation
* [Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant)
  - [Dashboard](https://github.com/TechHutTV/homelab/tree/main/homeassistant/dashboard)
  - [__LocalTuya__](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya)
    - [LocalTuya in Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#localtuya-in-home-assistant)
      - [Install and Setup LocalTuya Fork](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#install-and-setup-localtuya-fork)
      - [Credentials from Tuya IoT Platform](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#credentials-from-tuya-iot-platform)
      - [Adding Tuya devices in Home Assistant](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#adding-tuya-devices-in-home-assistant)
      - [Manual configuration tips](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#manual-configuration-tips)
    - [XMCOSY String Lights with LocalTuya](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#xmcosy-string-lights-with-localtuya)
      - [Finding Raw Data Values for Scenes](https://github.com/TechHutTV/homelab/tree/main/homeassistant/localtuya#finding-raw-data-values-for-scenes)
  - [Zigbee](https://github.com/TechHutTV/homelab/tree/main/homeassistant/zigbee)

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
#### Manual configuration tips
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

Want to know more about these codes and how they work?  Read the documentation here: [Tuya Cloud Development Instruction Set](https://developer.tuya.com/en/docs/iot/dj?id=K9i5ql3v98hn3#title-16-scene_select).
To decode and Encode these strings, you can utilize the Base64 Encoder/Decoder available here: [Base 64 Encoder/Decoder](https://emn178.github.io/online-tools/base64_decode.html). Be sure to set your Input/Output Encoding to HEX.

__Wish List__: I feel like these could be possible with python scripts if I figure out how to decode that raw scene string to something a little more human friendly. I've reached out to XMCOSY, but I'm waiting on them to get back to me.
- [ ] Per-bulb control with a custom card
- [ ] Ability to save current configuration as a scene directly in HA
- [ ] Mode option in HA (pulse, fade, etc.)
