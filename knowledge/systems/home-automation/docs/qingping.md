<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Qingping

## Setup new Airmonitor

### Bootup

Bootup the new device and connect it to the iot network.
Wait for the device to show up in the network (approx. 5 minutes).

Check for new Firmware updates in Settings > System update.

### Add device to Qingping IOT App

Goto Settings > Qingping IoT App.

Open Qingping IOT App on your phone and add the device.
Enter the verification code shown on the device.
Name your device, `qingping-<room>`. Do not add a group.
The device will show "Connected".

### Add device to Qingping Developer Portal

Goto <https://developer.qingping.co/private/device-binding> and login.
Goto Access Configuration and add new configuration.

```config
Name: qingping-<room>
Device Model: Qingping Air Monitor
Private Type: Self-built MQTT
Self-built MQTT Information:
  Host: mqtt.home
  Port: 1883
```

Goto Device Binding and add new device.
Select device model and the device and configuration recently created.

### Add device to network and add firewall rules

Goto <https://opnsense.techtales.io>.
Goto Services > ISC DHCPv4 > Leases and check for the new device in the iot network.
Name should be ``snow2` or something, double check the MAC address on the device Settings > About.
Configure static IP, see similar entries.
Hit "Save" and "Apply changes" on top!

Goto <https://github.com/techtales-io/terraform-opnsense>.
Check in data/firewall-filters for existing rules and add a new rule for the new device.
Create PR and let atlantis do the work!

### Add device to Home Assistant

Goto <https://hass.techtales.io>.
Goto Settings > Devices & Services > Integrations > Qingping Pro AQM and "add entry".
If device is not found, add manually with "Submit".
Add device name and Mac address (upper case and without colons, f.e. ABCD12345678).
Change tvoc to ppb.
