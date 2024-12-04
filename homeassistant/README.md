# Home Assistant Guides, Templates, Tips and Tricks!
This is a work in progress, check back for more!

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

Working on creating templates based on [this](https://www.reddit.com/r/homeassistant/comments/1dt1ngp/tuya_scenes_in_home_assistant_for_rgbcw_light/).

```
# XMcosy RGBCW String Lights, 24 bulbs, 64ft Acrylic Holiday Lights
Christmas: ASMJMgkAAABkAABkAAAAAGQAPGQAAAAAZAC7VAAAAABkAOpXAAAAAGQBCGMAAAAA
Christmas eve: AVATMhkAAABkALxkAAAAAGQALU4AAAAAZAAAZAAAAABkAGQ8AAAAAA==
Fathers Day: ASkJMk8AAABkAN1LAAAAAGQAvVMAAAAAZADYXQAAAAA
Flower harbor: AQEDMjIAAABkAUVkAAAAAGQBIjkAAAAAZAEeZAAAAABkATpfAAAAAGQAAWQAAAAAZAAnWgAAAAA=
Football Day: ASoJMgoAAABkAAFkAAAAAGQAiGQAAAAAZAC7ZAAAAAA
Forest Day: AScJMiQAAABkAJVbAAAAAGQAtGQAAAAAZACRYQAAAAA
Halloween: ASUKMjIAAABkAApfAAAAAGQBBmQAAAAAZADvXAAAAABkALAsAAAAAGQAfFQAAAAA
Mothers Day: ASgKMgcAAABkAU49AAAAAGQBF1UAAAAAZAEMOwAAAAA
Thanksgiving: ASYUMjIAAABkADxVAAAAAGQBD2QAAAAAZADNQwAAAABkABZgAAAAAA
Valentines: ASQMMjIAAABkARJkAAAAAGQA71cAAAAAZAFBUwAAAABkAVJkAAAAAA
```
