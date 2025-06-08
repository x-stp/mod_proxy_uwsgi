# Apache mod_proxy_uwsgi

## Overview

This repository provides a testing lab for httpd < 2.4.44 ; currently used
to test against for the CVE-2020-11984 vulnerability. 

## CVE-2020-11984

Apache mod_proxy_uwsgi uses `uint16_t` (16-bit unsigned integer) for packet size calculations, but fails to verify that the total size doesn't overflow this limit. When HTTP headers exceed 65,535 bytes, the packet size wraps around, causing Apache to send malformed packets to the UWSGI backend.

### Technical Details

From the vulnerable code in `modules/proxy/mod_proxy_uwsgi.c`:

```c
apr_size_t headerlen = 4;
apr_uint16_t pktsize, keylen, vallen;

// Calculate total header size
for (j = 0; j < env_table->nelts; ++j) {
    headerlen += 2 + strlen(env[j].key) + 2 + strlen(env[j].val);
}

// This assignment causes integer overflow when headerlen > 65539
pktsize = headerlen - 4;  // uint16_t max is 65535!
```

### Potential impact

1. **Information Disclosure**: The malformed packet can cause UWSGI to misinterpret the packet structure, potentially exposing environment variables in the response body.

2. **Denial of Service**: Malformed packets can crash or hang the UWSGI backend.

3. **Remote Code Execution**: In specific configurations (persistent UWSGI mode or with additional vulnerabilities), attackers might inject malicious UWSGI variables like `UWSGI_FILE`.

## References

- [CVE-2020-11984 Details](https://nvd.nist.gov/vuln/detail/CVE-2020-11984)
- [Apache Security Advisory](https://httpd.apache.org/security/vulnerabilities_24.html)
- [Apache SVN Commit r1880205](https://svn.apache.org/viewvc?view=revision&revision=1880205)

## Credits

CVE-2020-11984 was discovered by Felix Wilhelm of Google Project Zero 

## httpd

Continuation of NCSA HTTPd by Rob McCool and others. 

The Apache Software Foundation is the current maintainer.

## UWSGI

UWSGI was written by Roberto De Ioris and is now maintained by the UWSGI project.

## References

- [Apache httpd](https://httpd.apache.org/)
- [UWSGI](https://uwsgi-docs.readthedocs.io/en/latest/)
