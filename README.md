# C3RB3R / .L0CK3D — Synology NAS Ransomware Recovery Research

**Open research and collaboration on the October 2026 C3RB3R / `.L0CK3D` incident affecting Synology NAS devices.**

> **Research status (2026-10-30): No publicly verified universal decryptor has been established for the file format examined here.** This repository does **not** provide a working decryptor or promise that affected files can be recovered.

## Why this repository exists

This project documents reproducible observations from encrypted files, supports comparison across affected systems, and coordinates efforts to identify practical recovery paths. Contributions from incident responders, reverse engineers, forensic analysts, Hyper Backup specialists, and other affected users are welcome.

Our priorities are:

1. **Preserve evidence and remaining intact data.**
2. **Understand the encryption format and implementation.**
3. **Validate potential key-recovery or decryption approaches against known plaintext/ciphertext pairs.**
4. **Investigate recovery from unaffected or partially encrypted backups and data blocks.**

## Incident overview

The investigated incident was identified on **October 28, 2026**. Observed characteristics include:

- Encrypted filename suffix: **`.L0CK3D`** (the character after `L` is the digit **zero**, not the letter `O`).
- Ransom note filename: **`read-me3.txt`**, headed `C3RB3R INSTRUCTIONS`.
- Affected material includes ordinary user files and parts of a Synology Hyper Backup repository.
- At least one investigated device used a **Synology SHR / Btrfs** volume.

Several devices and backup media are being assessed. **The scope of encryption and the initial intrusion vector have not been fully established.** No public attribution or propagation timeline is claimed here.

## Preliminary encrypted-file observations

We have compared selected encrypted files with their original, unencrypted counterparts. These are **case-specific observations**, not a complete or independently verified specification of all C3RB3R variants.

| Property | Observed result | Confidence / limitation |
| --- | --- | --- |
| Added end-of-file data | **588 bytes** in multiple examined pairs | Seen in the available samples; may differ between variants |
| Trailer marker | `96 4D 7F 45` | Observed byte sequence; exact semantic role not confirmed |
| Additional bytes | `82 A9 03 00` | Observed field; interpretation not confirmed |
| Mode-related values | `0x24`, `0x25`, `0x26` | Tentative relationship to file-size-dependent encryption |
| Encryption behavior | Full, prefix-only, or intermittent regions | Depends on sampled file and size; not a universal rule |

### Example: intermittent encryption

In one MP3 plaintext/ciphertext pair, the original file length was **8,254,134 bytes**. Comparison indicated three altered regions of approximately **330 KB** each, while roughly **88%** of original data bytes appeared unchanged.

This **does not** establish that 88% of the playable audio is recoverable. Untouched byte ranges may still depend on encrypted metadata, offsets, or codec structures.

Other examined pairs included a small JPEG and a PPTX file, with evidence of differing encryption coverage. **A verified cryptographic decryption of these case files has not yet been demonstrated.**

## Cryptography: open questions

