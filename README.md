Zcash-flavored Ed25519 for use in [Zebra][zebra].

Zcash uses Ed25519 for [JoinSplit signatures][zcash_protocol_jssig] with
particular validation rules around edge cases in Ed25519 signatures.  Ed25519,
as specified in [RFC8032], does not specify behaviour around these edge cases
and so does not require conformant implementations to agree on whether a
signature is valid.  For most applications, these edge cases are irrelevant,
but in Zcash, nodes must be able to reach consensus on which signatures would
be valid, so these validation behaviors are *consensus-critical*.

Because the Ed25519 validation rules are consensus-critical for Zcash, Zebra
requires an Ed25519 library that implements the Zcash-flavored validation rules
specifically, and since it is unreasonable to expect an upstream dependency to
maintain Zcash-specific behavior, this crate provides an Ed25519 implementation
matching the Zcash consensus rules exactly.

This crate also provides an experimental interface meant to explore batch
verification of heterogeneous data, documented in the `batch` module and
feature-gated behind the `batch` feature.

## Example

```
use std::convert::TryFrom;
use rand::thread_rng;
use ed25519_zebra::*;

let msg = b"Zcash";

// Generate a secret key and sign the message
let sk = SecretKey::new(thread_rng());
let sig = sk.sign(msg);

// Types can be converted to raw byte arrays with From/Into
let sig_bytes: [u8; 64] = sig.into();
let pk_bytes: [u8; 32] = PublicKey::from(&sk).into();

// Verify the signature
assert!(
    PublicKey::try_from(pk_bytes)
        .and_then(|pk| pk.verify(&sig_bytes.into(), msg))
        .is_ok()
);
```

[zcash_protocol_jssig]: https://zips.z.cash/protocol/protocol.pdf#concretejssig
[RFC8032]: https://tools.ietf.org/html/rfc8032
[zebra]: https://github.com/ZcashFoundation/zebra