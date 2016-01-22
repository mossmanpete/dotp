# dOTP
##decentralized One Time Passwords

_Don't use me, I'm very new, untested and more of a prototype of an idea than a functional system_

### Details

Based on the NaCL encryption library from Daniel Bernstein, it uses public key encryption to build a challenge and response system for creating One Time Passwords (OTP)

Unlike typical One Time Passwords where a secret is shared between the two parties, dOTP only requires that the 'challenger' know the user's public key. The challenger then creates a random One Time Password and encrypts it with the public key of the recipient. This is then displayed to the user as a QR code which can be scanned and decrypted with a mobile app to reveal the One Time Password

Android/iOS Demo included in the repo.

#### Basic steps to using dOTP

1. Launch the mobile app and create a new keypair if you don't already have one.
2. Export your public key and give it to the authenticating server (Can be [scanned via your laptops webcam](https://mdp.github.io/dotp/scan/?redir=https%3A%2F%2Fmdp.github.io%2Fdotp%2Fdemo%2F%23%2F%3F))
3. When logging into the authentication server, the server will encode a One Time Password and encrypt with the users public key. A QR Code will then be displayed for the user to scan with the mobile app. [Example](https://mdp.github.io/dotp/demo/#/BPAkh9cmVnQYwJN5QCmoysNp89355PfNyDfApBWmuMQZL?_k=6y3749)
4. The mobile app will decrypt the message and display it to the user as a One Time Password.
5. The user will login using the displayed password.

#### Encoding a Challenge into a QRCode

Challenges are meant to be encoded as a QR Code and scanned by the authenticating user. They have the following format (Value[byte size]):

Base58Encode(Version[1]|PublicKeyFirstByte[1]|ChallengersPublicKey[32]|Box[...])

Broken down:

- Version: Currently at 0, allows the challenge protocol to change as needed
- PublicKeyFirstByte: Lets the client narrow down keys to attempt, but doesn't give away the authenticators public key
- ChallengersPublicKey: the 32 byte public key of the challenger. Should be from a new keypair each time
- Box: the NaCL cipher text, variable size

In JavaScript it looks like this:

```javascript
function(recPubKeyFirstByte, challengerPub, box) {
  var challenge = new Uint8Array(1+1+32+box.length)
  challenge[0] = VERSION
  challenge[6] = recPubKeyFirstByte
  challenge.set(challengerPub, 31)
  challenge.set(box, 63)
  return Base58.encode(challenge)
}
```

##### Cryptography behind dOTP

The basics are as follows:
- NaCL [crypto box](https://nacl.cr.yp.to/box.html) function used to encrypt the OTP, passed the following values
  - The OTP we are encrypting for the recipient/authenticator
  - Nonce consisting of 24 '0' bytes
  - Public Key of the recipient/authenticator
  - Challengers secret key *Must be a newly generated secret each time we encrypt due to the reused nonce*

`NaCl.crypto_box(otp, nonce[24 bytes], publicKey[32 bytes], ChallengerSecretKey[32 bytes])`

