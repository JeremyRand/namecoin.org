## TLS Positive Overrides

{{ page.application }} for {{ os }} can be used with Namecoin for TLS positive overrides; this allows certificates for `.bit` domains that match the blockchain to be used without errors.  Instructions:

1. Find your {{ page.application }} profile folder; it will usually be a subfolder of `{{ nssdbdir }}`.  For example, `{{ nssdbdir }}/r3a8ono6.default`.{% if os == "Windows" %}  Note that you must use forward slashes, not backslashes.{% endif %}
1. Make sure that your {{ page.application }} profile folder is readable and writeable by the user who is running ncdns.{% if os == "Windows" %}  If you installed ncdns via the installer, ncdns is run by the user `NT SERVICE\ncdns`.{% endif %}
1. Add the following to `ncdns.conf`, substituting your {{ page.application }} profile folder for `$PROFILEDIR`:
   
       [tlsoverridefirefox]
       sync=true
       profiledir="$PROFILEDIR"
   
1. Restart ncdns.

TLS positive overrides will only be synchronized to {{ page.application }} while {{ page.application }} is not running; this means that, every so often, you should close Firefox for a few minutes and then re-open it.

You can now visit in {{ page.application }} a `.bit` website that supports TLS, e.g. [the Namecoin forum's `.bit` domain](https://nf.bit/).  The website should load in {{ page.application }} without errors.

