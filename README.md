# noble-secp256k1

[secp256k1](https://en.bitcoin.it/wiki/Secp256k1), an elliptic curve that could be used for assymetric encryption and ECDSA signature scheme.

### This library belongs to *noble* crypto

> **noble-crypto** — high-security, easily auditable set of contained cryptographic libraries and tools.

- No dependencies, one small file
- Easily auditable TypeScript/JS code
- Uses es2019 bigint. Supported in Chrome, Firefox, node 10+
- All releases are signed and trusted
- Check out all libraries:
  [secp256k1](https://github.com/paulmillr/noble-secp256k1),
  [ed25519](https://github.com/paulmillr/noble-ed25519),
  [ripemd160](https://github.com/paulmillr/noble-ripemd160)

## Usage

> npm install noble-secp256k1

```js
import * as secp256k1 from "noble-secp256k1";

// You can also pass BigInt:
// const PRIVATE_KEY = 0xa665a45920422f9d417e4867efn;
const PRIVATE_KEY = Uint8Array.from([
  0xa6, 0x65, 0xa4, 0x59, 0x20, 0x42, 0x2f,
  0x9d, 0x41, 0x7e, 0x48, 0x67, 0xef
]);
const MESSAGE_HASH = "9c1185a5c5e9fc54612808977ee8f548b2258d31";

const publicKey = secp256k1.getPublicKey(PRIVATE_KEY);
const signature = secp256k1.sign(MESSAGE_HASH, PRIVATE_KEY);
const isMessageSigned = secp256k1.verify(signature, MESSAGE_HASH, publicKey);
```

## API

```typescript
function getPublicKey(privateKey: Uint8Array, isCompressed?: false): Uint8Array;
function getPublicKey(privateKey: string, isCompressed?: false): string;
function getPublicKey(privateKey: bigint): Point;
```
`privateKey` will be used to generate public key.
  Public key is generated by doing scalar multiplication of a base Point(x, y) by a fixed
  integer. The result is another `Point(x, y)` which we will by default encode to hex Uint8Array.
`isCompressed` (default is `false`) determines whether the output should contain `y` coordinate of the point.

```typescript
function sign(hash: Uint8Array, privateKey: Uint8Array | bigint, k?: bigint): Uint8Array;
function sign(hash: string, privateKey: string | bigint, k?: bigint): string;
```
- `hash: Uint8Array | string` - message hash which would be signed
- `privateKey: Uint8Array | string | bigint` - private key which will sign the hash
- `k?: bigint` - *optional* random seed. Default is one from `crypto.getRandomValues()`. **Must be cryptographically secure**, which means `Math.random()` won't work.
- Returns DER encoded ECDSA signature, as hex uint8a / string.

```typescript
function verify(signature: Uint8Array | string | SignResult, hash: Uint8Array | string): boolean
```
- `signature: Uint8Array` - object returned by the `sign` function
- `hash: string | Uint8Array` - message hash that needs to be verified
- `publicKey: string | Point` - e.g. that was generated from `privateKey` by `getPublicKey`
- Returns `boolean`: `true` if `signature == hash`; otherwise `false`

The library also exports helpers:

```typescript
// 𝔽p
secp256k1.P // 2 ^ 256 - 2 ^ 32 - 977

// Prime order
secp256k1.PRIME_ORDER // 2 ^ 256 - 432420386565659656852420866394968145599

// Base point
secp256k1.BASE_POINT // new secp256k1.Point(x, y) where
// x = 55066263022277343669578718895168534326250603453777594175500187360389116729240n
// y = 32670510020758816978083085130507043184471273380659243275938904335757337482424n;

// Elliptic curve point
secp256k1.Point {
  constructor(x: bigint, y: bigint);
  // Compressed elliptic curve point representation
  static fromHex(hex: Uint8Array | string);
  static fromCompressedHex(hex: string);
  toHex(): string;
  toCompressedHex(): string;
}
secp256k1.SignResult {
  constructor(r: bigint, s: bigint);
  // DER encoded ECDSA signature
  static fromHex(hex: Uint8Array | string);
  toHex()
}
```

## Security

Noble is production-ready & secure. Our goal is to have it audited by a good security expert.

We're using built-in JS `BigInt`, which is "unsuitable for use in cryptography" as [per official spec](https://github.com/tc39/proposal-bigint#cryptography). This means that the lib is vulnerable to [timing attacks](https://en.wikipedia.org/wiki/Timing_attack). But:

1. JIT-compiler and Garbage Collector make "constant time" extremely hard to achieve in a scripting language.
2. Which means *any other JS library doesn't use constant-time bigints*. Including bn.js or anything else. Even statically typed Rust, a language without GC, [makes it harder to achieve constant-time](https://www.chosenplaintext.ca/open-source/rust-timing-shield/security) for some cases.
3. Overall they are quite rare; for our particular usage they're unimportant. If your goal is absolute security, don't use any JS lib — including bindings to native ones. Try LibreSSL & similar low-level libraries & languages.
4. We however consider infrastructure attacks like rogue NPM modules very important; that's why it's crucial to minimize the amount of 3rd-party dependencies & native bindings. If your app uses 500 dependencies, any dep could get hacked and you'll be downloading rootkits with every `npm install`. Our goal is to minimize this attack vector.

## License

MIT (c) Paul Miller (https://paulmillr.com), see LICENSE file.