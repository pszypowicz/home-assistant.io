---
title: Ampio
description: Instructions on how to integrate Ampio Smart Home with Home Assistant.
ha_category:
  - Hub
  - Sensor
ha_release: 2026.6
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@pszypowicz'
ha_domain: ampio
ha_integration_type: hub
ha_platforms:
  - diagnostics
  - sensor
ha_quality_scale: bronze
---

The **Ampio** {% term integration %} lets you read sensors and module status from an [Ampio Smart Home](https://ampio.com/) installation through the Ampio M-SERV controller. The integration connects directly to the M-SERV's local MQTT broker, so values arrive in Home Assistant as soon as the bus reports them, without any cloud round trip.

## Supported devices

The integration discovers modules and DB objects from the M-SERV at setup time. Modules report their hardware type, and the integration resolves that to a human-readable model name.

The following module families are confirmed to work today as sources of sensor data:

- M-SENS environmental sensors (temperature, humidity, pressure, illuminance, sound pressure, CO2, indoor air quality).
- M-SERV controllers, which appear as the hub device for the rest of the installation.

Any other module that the M-SERV publishes through its DB object protocol still appears as a device, with a diagnostic *Last seen* timestamp so you can verify that the bus is reachable.

## Unsupported devices

This first release only exposes sensors. Switching modules (M-REL family), dimmers, blinds, RGBW, and DALI bridges are visible as devices but are not yet exposed as actionable entities.

## Prerequisites

Before adding the integration:

- Your Ampio M-SERV must be reachable from the Home Assistant host on its MQTT port (`1883` by default).
- You need an Ampio account on the M-SERV with permission to read the configuration. Restricted accounts that cannot read the device list are refused at setup, because the integration cannot derive a stable identity for the M-SERV without it.

{% include integrations/config_flow.md %}

{% configuration_basic %}
Host:
  description: "Hostname or IP address of your Ampio M-SERV controller."
Port:
  description: "MQTT port of the M-SERV. The default is `1883`."
Username:
  description: "Username of an Ampio account with permission to read the configuration."
Password:
  description: "Password for that Ampio account."
{% endconfiguration_basic %}

The integration verifies the credentials and reads the M-SERV identity during the configuration flow. If the M-SERV cannot report its identity, the flow shows an error rather than creating an entry with an unstable identifier.

## Supported functionality

### Sensors

For every classified sensor channel reported by the M-SERV, the integration creates a sensor entity with the appropriate device class, unit, and state class. The user-given object name from the Ampio configuration becomes the entity's friendly name.

For each module, the integration also creates a diagnostic *Last seen* timestamp sensor that updates every time any of the module's objects reports a state. This is useful for spotting bus partitions or a module that has dropped off without any other entity going unavailable.

## Data updates

The integration is push only. The M-SERV publishes object state changes on dedicated MQTT topics, and the integration translates each push into a Home Assistant state update. There is no polling and no user-configurable refresh interval.

If the connection to the M-SERV drops, all entities backed by the integration become unavailable. The integration reconnects automatically with capped exponential backoff and restores entities as soon as the bus is reachable again.

## Known limitations

- Only sensor objects are exposed in this release. Switching, dimming, blinds, RGBW, and DALI control are not implemented yet.
- The Ampio cloud is intentionally not used. The integration only talks to the local M-SERV broker.
- Object classification relies on the M-SERV's reported `typ_komponentu` and `interpretacja` fields. Channels whose interpretation the integration does not yet recognize appear as generic value-only sensors with no unit.

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
