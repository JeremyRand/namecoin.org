---
layout: post
title: "Reducing Size With Concise Binary Object Representation (CBOR) and Name Gifting"
author: Robert Nganga
tags: [News]
---

Several new Electrum-NMC enhancements have been added.

## Minimizing Size with Concise Binary Object Representation (CBOR) 

Incorporating Concise Binary Object Representation (CBOR) offers significant advantages, including reduced data size and improved efficiency. Currently, to use CBOR, you need to convert your JSON data into CBOR format and paste the hex equivalent into the hex tab.

![Binary Encoding (value)]({{ "/images/screenshots/electrum-nmc/Configure-Name-Dialog-Hex-2024-05-01.png" | relative_url }})

While this process can be a bit confusing, we are working to simplify it by providing an avenue to do the conversion directly within Electrum-NMC. Once you've pasted the CBOR hex data, you can confirm its validity by parsing it and displaying its contents in the DNS builder dialog, by clicking the DNS Editor button, you'll see the different records contained in the CBOR data.  Unfortunately, if you parse your CBOR hex data, you'll need to paste it again in the hex tab, but we're actively working to streamline this process.

With the addition of CBOR support in ncdns, after registering a name with a CBOR encoded value, ncdns should be able to parse and provide the DNS records associated with that name.

## Gifting Names

Similar to requesting a payment, you can now also request a name. When registering a name that is available, you'll see an extra button and field to request someone to pay for it on your behalf. Additionally, there is an extra field where you can add an additional amount to your request. 

![Request Name (value)]({{ "/images/screenshots/electrum-nmc/Buy-Name-Request-Name.png" | relative_url }})

After clicking the "Request Registration" button, a UI with your request will be displayed, similar to what is seen in the request tab. From there, you can send the request URL or QR code to the recipient `(gifter)`.

![Request Name Tabs (value)]({{ "/images/screenshots/electrum-nmc/Request-Name-UI-Tabs.png" | relative_url }})

The gifter can now paste the payment URL in the receive tab or use the QR code. The details will be automatically populated, allowing the gifter to proceed with the payment.

![Send Tab (value)]({{ "/images/screenshots/electrum-nmc/Send-UI-Request-Name.png" | relative_url }})

After your invoice is settled, you simply need to proceed with the normal name registration process. You'll only be required to pay minimal update fees, as the name has already been paid for.

<video controls>
<source src="{{ site.files_url }}/files/videos/docs/gifting/gifting.webm" type="video/webm">
Demo video of Gifting Names in Electrum-NMC.
</video>

This work was funded by NLnet Foundation’s NGI Assure Fund.
