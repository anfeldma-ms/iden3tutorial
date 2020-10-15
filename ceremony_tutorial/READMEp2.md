# snarkjs Powers of Tau ceremony tutorial Phase 2

This guide is pulled directly from the [snarksjs README](https://github.com/iden3/snarkjs) to help you with Powers of Tau key generation. It covers all of the key-generation steps which **require** the compiled circuit. The end result is the verification key.

## The process

### 1. Generate the reference `zkey` without phase 2 contributions
```sh
snarkjs zkey new circuit.r1cs pot12_final.ptau circuit_0000.zkey
```

The `zkey new` command creates an initial `zkey` file with zero contributions.

The `zkey` is a zero-knowledge key that includes both the proving and verification keys as well as phase 2 contributions.

Importantly, one can verify whether a `zkey` belongs to a specific circuit or not.

Note that `circuit_0000.zkey` (the output of the `zkey` command above)  does not include any contributions yet, so it cannot be used in a final circuit.

*The following steps (15-20) are similar to the equivalent phase 1 steps, except we use `zkey` instead of `powersoftau` as the main command, and we generate `zkey` rather that `ptau` files.*

### 2. Contribute to the phase 2 ceremony
```sh
npx snarkjs zkey contribute circuit_0000.zkey circuit_0001.zkey --name="1st Contributor Name" -v
```

The `zkey contribute` command creates a `zkey` file with a new contribution.

As in phase 1, you'll be prompted to enter some random text to provide an extra source of entropy.


### 3. Provide a second contribution
```sh
npx snarkjs zkey contribute circuit_0001.zkey circuit_0002.zkey --name="Second contribution Name" -v -e="Another random entropy"
```

We provide a second contribution.

### 4. Provide a third contribution using third party software

```sh
npx snarkjs zkey export bellman circuit_0002.zkey  challenge_phase2_0003
npx snarkjs zkey bellman contribute bn128 challenge_phase2_0003 response_phase2_0003 -e="some random text"
npx snarkjs zkey import bellman circuit_0002.zkey response_phase2_0003 circuit_0003.zkey -n="Third contribution name"
```

And a third using [third-party software](https://github.com/kobigurk/phase2-bn254).

### 5. Verify the latest `zkey`
```sh
npx snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_0003.zkey
```

The `zkey verify` command verifies a `zkey` file. It also prints the hashes of all the intermediary results to the console.

We verify the `zkey` file we created in the previous step.  Which means we check all the contributions to the second phase of the multi-party computation (MPC) up to that point.

This command also checks that the `zkey` file matches the circuit.

If everything checks out, you should see the following:

```
[INFO]  snarkJS: ZKey Ok!
```

### 6. Apply a random beacon
```sh
npx snarkjs zkey beacon circuit_0003.zkey circuit_final.zkey 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon phase2"
```

The `zkey beacon` command creates a `zkey` file with a contribution applied in the form of a random beacon.

We use it to apply a random beacon to the latest `zkey` after the final contribution has been made (this is necessary in order to generate a final `zkey` file and finalise phase 2 of the trusted setup).

### 7. Verify the final `zkey`
```sh
npx snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_final.zkey
```

Before we go ahead and export the verification key as a `json`, we perform a final check and verify the final protocol transcript (`zkey`).

### 8. Export the verification key
```sh
npx snarkjs zkey export verificationkey circuit_final.zkey verification_key.json
```
We export the verification key from `circuit_final.zkey` into `verification_key.json`.
