---
title: Configure webhook URLs with reverse proxy
description: Customize n8n webhook URLs for compatibility with reverse proxy setups.
contentType: howto
---

# Configure n8n webhooks with reverse proxy

n8n creates the webhook URL by combining `N8N_PROTOCOL`, `N8N_HOST` and `N8N_PORT`. If n8n runs behind a reverse proxy, that won't work. That's because n8n runs internally on port 5678 but the reverse proxy exposes it to the web on port 443.

## Why proper webhook URL configuration matters

Incorrect webhook URL configuration can cause OAuth authentication failures, particularly with services like Twitter/X, as n8n may generate callback URLs using internal addresses (e.g., `http://localhost:5678`) instead of the publicly accessible URL. This results in OAuth providers being unable to redirect users back to your n8n instance after authentication.

## Configuration requirements

When running n8n behind a reverse proxy, it's important to do the following:

* Set the webhook URL manually with the `WEBHOOK_URL` environment variable so that n8n can display it in the editor UI and register the correct webhook URLs with external services.
* Set the `N8N_PROXY_HOPS` environment variable to `1`.
* On the last proxy on the request path, set the following headers to pass on information about the initial request:
    * [`X-Forwarded-For`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-For)
    * [`X-Forwarded-Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-Host)
    * [`X-Forwarded-Proto`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-Proto)

## Configuration example

```bash
export WEBHOOK_URL=https://n8n.example.com/
export N8N_PROXY_HOPS=1
```

### Docker Compose example

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n
    environment:
      - WEBHOOK_URL=https://n8n.example.com/
      - N8N_PROXY_HOPS=1
      - N8N_PROTOCOL=https
      - N8N_HOST=n8n.example.com
    ports:
      - "5678:5678"
```

## Expected behavior after configuration

Once properly configured:

* OAuth callbacks (including Twitter/X authentication) will use the correct external URL (`https://n8n.example.com/`) instead of internal addresses
* Webhook nodes will display and register the publicly accessible URL
* External services will be able to successfully send webhook requests to your n8n instance
* Twitter/X OAuth authentication will complete successfully, as the OAuth provider can properly redirect back to your n8n instance

## Troubleshooting

If OAuth authentication still fails after configuration:

1. Verify that `WEBHOOK_URL` matches your publicly accessible domain (including the protocol `https://`)
2. Ensure your reverse proxy is properly forwarding the X-Forwarded headers
3. Check that your OAuth application settings in the external service (e.g., Twitter Developer Portal) have the correct callback URL registered
4. Restart n8n after changing environment variables

Refer to [Environment variables reference](/hosting/configuration/environment-variables/endpoints.md) for more information on these variables.
