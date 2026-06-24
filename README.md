# cusf_sidechain_proto

Protobuf definitions for the CUSF Drivechain stack: the BIP300/301 enforcer,
its wallet, and the interface sidechain nodes implement. The services here are
served over [Connect](https://connectrpc.com) (gRPC-compatible), so they can be
called from gRPC clients or over plain HTTP.

The module is published as `buf.build/LayerTwo-Labs/cusf`.

## What's here

```
proto/cusf/
  common/v1/common.proto       shared hex encodings (ConsensusHex, Hex, ReverseHex)
  crypto/v1/crypto.proto       CryptoService: hashing + secp256k1 helpers
  mainchain/v1/common.proto    OutPoint, SidechainDeclaration
  mainchain/v1/validator.proto ValidatorService: BIP300/301 chain state
  mainchain/v1/wallet.proto    WalletService: mainchain wallet
  sidechain/v1/sidechain.proto SidechainService: interface a sidechain node exposes
```

`ValidatorService` and `WalletService` are implemented by the
[bip300301_enforcer](https://github.com/LayerTwo-Labs/bip300301_enforcer), which
is the reference server. `SidechainService` is the shape a sidechain node (for
example thunder, bitnames, bitassets) exposes back to the stack.

## Using it

Consumers generate code straight from this repo with
[buf](https://buf.build/docs/installation). You can point at the published
module or at the git repository directly.

From the module:

```yaml
# buf.yaml
deps:
  - buf.build/LayerTwo-Labs/cusf
```

Or generate against the repo URL, which is what the existing consumers do:

```bash
# Rust (see bip300301_enforcer/buf.gen.yaml)
buf generate --template buf.gen.yaml https://github.com/LayerTwo-Labs/cusf_sidechain_proto.git

# TypeScript / Go / Dart follow the same pattern with their own templates
buf generate --template buf.web.gen.yaml https://github.com/LayerTwo-Labs/cusf_sidechain_proto.git
```

Once a server is running you can hit any unary method over HTTP without a
generated client:

```bash
curl http://localhost:50051/cusf.mainchain.v1.ValidatorService/GetChainInfo
```

## Development

You need [buf](https://buf.build/docs/installation) installed. The same two
checks that run in CI:

```bash
buf lint
buf format --diff --exit-code   # add -w to apply
```

## Contributing

A few conventions that keep this module easy to depend on:

- Changes should be backwards compatible. Add fields and methods, don't renumber
  or repurpose existing ones. Use `reserved` for removed field numbers.
- Document new messages, fields, and RPCs with `///` comments. Several consumers
  surface these directly.
- Keep packages versioned (`v1`) and field/RPC naming consistent with the
  surrounding file.
