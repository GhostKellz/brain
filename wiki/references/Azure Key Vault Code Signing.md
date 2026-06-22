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

## 4. ScreenConnect / ConnectWise Control code signing

Self-hosted **ScreenConnect (ConnectWise Control)** builds a fresh access-agent
installer (`.exe` / `.msi`) every time you create a support session, deploy an
access agent, or rebuild the client. Unsigned, those binaries trip Windows
**SmartScreen** and "Unknown Publisher" UAC prompts — a real conversion-killer
for end users. Signing every generated agent with your OV/EV code-signing cert
makes them trusted and silences the warnings.

### How ScreenConnect signs

ScreenConnect performs the signing **itself, at agent-build time**, by shelling
out to Authenticode signing on the host running the service. It needs:

1. A code-signing certificate it can present (subject/thumbprint or a `.pfx`).
2. Configuration telling it which cert to use + a timestamp URL.

The configuration lives in the instance's **`web.config`** under `<appSettings>`
(on Windows, `C:\Program Files (x86)\ScreenConnect\web.config`). The relevant
keys across recent builds:

```xml
<appSettings>
  <!-- Thumbprint of a cert installed in the host's certificate store -->
  <add key="CodeSigningCertificateThumbprint" value="<cert-thumbprint>" />
  <!-- RFC-3161 timestamp authority (always set one) -->
  <add key="CodeSigningTimestampServer" value="http://timestamp.digicert.com" />
  <!-- Optional: digest algorithm -->
  <add key="CodeSigningDigestAlgorithm" value="SHA256" />
</appSettings>
```

> [!key-insight]
> Setting names have drifted across ConnectWise Control versions (older builds
> used `CodeSigningCertificatePath` + `CodeSigningCertificatePassword` pointing
> at a `.pfx`; newer builds prefer a **thumbprint** referencing the Windows cert
> store). Always confirm the exact keys against the ConnectWise docs for your
> build before editing `web.config`, and restart the **ScreenConnect** service
> after any change.

### Path A — exportable PFX (simplest, weakest)

If you still hold a legacy **exportable** OV cert as a `.pfx`:

1. Import it into the host store: `certutil -user -p <pfx-pass> -importpfx <cert>.pfx`
2. Grab the thumbprint (Certificates MMC → Details, or `Get-ChildItem Cert:\CurrentUser\My`).
3. Put the thumbprint in `web.config`, set the timestamp URL, restart the service.

ScreenConnect now signs each generated agent automatically. **This path is going
away** — post-2023 CA/B rules require keys on hardware, so new EV certs are
non-exportable and cannot live as a plain `.pfx`.

### Path B — non-exportable Key Vault / HSM cert (current standard)

A Key Vault HSM key is **non-exportable** — ScreenConnect can't be handed a
private key it cannot read. Two viable patterns:

**B1. Out-of-band post-build signing (recommended, simplest to reason about).**
Leave ScreenConnect's own signing **disabled** and sign the artifact it produces
with AzureSignTool (step 3) before distribution:

- For a fixed **access agent** (MSI/EXE you deploy at scale), build it once in
  the admin UI, download the unsigned installer, sign it with AzureSignTool, and
  host the signed copy for deployment.
- Automate it: a small watcher/script signs new agent artifacts as they're
  produced, e.g.

  ```bash
  AzureSignTool sign \
    --azure-key-vault-url "https://<vault-name>.vault.azure.net" \
    --azure-key-vault-certificate "<cert-name>" \
    --azure-key-vault-tenant-id "<tenant-id>" \
    --azure-key-vault-client-id "<app-registration-client-id>" \
    --azure-key-vault-client-secret "$AZURE_KV_CLIENT_SECRET" \
    --timestamp-rfc3161 "http://timestamp.digicert.com" \
    --file-digest sha256 -v "<agent-installer>.exe"
  ```

  Trade-off: per-session *support* installers (built ad hoc) aren't auto-signed
  this way — B1 fits the **access agent** (stable, deployed broadly) far better
  than one-off attended-support downloads.

**B2. Key Vault CNG/KSP provider (lets ScreenConnect sign HSM-backed).**
Install the **Azure Key Vault key-storage provider** (the dlib/CNG provider that
exposes the vault key to Windows CNG) on the ScreenConnect host so the OS cert
store can reference the non-exportable vault key by thumbprint. ScreenConnect's
thumbprint-based signing then works without the key ever being exported —
signing operations are proxied to the vault. This keeps ScreenConnect's
auto-sign-every-agent behaviour intact while satisfying the HSM requirement.
More moving parts (provider install, app-registration auth on the host, network
egress to Key Vault) than B1.

### Verify a signed agent

```powershell
signtool verify /pa /v "<agent-installer>.exe"
# confirm: chains to a trusted root, has a timestamp, digest = SHA256
```

> [!warning]
> Never store the code-signing `.pfx`/password, the Key Vault client secret, the
> cert thumbprint's backing key, or any ScreenConnect licence in the wiki or a
> tracked repo. Inject secrets as environment variables / from a secret store at
> sign time. Everything cert-specific here is a placeholder.

## Related

- [[Self-Hosted Services]] — ScreenConnect/Control as a self-hosted service
- [[Windows Administration]] — signtool / Authenticode verification context
- [[acme.sh - DNS-01 Certificates]] — the TLS-cert counterpart to this PKI workflow
