
  
  

#  WebAuthn Implementation (WIP NOT READY!)
- Implementation Owner: (@PineappleIOnic)
- Start Date: (31-05-2021)
- Target Date: N/A
- Appwrite Discussion: [https://github.com/appwrite/appwrite/discussions/907](https://github.com/appwrite/appwrite/discussions/907)

##  Summary

[summary]:  #summary

WebAuthn is a web API that most [major browsers support](https://caniuse.com/webauthn) and is designed to reduce and eventually replace the use of passwords in modern web applications instead opting to use physical means such as Biometric devices like Fingerprint and Iris identification (Platform Identification) or [FIDO Compliant Devices](https://fidoalliance.org/certification/fido-certified-products/) such as Yubikey and Google's Titan Key (External Idenification). The basis of webauthn is to instead use cryptography to handle authentication by giving the user's device a challenge to sign and validating the response with the initial key that was setup when they registered.

WebAuthn is the Web API that gives us the ability to tap into these modern security methods through one easy and simple to use API. Implementing it gives appwrite users the ability to go passwordless.

##  Problem Statement (Step 1)

[problem-statement]:  #problem-statement

**What problem are you trying to solve?**

Using WebAuthn tackles many issues that modern users of the internet have to deal with such as phishing, having to remember complex passwords (or just using the same common ones). Replacing passwords with more secure means of authentication.

**What is the context or background in which this problem exists?**

Appwrite does not currently support WebAuthn or Passwordless means of authentication nor does it support hardware 2FA Devices (FIDO2)

**Once the proposal is implemented, how will the system change?**

A new authentication method will be introduced in the Web SDK which will allow users to register and Sign-in using the WebAuthn API.

##  Design proposal (Step 2)

[design-proposal]:  #design-proposal

###  New Server Methods

A couple new server methods will need to be created for Registering and Signing-in with WebAuthn

###  New Web SDK Methods

A new authentication method will be introduced into the Web SDK that will allow users to register using public keys generated from the WebAuthn API.

####  Registering a user using WebAuthn

The flow will most likely go as follows:

1. User requests registration from Server sending all details we need apart from a password (email, username, etc...) `POST /v1/webauthn`

2. Server responds with a challenge (cryptographic randomly generated string), a user ID and relying party info

<details>
<summary>First Draft Example Response:</summary>
<br />

```json
{
  "challenge": "bWwvPTa6dRQrXwNXP1vldYqtSUyMdpHU36h9tcSG", // NOTE: Will be converted to Uint8Array by Client SDK
  "uid": "FZtJm7I9NR9iQcIyTwGEVAkMNQ62M7MlCDcraQ3y", // NOTE: Will be converted to Uint8Array by Client SDK
  "partyinfo": {
    "name": "Name of project or something else (note: this gets displayed to the user.)"
  }
}
```
</details>

3. Client calls the webauthn API to generate a new private and public key using a user's authentication method (Biometric Device, FIDO Devices, etc...)

<details>
<summary>Example Webauthn JS Code:</summary>
<br>

```js
const publicKey = {
    attestation: "none",
    challenge: "Challenge sent by server.", // NOTE: The type needs to be Uint8Array. Using a string for an example.
    rp: {
        name: "partyInfo name given by server",
        id: "Origin Domain (if omited defaults to current domain)"
    },
    timeout: 60000, // 1 Minute in seconds
    user: {
        name: "User's Email Address",
        displayName: "User's Username or Email Address",
        id: "uid Generated from server" // NOTE: The type needs to be Uint8Array. Using a string for an example.
    },
    pubKeyCredParams: [{
        type: "public-key",
        alg: -7
    }],
}

navigator.credentials.create({
        publicKey
    })
    .then(function(newCredentialInfo) {
        // send attestation response and client extensions
        // to the server to proceed with the registration
        // of the credential

        // newCredentialInfo stores something like:
        // {
        //   "id": "X9FrwMfmzj...",
        //   "response": {
        //     "attestationObject": "o2NmbXRoZmlk...",  // Authenticator Data
        //     "clientDataJSON": "eyJjaGFsbGVuZ..." // 
        //   },
        //   "clientExtensionResults": {}
        // }
        //
        //
    }).catch(function(err) {
        console.error(err);
    });
```

</details>


4. newCredentialInfo is sent back to the server `POST /v1/webauthn/finalise` which verifies it like so:
`NOTE: TODO`

6. If everything checks out the new user is created! 🥳

####  Signing in a WebAuthn User
1. Ask the server to sign in. The server will respond with a challenge

2. Use the WebAuthn API to sign the challenge with the private key that was generated using a user's authentication method (Biometric Device, FIDO Devices, etc...)

3. Give the server the signed challenge back

4. If signed challenge is confirmed to originate from the same private key which generated public key stored on the server then sign in the user! 🥳

###  Updating Documentation
Documentation will need to be updated to show users how to use the new methods in the Web SDK that allows for WebAuthn

###  Tutorial

A tutorial on a standard Appwrite login and register page that supports WebAuthn will need to be created.

###  Prior art

[prior-art]:  #prior-art

-  [Webauthn Demo by Duo Labs](https://webauthn.io/)

-  [Webauthn Guide by Duo Labs](https://webauthn.guide/)

-  [MDN Docs on Webauthn](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API)

-  [FIDO Alliance Members](https://fidoalliance.org/members/)

- [Proposed WebAuthn PHP Library](https://github.com/web-auth/webauthn-framework)
- [LIbrary Docs](https://webauthn-doc.spomky-labs.com/)

Note: To test WebAuthn Demos you will need a FIDO Compatable security key. These include the following:
 - Android Devices can act as security keys in their browsers
 - Yubikey
 - Windows Hello (A device with Hardware verification and a TPM Chip)
 - Any apple device with a fingerprint sensor

###  Unresolved questions

[unresolved-questions]:  #unresolved-questions

- Where to place in user object

- How to implement into other client SDK's (if possible)
  
###  Future possibilities

[future-possibilities]:  #future-possibilities

- 2FA Authentication with WebAuthn

### TODO
 - Add more example code
