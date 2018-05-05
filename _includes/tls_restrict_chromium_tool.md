## TLS Negative Overrides{% if tlsrestrictchromiumtoolmethodnum %} (Method {{ tlsrestrictchromiumtoolmethodnum }}){% endif %}

{{ page.application }} for {{ os }} can be used with Namecoin for TLS negative overrides; this prevents malicious or compromised public CA's from issuing certificates for `.bit` domains.  Instructions:

1. Exit {{ page.application }}.
1. Run the following, and note the output:
   
       ls {{ page.chromiumprofile }}/*/TransportSecurity
   
1. For each `TransportSecurity` file that you found above, run the following, substituting the full path of the `TransportSecurity` file for `$TS_PATH`:
   
       ./tlsrestrict_chromium_tool --tlsrestrict.chromium-ts-path=$TS_PATH
   
1. You can now re-launch {{ page.application }}.
