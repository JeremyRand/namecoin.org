## TLS Positive Overrides

{{ page.application }} for {{ os }} can be used with Namecoin for TLS positive overrides; this allows certificates for `.bit` domains that match the blockchain to be used without errors.  Instructions:

1. Create a certificate storage directory for ncdns.  Make sure that it is only readable and writeable by the user running ncdns.
1. Run `echo "{{ nssdbdir }}"` and note the output; this is your NSS database directory.
1. Add the following to your `ncdns.conf`.  Substitute the certificate storage directory you created above for `$CERTDIR`, and substitute the NSS database directory that you found above for `$NSSDBDIR`.
   
       [certstore]
    
       nss=true
       nsscertdir="$CERTDIR"
       nssdbdir="$NSSDBDIR"
   
1. Restart ncdns.

You can now visit in {{ page.application }} a `.bit` website that supports TLS, e.g. [the Namecoin forum's `.bit` domain](https://nf.bit/).  The website should load in {{ page.application }} without errors.
