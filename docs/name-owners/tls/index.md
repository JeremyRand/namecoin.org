---
layout: page
title: Setting up TLS (for name owners)
---

{::options parse_block_html="true" /}

ncdns includes the tool `generate_nmc_cert`.  This tool can be used to create a self-signed Namecoin TLS certificate that is compatible with ncdns's Chromium integration.  (Standard certificate generation tools like `openssl` are not usable for this purpose.)

Example usage:

~~~
$ ./generate_nmc_cert -host www.example.bit
2019/12/11 06:47:41 written cert.pem
2019/12/11 06:47:41 written key.pem
2019/12/11 06:47:41 Your Namecoin cert is: {"d8":[1,"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEOcD7/hnngrlW3XrTviUjXHgbvraj3mw7Wa872Iti6Dp0Jrb9P6ZsINAZSU4mucH37zX7/pscyb6UBO08SqDP+w==",5253561,5358681,10,"MEUCIQDQF/IAHUv1UqGTGrWMvu/Tfj6PiNRBb3eTerpyZmTJjAIgeEc3a8yyNY09PrDSGkEOh0T6ZmVqbesqnZVLYszEnWI="]}
2019/12/11 06:47:41 SUCCESS: The cert rehydrated to an identical form.  Place the generated files in your HTTPS server, and place the above JSON in the "tls" field for your Namecoin name.
~~~

By default, certificates use the P256 curve and are valid from generation time for approximately 1 year.  Users who know what they are doing can choose different ECDSA curves or validity periods; use the `-help` option to see the list of available options.

The JSON object displayed should be enclosed in an array, and placed in the `tls` field for the domain where you want the TLSA record to appear.  Usually, this will be the `_443._tcp` subdomain of the domain name that points to the website.  An example of a typical configuration is:

~~~
{
    "ip": "73.239.16.12", 
    "map": {
        "_tcp": {
            "map": {
                "_443": {
                    "tls": [
                        {
                            "d8": [
                                1, 
                                "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEQ+XwIFSYT8U24gBKFDgQ/iRuFcAQGNhJweooIRYw5G9TtAJJ2CyTHWNsfbq+5c6LZ7fErMOdIXHQhHbP68dnZA==", 
                                4944108, 
                                5259468, 
                                10, 
                                "MEQCICkDDuAmj7F9RvhUaSaOpIfW2HvSPP1YUqQMSBwycYZFAiB/u7K+F2xlfTBizkFLvFPiRfj2oFqttaXBZzO/UKewPw=="
                            ]
                        }
                    ]
                }
            }
        }
    }
}
~~~

Remember that if you have an `ns` record above the `tls` record, the `ns` record will suppress the `tls` record.  For example, the following configuration will **not** work:

~~~
{
    "ns": "ns21.cloudns.net.",
    "map": {
        "_tcp": {
            "map": {
                "_443": {
                    "tls": [
                        {
                            "d8": [
                                1, 
                                "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEQ+XwIFSYT8U24gBKFDgQ/iRuFcAQGNhJweooIRYw5G9TtAJJ2CyTHWNsfbq+5c6LZ7fErMOdIXHQhHbP68dnZA==", 
                                4944108, 
                                5259468, 
                                10, 
                                "MEQCICkDDuAmj7F9RvhUaSaOpIfW2HvSPP1YUqQMSBwycYZFAiB/u7K+F2xlfTBizkFLvFPiRfj2oFqttaXBZzO/UKewPw=="
                            ]
                        }
                    ]
                }
            }
        }
    }
}
~~~

The `cert.pem` and `key.pem` files can be used with HTTPS servers like Caddy, Nginx, or Apache, just like for any other certificate.

Note that wildcard certificates are not currently supported; each domain name needs its own certificate.

You can test your setup by visiting your `.bit` domain in Chromium on Windows when ncdns is installed.
