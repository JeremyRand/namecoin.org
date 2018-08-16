{::options parse_block_html="true" /}

{% assign os = "GNU/Linux" %}
{% assign tlsrestrictchromiumtoolmethodnum = "A" %}
{% assign nssdbdir = "$HOME/.pki/nssdb" %}
{% assign tlsrestrictnsstoolmethodnum = "B" %}


{% include tls_certinject_nss.md %}

{% include tls_restrict_chromium_tool.md %}

{% include tls_restrict_nss_tool.md %}
