---
title: "Migrate from Community Edition to  Cloud"
description: ""
weight: 2
---

This section explains how to migrate your devices from [{{% tts %}} Community Edition]({{< ref "/getting-started/ttn" >}}) to [{{% tts %}} Cloud]({{< ref "/getting-started/cloud-hosted" >}}).

<!--more-->

You can migrate your devices from {{% tts %}} Community Edition to {{% tts %}} Cloud using {{% tts %}} [Console]({{< ref "/getting-started/console" >}}), the [CLI]({{< ref "/getting-started/cli" >}}) or the [migration tool]({{< ref "/getting-started/migrating/migration-tool" >}}), depending on if you want to migrate them with or without their existing session. Find your scenario and follow the instructions below.

## Prerequisites

1. A user account on {{% tts %}} Community Edition with an application containing registered devices.
2. A user account on {{% tts %}} Cloud.

We recommend testing migration on a single end device or a small batch of end devices in order to make sure the migration process goes as expected.

## Add Application in {{% tts %}} Cloud

You first need to add a new application in {{% tts %}} Cloud. See [Adding Applications]({{< ref "/integrations/adding-applications" >}}) for detailed instructions.

When adding an application in {{% tts %}} Cloud, the Application ID does not have to be the same as the one in {{% tts %}} Community Edition.

## Add Payload Formatters and Integrations in {{% tts %}} Cloud

After creating an application in {{% tts %}} Cloud, you need to add the associated elements like application-level payload formatters and integrations.

See [Payload Formatters]({{< ref "/integrations/payload-formatters" >}}) and [Integrations]({{< ref "/integrations" >}}) for more info.

## Migrate Devices without Persisting Active Sessions

Migrating a device without persisting its active session means the device will have to perform a join on {{% tts %}} Cloud network after its migration, i.e. it will establish a new session with {{% tts %}} Cloud. The device will negotiate about network parameters with {{% tts %}} Cloud Network Server, and during that negotiation, the device will be assigned with a new DevAddr from {{% tts %}} Cloud DevAddr block.

{{< note >}} Migrating devices without persisting active sessions is the preferred migration method. {{</ note >}}

Since {{% tts %}} Community Edition and {{% tts %}} Cloud are both connected to Packet Broker, Packet Broker will be able to route your device's traffic to {{% tts %}} Cloud even if your gateway stays connected to {{% tts %}} Community Edition, i.e. you don't have to migrate your gateway to {{% tts %}} Cloud, only your device. However, if you want to migrate your gateway to {{% tts %}} Cloud too, see [Migrating Gateways]({{< ref "/getting-started/migrating/gateway-migration" >}}) for instructions.

To migrate your devices from {{% tts %}} Community Edition to {{% tts %}} Cloud without active sessions, follow the steps described below.

{{< tabs/container "Console" "CLI">}}

{{< tabs/tab "Console" >}}

Keep in mind that this method is covenient only when you have a few devices to migrate. For larger groups of devices, we recommend using the CLI or the migration tool.

Recreate your device on {{% tts %}} Cloud. See [Adding Devices]({{< ref "/devices/adding-devices" >}}) for instructions on creating a device. You can reuse the DevEUI, AppEUI/JoinEUI and AppKey values from {{% tts %}} Community Edition. You can also generate new values, but then you will need to re-program your device using those values.

{{< /tabs/tab >}}

{{< tabs/tab "CLI" >}}

First, configure your CLI to connect to {{% tts %}} Community Edition. See [Configuring the CLI]({{< ref "/getting-started/cli/configuring-cli" >}}) guide for instructions. Make sure you also perform a [Login with the CLI]({{< ref "/getting-started/cli/login" >}}) to {{% tts %}} Community Edition.

{{< note >}} We recommend to use the latest version of the CLI. Instructions for upgrading the CLI if you already have it installed are available in the [Installing the CLI]({{< ref "/getting-started/cli/login" >}}) guide. {{</ note >}}

Now, use the CLI to export your device's description from {{% tts %}} Community Edition:

```bash
ttn-lw-cli end-devices get --application-id <app-id> --device-id <device-id> --name --description --lorawan-version --lorawan-phy-version --frequency-plan-id --supports-join --root-keys --mac-settings > device-description.json
```

The command above will export your device's description to the `device-description.json` file in the current folder. Open the file with a text editor and remove the following fields: `join_server_address`, `network_server_address` and `application_server_address`.

