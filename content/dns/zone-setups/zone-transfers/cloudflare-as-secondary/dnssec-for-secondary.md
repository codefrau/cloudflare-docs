---
pcx_content_type: concept
title: DNSSEC options
weight: 3
meta:
    title: DNSSEC for Secondary DNS
---

# DNSSEC for incoming zone transfers

[DNS Security Extensions (DNSSEC)](https://www.cloudflare.com/learning/dns/dns-security/) increase security by adding cryptographic signatures to DNS records. When you use multiple providers and Cloudflare is secondary, you have a few options to enable DNSSEC for records served by Cloudflare.

- **[Multi-signer DNSSEC](/dns/dnssec/multi-signer-dnssec/setup/)**: Both Cloudflare and your primary DNS provider know the signing keys of each other and perform their own live-signing of DNS records, in accordance with [RFC 8901](https://www.rfc-editor.org/rfc/rfc8901.html).
- **[Live signing](#set-up-live-signing-dnssec)**: If your domain is not delegated to your primary provider's nameservers and Cloudflare secondary nameservers are the only nameservers authoritatively responding to DNS queries (hidden primary setup), you can choose this option to allow Cloudflare to perform live-signing of your DNS records.
- **[Pre-signed](#set-up-pre-signed-dnssec)**: Your primary DNS provider signs records and transfers out the signatures. Cloudflare then serves these records and signatures as is, without doing any signing. Cloudflare only supports [NSEC records](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/)(and not NSEC3 records) and this setup does not support [Secondary DNS Overrides](/dns/zone-setups/zone-transfers/cloudflare-as-secondary/proxy-traffic/) nor [Load Balancing](/load-balancing/).

---

## Set up multi-signer DNSSEC

Refer to [Set up multi-signer DNSSEC](/dns/dnssec/multi-signer-dnssec/setup/) and follow the instructions, considering the note about Cloudflare as Secondary.

---

## Set up live signing DNSSEC

If you use Cloudflare secondary nameservers as the only nameservers authoritatively responding to DNS queries (hidden primary setup), you can enable live signing DNSSEC to have Cloudflare sign the records for your zone.

In this setup, DNSSEC on your pirmary DNS provider does not need to be enabled.

{{<tabs labels="Dashboard | API">}}
{{<tab label="dashboard" no-code="true">}}

1.  Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/login) and select your account and zone.
2.  Go to **DNS** > **Settings**.
3. Under **DNSSEC with Secondary DNS** select **Live signing**. You will then have access to several necessary values to create a **DS** record at your registrar.

4. {{<render file="_dnssec-registrar-steps.md">}}

{{</tab>}}
{{<tab label="api" no-code="true">}}

1. Use the [Edit DNSSEC Status endpoint](/api/operations/dnssec-edit-dnssec-status) and set a `status` of `active` for your zone.

```bash
curl --request PATCH https://api.cloudflare.com/client/v4/zones/{zone_id}/dnssec \
--header 'X-Auth-Email: <EMAIL>' \
--header 'X-Auth-Key: <KEY>' \
--header 'Content-Type: application/json' \
--data '{
   "status": "active"
  }'
```

2. Use the [DNSSEC Details endpoint](/api/operations/dnssec-dnssec-details) to get the necessary values to create a **DS** record at your registrar.

3. {{<render file="_dnssec-registrar-steps.md">}}

{{</tab>}}
{{</tabs>}}


---

## Set up pre-signed DNSSEC

{{<Aside type="warning" header="Important: NSEC3 not supported">}}

If your primary DNS provider uses NSEC3 instead of NSEC, Cloudflare will fail to serve the pre-signed zone. Authenticated denial of existence is an essential part of DNSSEC ([RFC 7129](https://www.rfc-editor.org/rfc/rfc7129.html)) and is only supported by Cloudflare through NSEC.
{{</Aside>}}

### Prerequisites

* Your secondary zone in Cloudflare already exists and zone transfers from your primary DNS provider are working correctly.
* Your primary DNS provider supports DNSSEC using NSEC records (and not NSEC3).
* Your primary DNS provider transfers out DNSSEC related records, such as RRSIG, DNSKEY, and NSEC.

### Steps

1. Enable DNSSEC at your primary DNS provider.
2. Enable DNSSEC for your zone at Cloudflare, using either the Dashboard or the API.

{{<Aside type="warning">}}

Pre-signed DNSSEC does not support [Secondary DNS Overrides](/dns/zone-setups/zone-transfers/cloudflare-as-secondary/proxy-traffic/) nor [Load Balancing](/load-balancing/).

{{</Aside>}}

{{<tabs labels="Dashboard | API">}}
{{<tab label="dashboard" no-code="true">}}

a. Select your zone and go to **DNS** > **Settings**.

b. Under **DNSSEC with Secondary DNS** select **Pre-signed**.

{{</tab>}}
{{<tab label="api" no-code="true">}}

Use the [Edit DNSSEC Status endpoint](/api/operations/dnssec-edit-dnssec-status) and set the `dnssec_presigned` value to `true`.

```bash
curl --request PATCH https://api.cloudflare.com/client/v4/zones/{zone_id}/dnssec \
--header 'X-Auth-Email: <EMAIL>' \
--header 'X-Auth-Key: <KEY>' \
--header 'Content-Type: application/json' \
--data '{
   "dnssec_presigned": true
  }'
```

{{</tab>}}
{{</tabs>}}


3. Make sure Cloudflare nameservers are added at your registrar. You can see your Cloudflare nameservers on the dashboard by going to **DNS** > **Records**.

4. Make sure there is a DS record added at your registrar. The DS record is obtained from your primary DNS provider (the signer of the zone) and is what indicates to DNS resolvers that your zone has DNSSEC enabled.