---
layout: page
title: Setting up TLS (for name owners)
---

{::options parse_block_html="true" /}

The Namecoin tool `generate_nmc_cert` can be used to create a TLS certificate that will be recognized as valid by TLS implementations where Namecoin is installed.  (Standard certificate generation tools like `openssl` are not usable for this purpose.)

There are currently 3 forms of Namecoin TLS certificates: Dehydrated, Constrained, and Encaya.  The Dehydrated form is older and is supported by more TLS implementations than the Constrained form.  The Constrained form is more secure and uses less blockchain storage than the Dehydrated form.  The Encaya form retains the Constrained form's security improvements, uses even less blockchain storage than the Constrained form, and has even better compatibility than the Dehydrated form.  It is likely that the Dehydrated and Constrained forms will be phased out once the Encaya form has been more widely adopted by the Namecoin ecosystem.  By default, `generate_nmc_cert` will produce a Dehydrated certificate.

Example usage (Dehydrated form):

~~~
$ ./generate_nmc_cert -host www.example.bit
2019/12/11 06:47:41 written cert.pem
2019/12/11 06:47:41 written key.pem
2019/12/11 06:47:41 Your Namecoin cert is: {"d8":[1,"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEOcD7/hnngrlW3XrTviUjXHgbvraj3mw7Wa872Iti6Dp0Jrb9P6ZsINAZSU4mucH37zX7/pscyb6UBO08SqDP+w==",5253561,5358681,10,"MEUCIQDQF/IAHUv1UqGTGrWMvu/Tfj6PiNRBb3eTerpyZmTJjAIgeEc3a8yyNY09PrDSGkEOh0T6ZmVqbesqnZVLYszEnWI="]}
2019/12/11 06:47:41 SUCCESS: The cert rehydrated to an identical form.  Place the generated files in your HTTPS server, and place the above JSON in the "tls" field for your Namecoin name.
~~~

To produce a Constrained certificate instead, use the `-use-ca` command-line flag, like this:

~~~
$ ./generate_nmc_cert -host www.example.bit -use-ca
2019/12/11 06:54:39 Your CA's "tls" record is: [2, 1, 0, "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEm1nvuS+A5WFgafCeYmzVSZOsokU1Fmnh5ZiBC7h0pRkbkx7cCA/MYPPh6zDdMB75ELvXSt0eLaoQQYaz1QDijw=="]
2019/12/11 06:54:39 written caKey.pem
2019/12/11 06:54:39 written cert.pem
2019/12/11 06:54:39 written key.pem
2019/12/11 06:54:39 SUCCESS.  Place cert.pem and key.pem in your HTTPS server, and place the above JSON in the "tls" field for your Namecoin name.
~~~

To produce an Encaya certificate instead, use both the `-use-ca` and `-use-aia` command-line flags, like this:

~~~
$ ./generate_nmc_cert -host example.bit -use-ca -use-aia
2021/03/14 05:00:43 wrote namecoin.json
2021/03/14 05:00:43 wrote caAIAKey.pem
2021/03/14 05:00:43 wrote caCert.pem
2021/03/14 05:00:43 wrote caKey.pem
2021/03/14 05:00:43 wrote cert.pem
2021/03/14 05:00:43 wrote key.pem
2021/03/14 05:00:43 SUCCESS. Place cert.pem and key.pem in your HTTPS server, and place the contents of "namecoin.json" in th
e "tls" field for "*.example.bit".
~~~

If you're using the Constrained or Encaya form, you can produce multiple certificates for a single Namecoin record.  This allows you to, for example, rotate TLS keys periodically without needing to update your Namecoin record.  To create a new certificate using an existing Namecoin record, set the `-parent-key` command-line argument to the `caKey.pem` file that you created with the first certificate, like this:

~~~
$ ./generate_nmc_cert -host www.example.bit -use-ca -parent-key ./caKey-www.example.bit.pem
2019/12/11 07:11:06 Using existing CA private key
2019/12/11 07:11:06 Your CA's "tls" record is: [2, 1, 0, "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEm1nvuS+A5WFgafCeYmzVSZOsokU1Fmnh5ZiBC7h0pRkbkx7cCA/MYPPh6zDdMB75ELvXSt0eLaoQQYaz1QDijw=="]
2019/12/11 07:11:06 written cert.pem
2019/12/11 07:11:06 written key.pem
2019/12/11 07:11:06 SUCCESS.  Place cert.pem and key.pem in your HTTPS server, and place the above JSON in the "tls" field for your Namecoin name.
~~~

By default, certificates use the P256 curve and are valid from generation time for approximately 1 year.  Users who know what they are doing can choose different elliptic curves or validity periods; use the `-help` command-line flag to see the list of available options.

The JSON object displayed should be enclosed in an array, and placed in the `tls` field for the domain where you want the TLSA record to appear.  For the Dehydrated and Constrained forms, this will usually be the `_443._tcp` subdomain of the domain name that points to the website.  For the Encaya form, this will usually be the `*` subdomain of the eTLD+1 domain name.  An example of a typical configuration (using the Dehydrated form) is:

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

An example of the Encaya form is:

~~~
{
    "ip": "73.239.16.12",
    "map": {
        "*": {
            "tls": [
                [
                    2,
                    1,
                    0,
                    "MDkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDIgADcO7sJ5kaRYGipnz5YMXwo6NDGLWV4qVoJIn+1by1oAo="
                ]
            ]
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

The `chain.pem` and `key.pem` files can be used with HTTPS servers like Caddy, Nginx, or Apache, just like for any other certificate.

If you are using the Dehydrated form, or the default settings for the Constrained or Encaya form, you will need to make sure that your HTTPS server supports either `ECDHE-ECDSA` or `DHE-ECDSA` ciphersuites.  If you require non-forward-secret ciphersuites or non-ECDSA keys, you will need to use the Constrained or Encaya form to generate a CA, and then manually use that CA to sign an end-entity certificate (via whatever non-Namecoin tooling you like) that uses the desired `KeyUsage` value or key type.

Wildcard certificates are only supported by the Encaya form.

You can test your setup by visiting your `.bit` domain in a [Namecoin-supported TLS client]({{ "/docs/tls-client/" | relative_url }}).
