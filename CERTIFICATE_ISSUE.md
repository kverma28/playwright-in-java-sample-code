# Playwright Certificate Issue on macOS

## Problem

When running a Playwright Java test, browser installation may fail with:

```text
Error: unable to get local issuer certificate
code: UNABLE_TO_GET_ISSUER_CERT_LOCALLY
```

This can happen on a corporate network when a proxy (for example, Zscaler) inspects HTTPS traffic and Node.js does not trust the corporate Certificate Authority (CA).

## Resolution

### 1. Obtain the corporate CA certificate

Ask your IT team for the approved corporate **root CA certificate** (and any required intermediate CA certificates) in PEM format. Do not use a website-specific certificate such as `cdn.playwright.dev` as the trusted root.

Example file location:

```text
/Users/<username>/DO_NOT_DELETE/Tesco Root CA.pem
```

### 2. Configure Node.js

Add the following to `~/.zshrc`, updating the path to match your certificate:

```bash
export NODE_EXTRA_CA_CERTS="/Users/<username>/DO_NOT_DELETE/Tesco Root CA.pem"
```

Reload the shell and verify the variable:

```bash
source ~/.zshrc
echo "$NODE_EXTRA_CA_CERTS"
```

Check that the certificate file exists and is readable:

```bash
ls -l "$NODE_EXTRA_CA_CERTS"
openssl x509 -in "$NODE_EXTRA_CA_CERTS" -noout -subject -issuer
```

The file should contain PEM certificate data beginning with:

```text
-----BEGIN CERTIFICATE-----
```

### 3. Configure IntelliJ IDEA

If the test is run from IntelliJ, the JUnit runner may not inherit variables from `~/.zshrc`.

1. Open **Run → Edit Configurations**.
2. Select the JUnit run configuration used by the test.
3. Add this environment variable:

   ```text
   NODE_EXTRA_CA_CERTS=/Users/<username>/DO_NOT_DELETE/Tesco Root CA.pem
   ```

4. Apply the change and rerun the test.

Node.js reads `NODE_EXTRA_CA_CERTS` when the process starts, so restart the test process after changing it.

## Important notes

- This variable is needed for Node.js operations that require the corporate CA, such as downloading Playwright browsers. It does not configure the Java browser itself to trust certificates.
- Once the required browser version is installed, Playwright usually does not download it again unless it is missing or a different version is required.
- Do **not** set `NODE_TLS_REJECT_UNAUTHORIZED=0`. That disables TLS certificate verification and is insecure.
- If the error continues, confirm that the certificate is the approved corporate CA chain and that the environment variable reaches the process performing the download.