A [360 Security analysis of the October 2026 Synology attack](https://news.safe.360.cn/n/13055.html) discusses **ChaCha8** and **RSA-2048** in relation to the campaign. We have **not independently confirmed** that every detail of that implementation matches the samples described above.

Key research questions:

- Does the 588-byte trailer represent the same format/build as samples analyzed by other teams?
- How are per-file symmetric keys and nonces produced and wrapped?
- Are there demonstrable weaknesses in randomness, nonce handling, key lifecycle, or segment selection?
- Can any recoverable key material remain in memory captures, swap, crash dumps, malware artifacts, or other preserved evidence?
- Which formats allow useful **partial reconstruction** without claiming complete decryption?

**Do not assume that another victim's private key, recovered key material, or decryptor will work on this case.**

## Synology Hyper Backup: a separate recovery path

An affected Hyper Backup repository was inventoried. The repository contained **202,378 files**, of which **164,136** had the `.L0CK3D` suffix and **38,242** did not. Within its `Pool` area, filename-based filtering identified:

| Candidate files without `.L0CK3D` suffix | Count |
| --- | ---: |
| `.bucket` | 6,178 |
| `.index` | 6,172 |
| Potentially corresponding bucket/index pairs | 6,168 |

The candidate `.bucket` and `.index` files total **approximately 258 GB in logical file lengths**. **None of these figures constitutes a verified recoverable-data estimate.** An unmodified filename does not prove an intact payload or usable backup index.

Current work is focused on validating representative blocks on **copies**, examining surviving metadata, and understanding whether historical backup content can be reconstructed independently of ransomware decryption.

## Related public research

- [360 Security — October 2026 Synology / Cerber incident analysis](https://news.safe.360.cn/n/13055.html)
- [soimy/c3rb3r-rescue](https://github.com/soimy/c3rb3r-rescue) — documents a successful **case-specific** recovery involving memory-derived key material; it is **not** evidence of a universal decryptor.

References are provided for technical context, not as an endorsement of third-party tools or a claim of compatibility.

## How to contribute

Please use **GitHub Discussions** for general collaboration, format comparisons, research hypotheses, and victim-to-victim information exchange. Use **Issues** for reproducible technical findings, documentation corrections, or concrete tool defects.

Useful contributions include:

- Sanitized hex offsets and byte ranges describing encrypted-file trailers.
- Original/encrypted file-size comparisons, with hashes and clear acquisition methods.
- Reproducible tests supporting or falsifying a proposed format field or encryption mode.
- Carefully documented recovery results, including validation against known original content.
- Non-destructive analysis of Synology backups, indexes, and data blocks.
- Links to original research, with publication dates and variant-specific caveats.

When reporting a finding, please state: **sample provenance (without identifying details), exact observation, method, verification, and remaining uncertainty**. Separate confirmed measurements from hypotheses.

### Sample-sharing safety

**Do not publicly upload:**

- Personal documents, photos, full NAS images, private backups, or unredacted ransom notes containing identifiers.
- NAS credentials, tokens, SSH keys, wallet or payment details, personal contact information, or internal network maps.
- Live ransomware binaries or executable payloads without an appropriate controlled research channel and clear safety review.

If a researcher needs plaintext/ciphertext pairs, begin with **non-sensitive test files**, hashes, metadata, and a mutually agreed secure transfer mechanism. Preserve originals; perform analysis on verified copies.

## Evidence and recovery standards

A recovery claim should describe **what was recovered** and **how it was validated**:

- **Full decryption:** cryptographic recovery yielding original file contents, validated where possible against a known-original SHA-256 hash.
- **Data recovery:** retrieval from backups, snapshots, unchanged blocks, or other sources, without claiming the encrypted file was decrypted.
- **Partial reconstruction:** a usable subset of content, with missing/corrupted regions explicitly documented.
- **Format analysis:** structural findings only; not a successful recovery.

We will not label a tool or method as a working decryptor without reproducible evidence.

## Safe handling principles

1. Keep affected NAS devices and original disks **offline** until a controlled response plan is established.
2. Preserve encrypted files, ransom notes, filesystem metadata, and possible system artifacts.
3. Prefer write-blocked imaging; otherwise take extra care to avoid automount, RAID/LVM activation, journal replay, filesystem repair, or writes to original media.
4. Never test unknown recovery utilities on the only copy of the data.
5. Treat advice from strangers, proposed decryptors, and uploaded binaries as untrusted until independently verified.

## Project status

| Workstream | Status |
| --- | --- |
| Incident indicators and ransom note identification | Documented |
| Selected plaintext/ciphertext comparisons | Preliminary observations available |
| 588-byte trailer / mode analysis | Under investigation |
| General-purpose decryption for these samples | **Not achieved** |
| Per-victim key recovery | **Not achieved** |
| Hyper Backup surviving-block validation | In progress |
| Offline forensic inventory and preservation | In progress |

### Scope and disclaimer

This is a volunteer/open research effort and **not** an official Synology, 360, or law-enforcement project. Findings can change as more evidence becomes available. Nothing here should be interpreted as a guarantee of recovery or a recommendation to pay a ransom.

**If you have independently verified research related to the October 2026 `.L0CK3D` Synology campaign, please open a Discussion and share the reproducible details.**