Next, you need to import the `device-description.json` file in your {{% tts %}} Cloud application. See instructions on how to [Import End Devices in {{% tts %}}]({{< ref "/getting-started/migrating/import-devices" >}}). Keep in mind that if you are using the CLI for import, you first have to re-configure it to connect to {{% tts %}} Cloud.

{{< /tabs/tab >}}

{{< /tabs/container >}}

When your device is registered or imported in {{% tts %}} Cloud as described above, you need to prevent it from re-joining {{% tts %}} Community Edition. To do so, you can change your device's AppKey on {{% tts %}} Community Edition (under device's **General settings &#8594; Join Settings &#8594; AppKey**) or completely delete your device from {{% tts %}} Community Edition.

Finally, your device needs to perform a new join on {{% tts %}} Cloud network. Some devices will perform an automatic re-join after they are deleted from {{% tts %}} Community Edition, but some devices need to be triggered, e.g. by sending a downlink to it, by power cycling it, etc. You should see a Join Request from your device coming in {{% tts %}} Cloud.

After joining {{% tts %}} Cloud network, you will see uplinks arriving from your device.

## Migrate Active Device Sessions

{{< note >}} Migrating devices without persisting active sessions is the preferred migration method. {{</ note >}}

Migrating an active device session means the device won't have to perform a join on {{% tts %}} Cloud network after its migration, i.e. the existing session that was established between the device and {{% tts %}} Community Edition will just be transferred to {{% tts %}} Cloud.

Note that {{% tts %}} Community Edition and {{% tts %}} Cloud use different DevAddr blocks. When an active device session is migrated from {{% tts %}} Community Edition to {{% tts %}} Cloud, the DevAddr that the device was assigned with when it joined {{% tts %}} Community Edition will be preserved (unlinke when you [Migrate Devices without Persisting Active Sessions]({{< ref "/getting-started/migrating/migrate-from-ce-to-ch#migrate-devices-without-persisting-active-sessions" >}})). Since Packet Broker routes traffic according to the DevAddr blocks, in this case it won't be able to route your device's traffic properly. Hence, to successfully migrate an active device session from {{% tts %}} Community Edition to {{% tts %}} Cloud, you also need to migrate your gateway to {{% tts %}} Cloud. See instructions for [Migrating Gateways]({{< ref "/getting-started/migrating/gateway-migration" >}}). The ideal scenario would be to migrate your gateway and your device simultaneously.

Since migrating an active session implies migrating a large number of parameters that cannot be configured manually, it is possible to do it only using the CLI or the migration tool.

To migrate active device sessions from {{% tts %}} Community Edition to {{% tts %}} Cloud, follow the steps described below.

First, configure your CLI to connect to {{% tts %}} Community Edition. See [Configuring the CLI]({{< ref "/getting-started/cli/configuring-cli" >}}) guide for instructions. Make sure you also perform a [Login with the CLI]({{< ref "/getting-started/cli/login" >}}) to {{% tts %}} Community Edition.

{{< note >}} We recommend to use the latest version of the CLI. Instructions for upgrading the CLI if you already have it installed are available in the [Installing the CLI]({{< ref "/getting-started/cli/login" >}}) guide. {{</ note >}}

Now, use the CLI to export your device's session from {{% tts %}} Community Edition:

```bash
ttn-lw-cli end-devices get --application-id <app-id> --device-id <device-id> --name --description --lorawan-version --lorawan-phy-version --frequency-plan-id --supports-join --root-keys --mac-settings --mac-state --session > device-session.json
```

The command above will export your device's session to the `device-session.json` file in the current folder. Open the file with a text editor and remove the following fields: `join_server_address`, `network_server_address` and `application_server_address`.

Before importing your device's session in {{% tts %}} Cloud, you need to completely delete your device from {{% tts %}} Community Edition to prevent conflicts.

Next, you need to import the `device-session.json` file in your {{% tts %}} Cloud application. See instructions on how to [Import End Devices in {{% tts %}}]({{< ref "/getting-started/migrating/import-devices" >}}). Keep in mind that if you are using the CLI for import, you first have to re-configure it to connect to {{% tts %}} Cloud.

When your device's session is imported in {{% tts %}} Cloud, assuming that your gateway is also migrated to {{% tts %}} Cloud, you will see uplinks arriving from your device.