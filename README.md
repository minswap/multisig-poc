Goal: We need a multisig wallet with require 2/3 signatures to unlock funds inside

1. Generate keys and compute pub key hash

```bash
# Alice
cardano-cli address key-gen --verification-key-file alice.vkey --signing-key-file alice.skey
cardano-cli address key-hash --payment-verification-key-file alice.vkey > alice.vkeyhash

# Bob
cardano-cli address key-gen --verification-key-file bob.vkey --signing-key-file bob.skey
cardano-cli address key-hash --payment-verification-key-file bob.vkey > bob.vkeyhash

# Charlie
cardano-cli address key-gen --verification-key-file charlie.vkey --signing-key-file charlie.skey
cardano-cli address key-hash --payment-verification-key-file charlie.vkey > charlie.vkeyhash
```

2. Create `script.json` with following content

```json
{
  "type": "atLeast",
  "required": 2,
  "scripts": [
    {
      "type": "sig",
      "keyHash": "<alice_vkey_hash>"
    },
    {
      "type": "sig",
      "keyHash": "<bob_vkey_hash>"
    },
    {
      "type": "sig",
      "keyHash": "<charlie_vkey_hash>"
    }
  ]
}
```

3. Generate script address

```bash
cardano-cli address build --payment-script-file script.json --testnet-magic 1097911063 > testnet.addr
```

4. Send some funds to above address

5. Try to unlock funds with just one signature, it should fail

```bash
cardano-cli transaction build \
  --tx-in 6dd7473526f6d1c74c942c19fd7e55fdee27b491716de1bc958bbf29d87db295#0 \
  --tx-in-script-file script.json \
  --tx-out addr_test1qqwhkez66hzs52h3gk5ywc7nhjwuf6kq4mwctdx2gfs0zsd26zs7vejeukqgtydn6wtg7jsma5hp796u8hrdn0ew8svqnnzzwx+100000000 \
  --change-address addr_test1wq2nsw556ps3lq4qcfcwghwn56zgh9v4al2gr678zflrznq6x44ym \
  --required-signer-hash $(cat alice.vkeyhash) \
  --testnet-magic 1097911063 \
  --out-file tx.raw

cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file alice.skey \
  --out-file tx.signed

cardano-cli transaction submit --tx-file tx.signed --testnet-magic 1097911063
```

6. Now let's do it properly

6.1. Build raw transaction

```bash
cardano-cli transaction build \
  --tx-in 6dd7473526f6d1c74c942c19fd7e55fdee27b491716de1bc958bbf29d87db295#0 \
  --tx-in-script-file script.json \
  --tx-out addr_test1qqwhkez66hzs52h3gk5ywc7nhjwuf6kq4mwctdx2gfs0zsd26zs7vejeukqgtydn6wtg7jsma5hp796u8hrdn0ew8svqnnzzwx+100000000 \
  --change-address addr_test1wq2nsw556ps3lq4qcfcwghwn56zgh9v4al2gr678zflrznq6x44ym \
  --required-signer-hash $(cat alice.vkeyhash) \
  --required-signer-hash $(cat bob.vkeyhash) \
  --testnet-magic 1097911063 \
  --out-file tx.raw
```

6.2. Alice sign and share witness file (signature)

```bash
cardano-cli transaction witness \
  --tx-body-file tx.raw \
  --signing-key-file alice.skey \
  --out-file alice.wit
```

6.3. Bob sign and share witness file

```bash
cardano-cli transaction witness \
  --tx-body-file tx.raw \
  --signing-key-file bob.skey \
  --out-file bob.wit
```

6.4. Create final tx with Alice and Bob's signatures

```bash
cardano-cli transaction assemble \
  --tx-body-file tx.raw \
  --witness-file alice.wit \
  --witness-file bob.wit \
  --out-file tx.signed
```

6.5. Submit transaction

```bash
cardano-cli transaction submit --tx-file tx.signed --testnet-magic 1097911063
```
