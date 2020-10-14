# snarkjs Powers of Tau ceremony tutorial Phase 1

This guide is pulled directly from the [snarksjs README](https://github.com/iden3/snarkjs) to help you with Powers of Tau key generation. It covers all of the key-generation steps which do note require the compiled circuit ("phase 1"). The output is an intermediate powers-of-tau file which you will use in the [next phase](READMEp2.md).

## The process

### 1. Start a new powers of tau ceremony
```sh
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
```

The `new` command is used to start a powers of tau ceremony.

The first parameter after `new` refers to the type of curve you wish to use. At the moment, we support both `bn128` and `bls12-381`.

The second parameter, in this case `12`, is the power of two of the maximum number of contraints that the ceremony can accept: in this case, the number of constraints is `2 ^ 12 = 4096`. The maximum value supported here is `28`, which means you can use `snarkjs` to securely generate zk-snark parameters for circuits with up to `2 ^ 28` (≈268 million) constraints.


### 2. Contribute to the ceremony
```sh
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
```

The `contribute` command creates a ptau file with a new contribution.

You'll be prompted to enter some random text to provide an extra source of entropy.

`contribute` takes as input the transcript of the protocol so far, in this case `pot12_0000.ptau`, and outputs a new transcript, in this case `pot12_0001.ptau`, which includes the computation carried out by the new contributor (`ptau` files contain a history of all the challenges and responses that have taken place so far).

`name` can be anything you want, and is just included for reference (it will be printed when you verify the file (step 5).

### 3. Provide a second contribution
```sh
snarkjs powersoftau contribute pot12_0001.ptau pot12_0002.ptau --name="Second contribution" -v -e="some random text"
```

By letting you write the random text as part of the command, the `-e` parameter allows `contribute` to be non-interactive.

### 4. Provide a third contribution using third party software
```sh
snarkjs powersoftau export challenge pot12_0002.ptau challenge_0003
snarkjs powersoftau challenge contribute bn128 challenge_0003 response_0003 -e="some random text"
snarkjs powersoftau import response pot12_0002.ptau response_0003 pot12_0003.ptau -n="Third contribution name"
```

The challenge and response files are compatible with [this software](https://github.com/kobigurk/phase2-bn254).

This allows you to use different types of software in a single ceremony.

### 5. Verify the protocol so far
```sh
snarkjs powersoftau verify pot12_0003.ptau
```

The `verify` command verifies a `ptau` (powers of tau) file. Which means it checks all the contributions to the multi-party computation (MPC) up to that point. It also prints the hashes of all the intermediate results to the console.

If everything checks out, you should see the following at the top of the output:

```sh
[INFO]  snarkJS: Powers Of tau file OK!
```

In sum, whenever a new zk-snark project needs to perform a trusted setup, you can just pick the latest `ptau` file, and run the `verify` command to verify the entire chain of challenges and responses so far.


### 6. Apply a random beacon
```sh
snarkjs powersoftau beacon pot12_0003.ptau pot12_beacon.ptau 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon"
```

The `beacon` command creates a `ptau` file with a contribution applied in the form of a random beacon.

We need to apply a random beacon in order to finalise phase 1 of the trusted setup.

> To paraphrase Sean Bowe and Ariel Gabizon, a random beacon is a source of public randomness that is not available before a fixed time. The beacon itself can be a delayed hash function (e.g. 2^40 iterations of SHA256) evaluated on some high entropy and publicly available data. Possible sources of data include: the closing value of the stock market on a certain date in the future, the output of a selected set of national lotteries, or the value of a block at a particular height in one or more blockchains. E.g. the hash of the 11 millionth Ethereum block (which as of this writing is some 3 months in the future). See [here](https://eprint.iacr.org/2017/1050.pdf) for more on the importance of a random beacon.

For the purposes of this tutorial, the beacon is essentially a delayed hash function evaluated on `0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f` (in practice this value will be some form of high entropy and publicly available data of your choice). The next input -- in our case `10` -- just tells `snarkjs` to perform `2 ^ 10` iterations of this hash function.

> Note that  [security holds](https://eprint.iacr.org/2017/1050) even if an adversary has limited influence on the beacon.

### 7. Prepare phase 2
```sh
snarkjs powersoftau prepare phase2 pot12_beacon.ptau pot12_final.ptau -v
```

We're now ready to prepare phase 2 of the setup (the circuit-specific phase).

Under the hood,  the `prepare phase2` command calculates the encrypted evaluation of the Lagrange polynomials at tau for `tau`, `alpha*tau` and `beta*tau`. It takes the beacon `ptau` file we generated in the previous step, and outputs a final `ptau` file which will be used to generate the circuit proving and verification keys.

---
**NOTE**

A ptau file for bn128 with the peraperPhase2 54 contributions and a beacon, can be found here:

https://www.dropbox.com/sh/mn47gnepqu88mzl/AACaJkBU7mmCq8uU8ml0-0fma?dl=0

There is a file truncated for each power of two.

The complete file is [powersOfTau28_hez_final.ptau](https://www.dropbox.com/sh/mn47gnepqu88mzl/AADgFSy_UnsSoDwOPy64tpCWa/powersOfTau28_hez_final.ptau?dl=0) which includes 2**28 powers.

And it's blake2b hash is:

55c77ce8562366c91e7cda394cf7b7c15a06c12d8c905e8b36ba9cf5e13eb37d1a429c589e8eaba4c591bc4b88a0e2828745a53e170eac300236f5c1a326f41a

You can find more information about the ceremony [here](https://github.com/weijiekoh/perpetualpowersoftau)

The last ptau file was geneerated using this procedure:

https://www.reddit.com/r/ethereum/comments/iftos6/powers_of_tau_selection_for_hermez_rollup/

---

### 8. Verify the final `ptau`
```sh
snarkjs powersoftau verify pot12_final.ptau
```

The `verify` command verifies a powers of tau file.

Before we go ahead and create the circuit, we perform a final check and verify the final protocol transcript.

> Notice there is no longer a warning informing you that the file does not contain phase 2 precalculated values.

