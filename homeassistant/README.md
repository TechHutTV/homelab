## Integrating XMCOSY String Lights with LocalTuya
First time I tired to do this I followed a few different [guides](https://thehelpfulidiot.com/integrating-xmcosy-outdoor-string-lights-with-home-assistant-locally), but had no luck. The main issue ended up being LocalTuya itself. I found that using [this fork](https://github.com/xZetsubou/localtuya) works great and it's more up to date. This quick guide assumes you have the Tuya app setup with your devices added. Also, you'll need a [Tuya developer account](https://platform.tuya.com/) to get this setup.

### Install and Setup LocalTuya Fork
1. Open HACS and navigate to Integrations Section.
2. Open the Overflow Menu (⋮) in the top right corner and click on Custom repositories.
3. Paste https://github.com/xZetsubou/localtuya into the input field and select Integration from the category dropdown then click ADD.
4. Now the integration should be added search in for it and install it, then restart Home Assistant.

### Credentials from Tuya IoT Platform
1. Sign in and create a new project slecting Smart Home for both Industry and Development Method. Next, on the API options add 'Device Log Query' as you may need it to see important information about your devices.
2. Within the project overview page select 'Devices' and then 'Link App Account'. Use the sanner in the settings of your Tuya app to scan the QR code. Once you athenticate your devices should appear under 'All Devices'. If you see Read under 'Device Permission' you need to change this so it's listed as Controllable.
3. Now let's collect our IDs and Keys to connect LocalTuya. Under the project overview tab you will see 'Authorization Key'. You will need to save both the Client ID and Client Secret. Next go back to Devices and then 'Link App Account' and save the UID as this will go under User ID in the next step.
4. Within Home Assisant nagivate to the newly added LocalTuya integration.

   ...wip
