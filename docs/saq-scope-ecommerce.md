# SAQ scope for e-commerce, and why client-side controls always apply

A short orientation before the requirement-specific notes: which SAQ you fall under, and
why Requirements 6.4.3 and 11.6.1 apply to you even if you "don't touch card data".

## Which SAQ applies

For e-commerce, the SAQ is decided by how the card data is captured in the browser:

| Capture model | Typical SAQ | What is in scope |
| --- | --- | --- |
| All cardholder data fields fully delegated to a PCI DSS compliant third party via redirect or iframe, and the merchant page does not receive card data | SAQ A | Far fewer controls, but still includes 6.4.3 and 11.6.1 for the page that hosts the redirect/iframe |
| Merchant page presents or can influence the card fields (direct post, JS that touches fields, or a single-page checkout that renders the fields) | SAQ D | The full requirement set |
| Service provider that stores, processes, transmits CHD, or can affect its security on behalf of others | SAQ D - Service Provider | The full set, with the stricter service-provider cadences |

If you build platforms used by other merchants, you are likely a service provider and the
SAQ D Service Provider obligations apply (some reviews are every six months rather than
twelve).

## Why 6.4.3 and 11.6.1 apply even to SAQ A

This is the part teams get wrong. Even when the card fields live entirely inside a third
party iframe or you redirect to a hosted page, the page that hosts that iframe or initiates
that redirect runs in the consumer's browser and can be tampered with. An attacker who
compromises a script on that page can:

- swap the iframe `src` to a look-alike origin and capture the card themselves;
- overlay a fake form on top of the legitimate iframe;
- redirect the checkout to an attacker-controlled clone.

None of that requires your server or the third party to be breached. That is exactly the
risk 6.4.3 (control and assure the scripts) and 11.6.1 (detect tampering of the page and
its headers) exist to address, which is why v4.0.1 made them mandatory for SAQ A too as of
31 March 2025.

## Practical takeaway

Treat the payment page (and any page that hosts a payment iframe or starts a redirect) as
in scope for client-side integrity and tamper detection regardless of your SAQ. The notes
in `docs/req-6.4.3-sri-csp.md` and `docs/req-11.6.1-payment-page-tamper-detection.md` apply
to all e-commerce models.
