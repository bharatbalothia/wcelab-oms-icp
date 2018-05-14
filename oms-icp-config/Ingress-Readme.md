<H1>Ingress Notes</H1>

Key points to note are as below:

1. Have a redirect rule annotation to handle http redirection

"ingress.kubernetes.io/proxy-redirect-from": "~^(http?://[^:]+):\\d+(?<relpath>/.+)$ $scheme://$http_host$relpath"

This rewrite rule is required because when WAS sends redirect (HTTP 302) responses, it always uses WAS port as per the behavior described in http://www-01.ibm.com/support/docview.wss?uid=swg21470923. But since we need the browser to be redirected without the port number as ingress controller listens only to http and https, this rewrite rule ensures that both http and https redirects are re-written without any port numbers so that they goto the default ports.

2. If HTTPS is enabled on ingress controller, HTTP requests are redirected by default to https. To disable this, add the following annotations

"ingress.kubernetes.io/ssl-redirect": "false"

3. If HTTPS support has to be enabled on ingress controller, follow steps to add TLS support - reference -  https://www.ibm.com/support/knowledgecenter/en/SS5PWC/front_end_tls_ingress_task.html

4. To support sticky sessions, add the following annotations

    "ingress.kubernetes.io/affinity": "cookie"
    "ingress.kubernetes.io/session-cookie-name": "route"
    "ingress.kubernetes.io/session-cookie-hash": "sha1"

Note: It's recommended to make sure that ingress annotations have taken effect by describe ingress command as wrong or unsupported annotations won't throw any errors and the only way to confirm they have taken effect is to review the describe output.
