{% if os == "Windows" %}
{% assign rbmtarget = "ncdns-windows-x86_64" %}
{% assign certdehydratedanerestapifile = "certdehydrate-dane-rest-api.exe" %}
{% assign libncp11file = "ncp11.dll" %}
{% assign libnssckbifile = "nssckbi.dll" %}
{% assign libnssckbitargetfile = "nssckbi-namecoin-target.dll" %}
{% endif %}

{% if os == "GNU/Linux" %}
{% assign rbmtarget = "ncdns-linux-x86_64" %}
{% assign certdehydratedanerestapifile = "certdehydrate-dane-rest-api" %}
{% assign libncp11file = "libncp11.so" %}
{% assign libnssckbifile = "libnssckbi.so" %}
{% assign libnssckbitargetfile = "libnssckbi-namecoin-target.so" %}
{% endif %}

{% if os == "macOS" %}
{% assign rbmtarget = "ncdns-osx-x86_64" %}
{% assign certdehydratedanerestapifile = "certdehydrate-dane-rest-api" %}
{% assign libncp11file = "libncp11.dylib" %}
{% assign libnssckbifile = "libnssckbi.dylib" %}
{% assign libnssckbitargetfile = "libnssckbi-namecoin-target.dylib" %}
{% endif %}

{% if application == "Tor Browser" %}
{% assign tornetwork = "1" %}
{% assign mozillapkix = "1" %}
{% endif %}

## TLS Positive and Negative Overrides

{{ page.application }} for {{ os }} can be used with Namecoin for TLS positive and negative overrides; this allows certificates for `.bit` domains that match the blockchain to be used without errors, and prevents malicious or compromised public CA's from issuing certificates for `.bit` domains.  Instructions:

1. Install [ncdns]({{site.baseurl}}docs/ncdns/).
1. {% if os == "GNU/Linux" %}Build{% else %}On a GNU/Linux system, build{% endif %} certdehydrate-dane-rest-api and ncp11 from source, like this:
   
       git clone https://github.com/namecoin/ncdns-repro.git
       cd ncdns-repro
       make submodule-update
       ./rbm/rbm build certdehydrate-dane-rest-api --target release --target {{ rbmtarget }}
       ./rbm/rbm build ncp11 --target release --target {{ rbmtarget }}
   
1. The certdehydrate-dane-rest-api binary will be a `.tar.gz` file in `./out/certdehydrate-dane-rest-api/`.
1. The ncp11 binary will be a `.tar.gz` file in `./out/ncp11/`.
1. Extract `{{ certdehydratedanerestapifile }}` from the certdehydrate-dane-rest-api `.tar.gz` file{% if os != "GNU/Linux" %} and copy it to your {{ os }} system where {{ page.application }} will be used{% endif %}.
1. Extract `{{ libncp11file }}` from the ncp11 `.tar.gz` file{% if os != "GNU/Linux" %} and copy it to your {{ os }} system where {{ page.application }} will be used{% endif %}.
1. Create a text file called `certdehydrate-dane-rest-api.conf` in the same directory where `{{ certdehydratedanerestapifile }}` is, and fill it with the following contents (if ncdns is listening on a different IP or port, change the following accordingly):
   
       [certdehydrate-dane-rest-api]
       nameserver="127.0.0.1"
       port="5391"
   
1. Run `{{ certdehydratedanerestapifile }}`.
1. If you want to test certdehydrate-dane-rest-api, try visiting `http://127.0.0.1:8080/lookup?domain=ca-test.bit` in a web browser.  You should see a certificate.  If you instead get an error or an empty page, something is wrong.
1. Make sure {{ page.application }} is installed.
{% if tornetwork %}1. Make sure {{ page.application }} is already [configured to use Namecoin for Tor name resolution]({{site.baseurl}}docs/tor-resolution/).
{% endif %}1. Make sure {{ page.application }} is shut down.
1. In {{ page.application }}'s `Browser` folder, rename `{{ libnssckbifile }}` to `{{ libnssckbitargetfile }}`.
1. Copy `{{ libncp11file }}` to {{ page.application }}'s `Browser` folder.
1. In {{ page.application }}'s `Browser` folder, rename `{{ libncp11file }}` to `{{ libnssckbifile }}`.

You can now visit in {{ page.application }} a `.bit` website that supports TLS, e.g. [the ncp11 test page](https://ca-test.bit/).  The website should load in {{ page.application }} without errors.  {% if mozillapkix %}Note that only CA trust anchors are accepted; end-entity trust anchors are not accepted.  This means that some older `.bit` domains will have their certificates rejected in {{ page.application }}.  We are working on contacting the affected `.bit` domain owners to ask them to upgrade their setup.{% endif %}

