# Migration From Testnet1 to Testnet2

{% hint style="info" %}
You need to do this ONLY if your node already participated Testnet1, or you can skip this page.
{% endhint %}

## 1. First Testnet → Second Testnet Migration Guide

* First -> Second Migration Steps (ONLY APPLICABLE FOR EXISTING TESTNET1 NODES)
*   Save your points mined from the testnet1 (wallet connection must use the same Solana-based wallet address that you will use for the next stop right below):

    [https://testnet.dephy.app/](https://testnet.dephy.app/)

## 2. Open your SSH tool (e.g., Termius) and execute the following command:

{% code overflow="wrap" %}
```bash
sudo docker rm --force worker && docker run --restart always --network host --name worker --volume $HOME/data:/opt/dephy-worker/data -d dephyio/dephy-testnet2-worker --owner-solana-address {replace with your Solana address} && sudo docker logs -f --tail 10 worker
```
{% endcode %}

Note: If your current docker container is named worker1, you need to modify the command to&#x20;

```bash
sudo docker rm --force worker1
```

## 3. Execute the Following Command After Login:

{% code overflow="wrap" %}
```bash
docker run --restart always --network host --name worker --volume $HOME/data:/opt/dephy-worker/data -d dephyio/dephy-testnet2-worker --owner-solana-address {replace with your Solana address} && sudo docker logs -f --tail 10 worker
```
{% endcode %}

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcEwxX_PZiVG7IBrtrl7R5TLCg78uy5g23JpZpNfuRWX73sTa59kG4IOgqt8COpiemU_ojmEK8qzMaj9N8TSVt05p4GZvOHT20wBRcFLZddYV372XisFZiD3Cu2__LL0yuU15udvA?key=d7SuYcqjEJUINTfx346VzS50" alt=""><figcaption></figcaption></figure>

## 4. After confirming your owner address

Visit [https://status.dephy.app/](https://status.dephy.app/) to verify if your worker has been successfully registered.\
Note: It may take a few minutes to appear

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXchauGwX_9SxI34J_RgFetXqpg-XzG2mdvIFKB2VRF-6kGjXudZQ9fQkKTNbsyUkB0KsxY-axjxRSQwIWs0VhysbHyU8ywa6Xnogj2NYiy9ids-omUPlwx-A4bT5PyMRCRTfT-w?key=d7SuYcqjEJUINTfx346VzS50" alt=""><figcaption></figcaption></figure>

## Frequently Asked Questions

{% content-ref url="node-setup-faq.md" %}
[node-setup-faq.md](node-setup-faq.md)
{% endcontent-ref %}
