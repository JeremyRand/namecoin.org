## TLS Negative Overrides{% if tlsrestrictnsstoolmethodnum %} (Method {{ tlsrestrictnsstoolmethodnum }}){% endif %}

{{ page.application }} for {{ os }} can be used with Namecoin for TLS negative overrides; this prevents malicious or compromised public CA's from issuing certificates for `.bit` domains.  Instructions:

1. Create a temporary directory; make sure that it only is readable/writeable by a user whom you trust with access to the {{ page.application }} certificate database.
{% if nssdbdirsubdir %}1. Find your NSS directory; it will usually be a subdirectory of `{{ nssdbdir }}`.  For example, `{{ nssdbdir }}/r3a8ono6.default`.
1. Run the following, substituting the temporary directory and NSS directory for `$TEMP_DIR` and `$NSS_DIR`:
   
       ./tlsrestrict_nss_tool --tlsrestrict.nss-temp-db-dir="$TEMP_DIR" --tlsrestrict.nss-dest-db-dir="$NSS_DIR"
   {% else %}1. Run the following, substituting the temporary directory for `$TEMP_DIR`:
   
       ./tlsrestrict_nss_tool --tlsrestrict.nss-temp-db-dir="$TEMP_DIR" --tlsrestrict.nss-dest-db-dir="{{ nssdbdir }}"
   {% endif %}
1. Wait a few minutes for `tlsrestrict_nss_tool` to finish running.

You'll need to redo the above steps whenever the built-in certificate list is updated.

If you've manually imported any non-built-in TLS trust anchors to {{ page.application }}, and you want to restrict them from intercepting `.bit` traffic, you should do the following for each such trust anchor:

1. Get a DER-encoded certificate of the trust anchor.
1. Run the following, substituing the path to your trust anchor certificate for `$CERT_PATH`:
   
       ./cross_sign_name_constraint_tool --cert.input-root-ca-path="$CERT_PATH"
   
1. 3 new certificates will be created: `root.crt`, `intermediate.crt`, and `cross-signed.crt`.
1. Delete the existing trust anchor from {{ page.application }}.
1. Import `root.crt` into {{ page.application }}; mark it as a trusted TLS root CA.
1. Import `intermediate.crt` and `cross-signed.crt` into {{ page.application }}; do not mark them as trusted.

If you decide later that you want to remove the negative overrides from {{ page.application }}, run the following:

    ./tlsrestrict_nss_tool --tlsrestrict.nss-temp-db-dir="$TEMP_DIR" --tlsrestrict.nss-dest-db-dir="{% if nssdbdirsubdir %}$NSS_DIR{% else %}{{ nssdbdir }}{% endif %}" --tlsrestrict.undo
