# Cennznet Identity / Attestation

The attestation module for Cennznet's Identity Service.

## Getting Started

Hello and welcome to CENNZnet! We're happy to have users like you joining our platform and we're beyond excited to welcome you in UNfucking the world! This README should hopefully allow you to get set up and start running our attestation SDK.

### Prerequisites

This is a fairly developer oriented document, so please make sure you are familiar with how to use JavaScript, Polkadot and the Substrate/CENNZnet ecoystem before proceeding further.

### Installation

Here's the code to get started. 
Create a new folder and run `npm init -y` and then run the following commands
```
$> npm config set registry https://npm-proxy.fury.io/centrality/
$> npm login
$> npm i @cennznet/attestation @cennznet/api @cennznet/types @cennznet/util @cennznet/wallet
```

Please ensure that you are on Node 10+ and NPM 5+ for the code to compile properly.

The above should be fairly standard for a developer versed in JavaScript and NodeJS, if you need a refresher however, [here is a good starting point](https://www.sitepoint.com/beginners-guide-node-package-manager/)

### Basic Setup 

Alright, let's get started with the code. If you want the full demo code without the explanations, see the examples/folder

First create a file and name it `index.js` and write the following two lines in:

```javascript
const {Api} = require('@cennznet/api')
const {Attestation} = require('@cennznet/attestation')
```

This loads up a version of the Attestation SDK that we will be using to call the code. This will be the foundation of your attestation requests.

### Creating your wallet and issuer account

In order to send claims, we have to create a wallet, to do so we need to import `@cennznet/wallet` so go ahead and add these lines to your file under the attestation imports

```javascript
const {SimpleKeyring, Wallet} = require('@cennznet/wallet')
const { stringToU8a } = require('@cennznet/util')
const simpleKeyring = new SimpleKeyring();
const wallet = new Wallet();
```

Alright, now we will need an account from which to issue a claim from. Navigate to [CENNZnet UI](https://cennznet-ui.centrality.me/#/explorer) and create an account by navigating to Accounts -> Create Account and selecting "Raw Seed" on  the dropdown.
![Cennznet UI Create Account](https://puu.sh/CSMAK/8b693a90ca.png)

Now that you've created that, let's create an object called "issuer" in your code under the wallet instantiation

```javascript
const issuer = {
  address: '<your address here>',
  seed: stringToU8a(('<your seed here>' as any).padEnd(32, ' ')),
};
```


Substrate accepts a Uint-8 Array as the signing mechanism, and we have a string so we must convert it to a Uint-8 Array. If your seed is less than 32 characters long, use String.prototype.padEnd to pad it with blank lines so that it can become 32 characters long and is consistent with the seed.

### Funding the issuer account 
In order to issue a claim to someone, we need to have some money, so navigate to the [CENNZnet Faucet](https://cennznet-faucet-ui.centrality.me) and transfer some money to your public address. 
     
![Cennznet Faucet](https://puu.sh/CSN2t/71b9886848.png)

### Setting up the holder account

Alright, now that we've got our issuer account set up, we need to set up our holder account.
Please follow the instructions on setting up an issuer account again, but do not fund it.

You should now create an object similar to this:

```
const holder = {
  address: '<your new holder address here>',
  seed: stringToU8a(('<your new holder seed here>' as any).padEnd(32, ' ')),
};
```

### Sending a claim with one issuer

Add the following lines to your file:

```javascript
async function main () {
    simpleKeyring.addFromSeed(issuer.seed);
    const api = await Api.create({provider: 'wss://cennznet-node-0.centrality.me:9944'});
    const passphrase = '<insert issuer passphrase here>';
    await wallet.createNewVault(passphrase);
    await wallet.addKeyring(simpleKeyring);
    api.setSigner(wallet);
    const attestation = new Attestation(api);
}
main()
```

This spins up a version of our attestation api inside the main function, we will now be writing all of our code inside the main function

To create a claim add the following lines inside the main function

```javascript
const topic = 'test'
const value = '1234'
const claim = await attestation.setClaim(
    holder.address,
    topic,
    value,
);

await claim.signAndSend(issuer.address, async result => {
    if (result.type === 'Finalised' && result.events !== undefined) {
      const { data } = result.events[0].event.toJSON();
      console.log(data)
    }
});
```
Topics should be ASCII strings that don't have any non typeable characters and be at most 32 characters and Values should be strings of a max of 64 character length.

If you've done everything properly, then you should get a response object of something similar to this:
```javascript
[ '5EfqejHV2xUUTdmUVBH7PrQL3edtMm1NQVtvCgoYd8RumaP3',
  '5FPCjwLUkeg48EDYcW5i4b45HLzmCn4aUbx5rsCsdtPbTsKT',
  1952805748,
  825373492 ]
```

