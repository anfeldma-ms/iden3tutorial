# End-to-end circuit tutorial: From Powers-of-Tau Ceremony to Proof Verification in Remix

The purpose of this tutorial is to provide a guide that works with circom and snarkjs as they are currently implemented as of October 2020. I find many tutorials online are out-of-date with changes made in the last several months. Or else, I find existing tutorials have great coverage on one aspect of ZK proof generation but poor coverage of others. This tutorial strives to give equally poor coverage of all areas.

References:
1. [iden3 blog "Create your first zero-knowledge snark circuit using circom and snarkjs"](https://blog.iden3.io/first-zk-proof.html)
1. [iden3 blog "Circom and snarkjs tutorial"](https://iden3.io/blog/circom-and-snarkjs-tutorial2.html)
1. [iden3 github "Circom and snarkjs tutorial"](https://github.com/iden3/circom/blob/master/TUTORIAL.md)
1. [iden3 github "snarkjs"](https://github.com/iden3/snarkjs) - Powers of Tau ceremony
1. [kobigurk github "phase2-bn254"](https://github.com/kobigurk/phase2-bn254) - Third-party tool for Powers of Tau ceremony
1. [Remix documentation "Run & Deploy (part 2)"](https://remix-ide.readthedocs.io/en/latest/udapp.html)
1. [Kendrick "A Practical Guide To Building Zero Knowledge dApps"](https://kndrck.co/posts/practical_guide_build_zk_dapps/) - Go beyond the basic hello world multiplier circuit

Other interesting topics:
* [Remix](https://remix.ethereum.org/)
* [iden3 project](https://www.iden3.io/)
* [Move toward off-chain models](https://medium.com/@miguelmota/evolution-of-blockchain-components-to-off-chain-models-ca3649fe2c83)
* [Hermez whitepaper on zkrollups](https://hermez.io/hermez-whitepaper.pdf)

## Node.js setup

Despite what the tutorials currently say it appears Circom and snarkjs are upgraded to use features only available in the latest Node.js. I recommend install the [latest current build](https://nodejs.org/en/download/current/) (v14 at time of writing) so that you can run this tutorial.

## Setup & test Circom, snarkjs (skip if the provided local install works for you)

I have pre-installed Circom & snarkjs in this repo. If these installs work for you, jump to the next step ("ZKP hello world")

**To test Circom and snarkjs:**

```
npx circom
npx snarkjs
```

*It is alright if you see an error message, as long as the error message is roughly saying that the input file(s) for the tool can't be found. If you see a dependency error, you may have an issue with your Node install.

**To install Circom and snarkjs locally:**

```
npm install circom
npm install snarkjs
```

**To install globally:**

```
npm install -g circom
npm install -g snarkjs
```

If you run into permissions issues in Linux (i.e. `EACCES: permission denied`) then [from the iden3 tutorial](https://blog.iden3.io/first-zk-proof.html):

```
sudo npm install -global --unsafe-perm circom
sudo npm install -global --unsafe-perm snarkjs
```


**To test snarkjs:**

`npx

## "ZKP hello world": write and compile a circuit to R1CS with Circom

A compiled Rank-1 Constraint System (R1CS) is what snarkjs needs to generate a proof. Let's build and compile the simplest circuit to R1CS using Circom.

## Prover/verifier keygen: Powers of Tau ceremony

This is the full multi-party key-generation process.

## Generate a zero-knowledge proof with snarkjs and the prover key

## Verify locally with the verifier key

## Verify on-chain with Remix
