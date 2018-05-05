## TLS Positive Overrides

{{ page.application }} for {{ os }} can be used with Namecoin for TLS positive overrides; this allows certificates for `.bit` domains that match the blockchain to be used without errors.  Instructions:

1. Run the following, substituting your Namecoin Core RPC credentials for `$USER` and `$PASS`.):
   
       ./ncdumpzone --format=firefox-override --rpcuser=$USER --rpcpass=$PASS > cert_override_nmc.txt
   
2. Wait for `ncdumpzone` to finish running; it should take about a minute.
3. Exit {{ page.application }} if it's running.
4. Append the contents of `cert_override_nmc.txt` to the file named `cert_override.txt` in your {{ page.application }} profile folder. (If the file doesn't already exist, you can create an empty text file there.)
5. Start {{ page.application }} again.

You can now visit in {{ page.application }} a `.bit` website that supports TLS, e.g. [the Namecoin forum's `.bit` domain](https://nf.bit/).  The website should load in {{ page.application }} without errors.  (Obviously you need to have `.bit` DNS resolution enabled on your system, but you do **not** need any TLS-related features of ncdns enabled.)

If a `.bit` website changes its TLS certificate and you want that site to work again, redo the above instructions (but remove the `.bit` lines from `cert_override.txt` before you append the new data to it).

