# Set Up a DePHY Node

## 1. Preparation Before Initial Boot

* Power adapter
* HDMI cable
* Monitor
* Keyboard

### 2. Connect the Node

* Connect the HDMI cable to the monitor and the node
* Plug in the power adapter to the node
* The device will typically boot automatically when the power is connected. If it does not boot, press the round power button on the front panel to start it.

## 3. Log in

If a login prompt appears on the screen, use the following credentials:

ID: admin

Password: admin

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXf6ggVt8bx1zvhl0Njms1rUD_yj7CNj682U80SbO7R85k_M2I7OyEHn9UdB6lKGL0xENrvKicLp1Tf3Pg_f9Vy4CMJr3R56cDqDl2gw3mjeXqU8l-bZ5OTVPXt8insfVJvGHMm3sg?key=d7SuYcqjEJUINTfx346VzS50" alt=""><figcaption><p>Sample outup while logging in</p></figcaption></figure>

{% hint style="info" %}
While typing in the password, it's normal when the password doesn't appear on the screen. It's hidden for security reason.
{% endhint %}

## 4. Network Configuration After Login

### Wifi Configuration

{% hint style="info" %}
If you are using a cable to connect your node, please skip this step check the below [Ethernet Connection](set-up-a-dephy-node.md#ethernet-connection) step.
{% endhint %}

Use the following command to connect to Wi-Fi by replacing your real SSID and password:

```bash
sudo nmcli dev wifi connect "YourSSID" password "YourPassword"
```

### Ethernet Connection

{% hint style="info" %}
If you don't have a network cable connecting your node, please check the above [Wifi Configuration](set-up-a-dephy-node.md#wifi-configuration) guide.
{% endhint %}

For most cases, a wired connection should automatically assign an IP address when the cable is connected. However, it is not confirmed if this device is pre-configured for automatic DHCP.

## 5. Check the IP Address

After configuring the network, use the following command to check the device’s IP address:

```bash
ip address
```

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcHnQzbrrwXcqHrilA8_DdomxcZc6F845y8MRbtEOvO8daeTlJdha1i14ZMUABZ1gHRWYKFHb1nucdLZAFrIVV0dhy4PYlOFK-vgco4EQbMLtiw-6HI7Mm7RO078ML0mMen5YfhTQ?key=d7SuYcqjEJUINTfx346VzS50" alt=""><figcaption><p>IP showed under the wlan0 section</p></figcaption></figure>

## 6. Setting Up SSH Terminal for Command Convenience

To easily copy and paste commands, connect to the device via an SSH terminal using tools like _**Putty**_ or _**Termius**_.

Configure the terminal with the device’s IP address and log in using:

ID: admin

Password: admin

## 7. Execute the Following Command After Login

Replace `{replace with your Solana address}` with your real Solana address:

{% code overflow="wrap" %}
```bash
docker run --restart always --network host --name worker --volume $HOME/data:/opt/dephy-worker/data -d dephyio/dephy-testnet2-worker --owner-solana-address {replace with your Solana address} && sudo docker logs -f --tail 10 worker
```
{% endcode %}

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcEwxX_PZiVG7IBrtrl7R5TLCg78uy5g23JpZpNfuRWX73sTa59kG4IOgqt8COpiemU_ojmEK8qzMaj9N8TSVt05p4GZvOHT20wBRcFLZddYV372XisFZiD3Cu2__LL0yuU15udvA?key=d7SuYcqjEJUINTfx346VzS50" alt=""><figcaption><p>Sample output after typing the command</p></figcaption></figure>

## 8. Check Registration Status

After confirming your owner address, visit [https://status.dephy.app/](https://status.dephy.app/) to verify if your worker has been successfully registered.

{% hint style="info" %}
Note: It may take a few minutes to appear on the website.
{% endhint %}

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXchauGwX_9SxI34J_RgFetXqpg-XzG2mdvIFKB2VRF-6kGjXudZQ9fQkKTNbsyUkB0KsxY-axjxRSQwIWs0VhysbHyU8ywa6Xnogj2NYiy9ids-omUPlwx-A4bT5PyMRCRTfT-w?key=d7SuYcqjEJUINTfx346VzS50" alt=""><figcaption></figcaption></figure>

## 9. Migrate to the Testnet2 from Testnet1

{% hint style="info" %}
You need to do this ONLY if your node already participated Testnet1, or you can skip this chapter.
{% endhint %}

Please check the guide:

{% content-ref url="migration-from-testnet1-to-testnet2.md" %}
[migration-from-testnet1-to-testnet2.md](migration-from-testnet1-to-testnet2.md)
{% endcontent-ref %}

## Frequently Asked Questions

{% content-ref url="node-setup-faq.md" %}
[node-setup-faq.md](node-setup-faq.md)
{% endcontent-ref %}
