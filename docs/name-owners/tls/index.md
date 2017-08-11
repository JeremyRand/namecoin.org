---
layout: page
title: Setting up TLS (for name owners)
---

{::options parse_block_html="true" /}

ncdns includes the tool `generate_nmc_cert`.  This tool can be used to create a self-signed Namecoin TLS certificate that is compatible with ncdns's Chromium integration.  (Standard certificate generation tools like `openssl` are not usable for this purpose.)

Example usage:

~~~
$ ./generate_nmc_cert -ecdsa-curve P256 -host www.example.bit -start-date "Jan 1 00:00:00 2017" -end-date "Jan 1 00:00:00 2020"
2017/08/10 20:03:59 written cert.pem
2017/08/10 20:03:59 written key.pem
2017/08/10 20:03:59 Your Namecoin cert is: {"d8":[1, "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE0bhYUDX2X+wpDs+Kdxfyz2goO7OygMZNZJStfBCOeJCA/LZnOvK2tkjIyMR+6cpG0o+GM74ALtwOdzdCBjL61w==",4944096,5259456,10,"MEUCIEdBFF9QqgIi64BM4XY1G3Fd9M2MgGdcYHsJzANhcxwwAiEAx/IqQR10fPia/d13z9EwHAgbilBM3kZCsW3LxnC3qqc="]}
2017/08/10 20:03:59 SUCCESS: The cert rehydrated to an identical form.  Place the generated files in your HTTPS server, and place the above JSON in the "tls" field for your Namecoin name.
~~~

(Users who know what they are doing can choose other ECDSA curves; use the `--help` option to see the list of available curves.)

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
    "ns": "ns21.cloudns.net",
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
