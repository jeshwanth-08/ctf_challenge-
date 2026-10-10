# Grokked Cipher

We recovered a neural network from a decommissioned key-management appliance.
It was trained to encrypt short messages over a 48-symbol alphabet, and it is
the only surviving copy of the cipher.

The appliance's engineers are gone, the source code is gone, and the parameter
names were stripped on export. What is left is a bag of tensors.

The cipher the network applies is a three-stage construction. Two of those
stages are *global* -- the same for every position in the message -- and they
are what we actually need. Recover those two stages and the archive opens.

## Files

- `cipher_weights.pt` -- the stripped parameter bag (`torch.load(..., weights_only=True)`)
- `examples.json` -- 64 known plaintext/ciphertext pairs, for checking your work
- `sealed_flag.json` -- AES-256-GCM sealed flag

## Sealing

The flag is sealed under a key derived from the two global stages:

```
key   = SHA256( domain || 0x00 || canon(P1) || b"|" || canon(P2) )
canon = int64 little-endian bytes of the permutation, then b"(48,)"
nonce = SHA256(b"nonce" || key)[:12]
aad   = domain
```

`P1` and `P2` are the two global permutations, each given as a length-48 array
where `P[s]` is the symbol that stage maps `s` to. `check` in the sealed file is
`SHA256(key)[:8]`, so you can confirm a guess before decrypting.

## A warning

Do not waste your time treating this as a black box. The network's
input/output behaviour is the *composition* of the three stages, and for any
choice of the two global permutations there exists a set of per-position
stages that reproduces every observation exactly. Querying the model cannot
distinguish them. The factorisation is only visible from the inside.
