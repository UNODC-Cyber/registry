# registry - published certificate records

This repository holds the signed public registry that the certificate verifier at https://unodc-cyber.github.io/
checks. It is the data side of the UNODC Cybercrime Programme training-certificate verification service.

**Status: nothing is published yet.** No snapshot, shard, manifest or key is stored here. Records appear only after the
verification service is launched; until then the verifier says that the service is not yet live.

## How the registry is read

The registry has **no web page**. It is plain data, read by the verifier directly from one fixed address:

    https://raw.githubusercontent.com/UNODC-Cyber/registry/main/

The verifier pins this address in its own reviewed code and never takes it from a link, a query parameter or the
registry itself. Every file is treated as untrusted bytes: it is hashed and checked against the signed manifest before
anything in it is used. The full verification rules are in the verifier's `SPEC.md`
(https://github.com/UNODC-Cyber/unodc-cyber.github.io/blob/main/SPEC.md, sections 4 and 7).

## Layout of a snapshot (19 files)

All JSON files are canonical: UTF-8, keys sorted, no insignificant whitespace, entries ordered by key, no timestamps
inside shards.

| Path | Content |
|---|---|
| `manifest.json` | signed index: schema version, snapshot number `seq`, `asOf` (last content change), `issued` (when signed), `expires`, number of entries, signing key id `kid`, a neutral `generator` version string, and the SHA3-512 digest of `meta.json` and of every shard |
| `manifest.sig.json` | hybrid signature (ML-DSA-65 and Ed25519; both must verify) over the exact bytes of `manifest.json` |
| `meta.json` | schema version and the certificate-type code table (`CMP` completion, `ATT` attendance, `APP` appreciation, `TRN` contribution) |
| `r/0.json` ... `r/f.json` | 16 shards; an entry is stored in the shard named by the first hexadecimal digit of the SHA3-512 hash of its certificate identifier |

The signing keys are **not stored here**. They are pinned in the verifier site's `keys.json`, so that keys and data never
come from the same place, and a registry can never introduce its own key.

## Record schema (public interface)

An entry is stored under an opaque key: the base64url encoding of the SHA3-512 hash of the 27-symbol certificate
identifier. The identifier itself is never stored, so a valid identifier cannot be harvested from the registry. Each
entry carries:

- `s` - public status: `V` (valid) or `R` (revoked)
- `t` - public type code from `meta.json`
- a sealed payload (authenticated encryption) that only the certificate holder can open, together with the
  key-derivation parameters, salt, key check values and nonces needed to open it

The verifier shows exactly one state to the public: Valid (with the certificate type), Revoked, or Not found.

## Privacy

- This repository contains **no personal data**: no names, no e-mail addresses, no organisation names and no course
  details in readable form.
- Identifiers are hashed; the hash cannot be reversed to a certificate number, and records cannot be browsed by person.
- Course details and the holder's name are sealed. Opening them needs the holder's own family name or e-mail address
  plus the reveal code printed on the certificate; neither is stored here.
- Do not open issues or pull requests containing personal data. If you believe personal data is present in this
  repository, report it privately as described in `SECURITY.md`.

## Snapshots

Each published snapshot replaces the previous files in one commit, with a higher `seq`. The verifier shows the date of
the snapshot it checked and refuses a snapshot older than one it has already accepted.

## Reuse

Certificate data is not licensed for reuse. See `DATA-NOTICE.md`.
