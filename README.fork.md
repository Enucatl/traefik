# Traefik with ML-DSA support

This fork builds upstream Traefik with Go 1.27.3 or newer so it can terminate HTTPS and
verify client certificates using ML-DSA. The complete upstream dashboard is
built and embedded; no Traefik application code is changed.
Publication requires Go 1.27.3 or newer.

```sh
docker pull ghcr.io/enucatl/traefik:latest
```

`main` rebases onto `traefik/traefik`'s `v3.7` branch daily at 04:43 UTC.
Pushes to `main` and manual runs also build and test the image. The daily job
updates `main` only after the rebased image passes its checks. A rebase conflict
or failed check prevents publication, preserving the previous image.

Only `latest` is published. Image labels record the fork revision; use the
image digest when a deployment needs a fixed build. The image targets Linux
amd64, matching the current Atlas host. Following upstream `v3.7` includes
maintenance changes before they appear in a release.

The pipeline builds the complete dashboard and runs unit tests and the HTTPS integration suite.
The mutual TLS test runs against both RSA and ML-DSA-44 PEM fixtures. It checks server
certificate verification, accepts clients signed by either trusted CA, and rejects
missing client certificates and clients signed by an untrusted CA.
The ML-DSA-44 examples are in `integration/fixtures/https/mldsa/`; regenerate them
with `integration/fixtures/https/generate_cert.sh` using OpenSSL 3.5 or newer.

Go requires ML-DSA private keys encoded with the seed only. Convert OpenSSL's
default key encoding before configuring Traefik:

```sh
openssl pkey -provparam ml-dsa.output_formats=seed-only -in server.key -out server-seed.key
```

To build and check locally:

```sh
make binary-linux-amd64
make test-unit
make test-integration TESTFLAGS='-run TestHTTPSSuite'
docker build --build-arg TARGETPLATFORM=linux/amd64 -t ghcr.io/enucatl/traefik:latest .
```
