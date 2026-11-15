day-1:
what is webHook ?
- webHook is a event-driven communication between two application via HTTP POST method . Where a client and Server communicate using a URL.

HOW it differs from API and HOW IT WORKS ?
- API works with polling which means a client will poke or try to communicate with the server a interval until it gets the desired output/data from the Server.

-Where as Webhook is different first it takes registered URL from the CLient eg www.clienPage.com/webhook/payments 
then when the server reaches a certain states of producing the data ,means when the desired data is produced then only the server sends the data to the URL then to Client. That's why its called reverse API.

-WebHooks are used in GITHUB (gitOps) and Stripe when we changes a code and push it tells us that something has been changed need to push . Or a user done a payment or a subscription then the Server sends a notification through WebHooks HTTP Post method.
- so webhooks are lighter , faster , it reduces the overload of the server (Polling) and it reliable.

-how a Delivery looks like 
POST /webhooks/billing HTTP/1.1
Host: api.example.com
Content-Type: application/json
webhook-id: msg_2KWPBgLlAfxdpx2AI54pPJ85f4W
webhook-timestamp: 1674087231
webhook-signature: v1,K5oZfzN95Z9UVu1EsfQmfVNQhnkZ2pj9o9NDN/H/pI4=

{"type":"invoice.paid","timestamp":"2022-11-03T20:26:10.344522Z","data":{"id":"inv_123"}}

-Here Only 3 parts matter that is registerURL , The Payload , The headers and the endpoint answers with 2xx status code  Reply in case of succes Anything else — a 500, a timeout, even a redirect — counts as a failed delivery, and the provider will usually try again.

How a WebHook delivery Payload looks like ??
- Providers makes a choice about what to send and what to no 
- "Thin" payload carries only identifiers size are small {"id":"inv_123"} and "Full" payload carries the whole snapshot or the whole payload 
- Status Code:
2xx - means delivered. Everything else is a failure .       
3xx — failure. Don’t follow redirects; the consumer should update the URL.
410 Gone — the endpoint no longer wants these webhooks; disable it.
429, 502, 504 — the receiver is overloaded; slow down, and honour Retry-After if present.

At-least-Once - A retry happens whenever the provider doesn’t see your 2xx — including when your handler did the work but the response was lost on the way back. So the same event will reach you twice sometimes.

Webhook Security: Verifying Signatures?
- Read the raw body. Verify the exact bytes the provider sent, before any JSON parsing.
Rebuild the signed content the way the provider defines it — body only (GitHub, Shopify), timestamp.body (Stripe), v0:timestamp:body (Slack), id.timestamp.body (Standard Webhooks).
-Compute HMAC-SHA256 with the endpoint’s secret.

-Compare in constant time — hmac.compare_digest in Python, crypto.timingSafeEqual in Node.js. GitHub’s docs say it directly: never use a plain ==.

-Check the timestamp against your clock. Five minutes is the common tolerance; Stripe warns that a tolerance of 0 disables the check entirely rather than making it strict.

The Raw-Body Bug ?
- The single most common reason verification fails is that the handler verifies a body that has already been parsed and re-serialized. Express’s express.json(), Flask’s request.json and most API frameworks turn the body into an object before your code runs. Serializing it back changes whitespace or key order, and a cryptographic signature changes with a single byte. Both Stripe and Shopify call this out in their troubleshooting guides; Stripe’s fix for Express is to register the webhook route before the JSON middleware.