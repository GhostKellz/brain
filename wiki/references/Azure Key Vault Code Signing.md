---
type: reference
title: "Azure Key Vault Code Signing"
created: 2026-06-22
updated: 2026-06-22
tags:
  - codesigning
  - azure
  - keyvault
  - screenconnect
  - authenticode
  - windows
status: developing
related:
  - "[[Self-Hosted Services]]"
  - "[[Windows Administration]]"
  - "[[acme.sh - DNS-01 Certificates]]"
---

> [!key-insight] Generalized from field notes; tenant/app-registration secrets are placeholders and must never be committed.

Signing Windows binaries with an **OV/EV code-signing certificate stored in
Azure Key Vault**, using [AzureSignTool](https://github.com/vcsjones/AzureSignTool),
and wiring the signed result into a self-hosted **ScreenConnect / ConnectWise
Control** instance so its generated access agents are trusted.

> [!key-insight]
> Since the CA/Browser Forum tightened the rules (mid-2023), code-signing
> private keys must live on hardware (HSM/FIPS-140 token). **Key Vault Premium
> (HSM-backed)** satisfies this — the key is **non-exportable**, signing happens
> *inside* the vault, and the private key never touches the build host. That
> non-exportability is the whole reason the workflow below exists instead of a
> plain `.pfx`.

## 1. Store the certificate in Key Vault

Two ways to get the cert object into the vault:

- **CSR generated in Key Vault (preferred for EV):** create a certificate with
  the issuer set to your CA, Key Vault produces a CSR, the CA signs it, and you
  merge the signed cert back. The private key is generated in — and never leaves
  — the HSM.
- **Import an existing PFX:** only possible if the key is exportable (OV certs
  issued the old way). Most current EV certs cannot be imported this way.

```bash
# Premium vault = HSM-backed keys (required for EV)
az keyvault create -n <vault-name> -g <resource-group> --sku premium

# Create the cert (CSR flow): policy references the issuer + key type
az keyvault certificate create \
  --vault-name <vault-name> \
  -n <cert-name> \
  -p @<policy.json>
# → download CSR, have the CA sign it, then:
az keyvault certificate pending merge \
  --vault-name <vault-name> -n <cert-name> --file <signed-cert.cer>
```

## 2. Grant a signing identity (no secrets in notes)

The build host authenticates as an **app registration / service principal**
that has *signing-only* access to the key. Grant the **Key Vault Crypto User**
role (or, on legacy access-policy vaults, `Sign` + `Get`) — never `Export`.

```bash
az role assignment create \
  --assignee <app-registration-client-id> \
  --role "Key Vault Crypto User" \
  --scope <key-vault-resource-id>
```

> [!warning]
> The service principal's **client secret / tenant ID / client ID are
> secrets**. Keep them in a vault or CI secret store, inject as environment
> variables at sign time, and **never** put them in a wiki, repo, or script.
> Placeholders only here.

## 3. Sign with AzureSignTool

AzureSignTool is a cross-platform signtool replacement that signs via Key Vault
without ever pulling the key down.

```bash
dotnet tool install --global AzureSignTool

AzureSignTool sign \
  --azure-key-vault-url "https://<vault-name>.vault.azure.net" \
  --azure-key-vault-certificate "<cert-name>" \
  --azure-key-vault-tenant-id "<tenant-id>" \
  --azure-key-vault-client-id "<app-registration-client-id>" \
  --azure-key-vault-client-secret "$AZURE_KV_CLIENT_SECRET" \
  --timestamp-rfc3161 "http://timestamp.digicert.com" \
  --file-digest sha256 \
  -v "<path-to-binary>.exe"
```

Verify the signature:

```powershell
signtool verify /pa /v "<path-to-binary>.exe"
```

> [!key-insight]
> **Always timestamp** (`--timestamp-rfc3161`). A timestamped signature stays
> valid after the signing cert expires; an un-timestamped one goes invalid the
> day the cert lapses. Use the CA's RFC-3161 timestamp URL.

## 4. ScreenConnect / ConnectWise Control integration

Self-hosted ScreenConnect signs the access-agent installers it generates so end
users don't get SmartScreen / "unknown publisher" warnings. The signing config
lives in the instance's `web.config` (`<codeSigning>` /
`CodeSigningCertificate*` app settings).

- **Exportable cert path:** if the cert is a real `.pfx` with a password,
  ScreenConnect can sign on the fly — point the app settings at the PFX path and
  password, restart the service.
- **Non-exportable Key Vault / HSM cert:** ScreenConnect can't be handed a
  private key it can't read. Two viable patterns:
  1. **Out-of-band signing** — let ScreenConnect emit the unsigned agent, then
     sign the produced installer artifact with AzureSignTool (step 3) before
     distribution. Good for occasional rebuilds.
  2. **Key Vault KSP/CNG** — install the Azure Key Vault key-storage provider on
     the Windows host so the OS cert store can reference the vault key; then a
     `signtool`-compatible command can sign using the cert by subject/thumbprint
     without exporting it.

Confirm the exact `web.config` keys against current ConnectWise docs for your
build — the setting names have shifted across versions.

> [!warning]
> Do **not** store the code-signing `.pfx`/password, the Key Vault client
> secret, or any ScreenConnect licence in the wiki or a tracked repo. They are
> all secrets.

## Related

- [[Self-Hosted Services]] — ScreenConnect/Control as a self-hosted service
- [[Windows Administration]] — signtool / Authenticode verification context
- [[acme.sh - DNS-01 Certificates]] — the TLS-cert counterpart to this PKI workflow
