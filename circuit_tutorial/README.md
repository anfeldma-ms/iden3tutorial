# End-to-end circuit tutorial: From Powers-of-Tau Ceremony to Proof Verification in Remix

To clone this repo:

```
git clone https://github.com/abf149/iden3tutorial.git
```

**By the end of this tutorial, you will run a zero-knowledge proof verifier on the blockchain!**

The purpose of this tutorial is to provide a guide that works with circom and snarkjs as they are currently implemented as of October 2020. I find many tutorials online are out-of-date with changes made in the last several months. Or else, I find existing tutorials have great coverage on one aspect of ZK proof generation but poor coverage of others. This tutorial strives to give coverage of all areas.

This tutorial is intended to be performed in the `iden3tutorial/circuit_tutorial` directory

Here are the basic steps:
1. Setup Node.js
2. Setup & test Circom and snarkjs
3. Write a "hello world" circuit and an input file
4. Compile the circuit and generate a witness
5. Phase 1 Powers of Tau key-generation (circuit-independent)
6. Phase 2 Powers of Tau key-generation (generate the circuit-specific verification key)
7. Verify the proof locally
8. **Verify the proof on-chain with Remix**

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

## Background

* [**iden3**](https://www.iden3.io/) - Open-source ZKP-based identity management platform. Developing Circom, Zkrollup, and TrustCommunity

* [**Circom**](https://github.com/iden3/circom/) - Programming language for building circuits and compiling them to Rank-1 Constraint form, for use in zero-knowledge proofs

* [**snarkjs**](https://github.com/iden3/snarkjs) - Javascript Web Assembly implementation of zkSNARK schemes. Generates ZKP from Circom output.

* **Powers of Tau ceremony** - 2017 process [used by ZCash for multiparty key generation](https://www.zfnd.org/blog/powers-of-tau/), improving on the limitations of the prior Zcash Sprout release. ZCash employs ZKP natively and therefore relies on secure key generation at the outset.

* **Verifier key** - public verification key embedded in the Verifier smart contract.

* **Verifier smart contract** - verifies the ZKP on-chain

* [**Remix**](https://remix.ethereum.org/) - testnet for on-chain verification 

## Node.js setup

Despite what the tutorials currently say it appears Circom and snarkjs are upgraded to use features only available in the latest Node.js. I recommend you install the [latest current build](https://nodejs.org/en/download/current/) (v14 at time of writing) so that you can run this tutorial.

## Setup & test Circom and snarkjs (skip if the provided local install works for you)

I have pre-installed Circom & snarkjs in this repo. If these installs work for you, jump to the next step ("ZKP hello world")

**To test Circom and snarkjs:**

```
npx circom
npx snarkjs
```

>*For Circom:* it is alright if you see `ENOENT: no such file or directory, open '/home/abf149/codeHome/iden3tutorial/circuit_tutorial/circuit.circom'`. As long as you don't see a dependency error.

>*For snarkjs:* the tool should print it's default "Usage" message since no arguments have been provided

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

>If you run into permissions issues in Linux (i.e. `EACCES: permission denied`) then [from the iden3 tutorial](https://blog.iden3.io/first-zk-proof.html):

```
sudo npm install -global --unsafe-perm circom
sudo npm install -global --unsafe-perm snarkjs
```

## "ZKP hello world": write and compile a circuit to R1CS with Circom

A compiled Rank-1 Constraint System (R1CS) is what snarkjs needs to generate a proof. Taken from the [iden3 github tutorial](https://github.com/iden3/circom/blob/master/TUTORIAL.md), here are the steps to write and build the simplest "hello world" multiplier circuit:

### 2.1 Create a circuit in a new directory

1. Navigate to the `circuit_tutorial` directory if you are not there already

> A suggestion from the iden3 tutorial: *In a real circuit, you will probably want to create a `git` repository with a `circuits` directory and a `test` directory with all your tests, and the needed scripts to build all the circuits.*

2. Create a new file named `circuit.circom` with the following content:

```
template Multiplier() {
    signal private input a;
    signal private input b;
    signal output c;

    c <== a*b;
}

component main = Multiplier();
```

This circuit has 2 private input signals named `a` and `b` and one output named `c`.

The only thing that the circuit does is forcing the signal `c` to be the value of `a*b`

After declaring the `Multiplier` template, we instantiate it with a component named`main`.

Note: When compiling a circuit, a component named `main` must always exist.

### 2.2 Compile the circuit

Here is the process to compile the circuit, which deviates slightly from what is shown in the [iden3 github tutorial](https://github.com/iden3/circom/blob/master/TUTORIAL.md):

```
circom circuit.circom --r1cs
circom circuit.circom ---wasm
circom circuit.circom --sym
```

> I find I have problems running circom with `--r1cs`, `--wasm`, and `--sym` flags altogether in one call.

**--r1cs:** Output `.r1cs` file representing the Rank-1 Constraint System
**--wasm:** Output a Node.js `.wasm` web assembly executable for generating witness
**--sym:** Output a `.sym` file representing the circuit

**^Check that each of the above files is not empty**

### 2.3 Create the input file for your circuit

Create a file `input.json` containing the following:

```
{"a": 3, "b": 11}
```

Which are the inputs a = 3 and b = 11.

### 2.4 Calculate the witness using the input file and executable web assembly

```
npx snarkjs wtns calculate circuit.wasm input.json witness.wtns
```

Calculate the witness (given the inputs a = 3 and b = 11).

### 2.5 Recommended: debug the final witness calculation

```
npx snarkjs wtns debug circuit.wasm input.json witness.wtns circuit.sym --trigger --get --set
```

And check for any errors in the witness calculation process (best practice).

The wtns debug command logs every time a new component starts/ends (`--trigger`), when a signal is set (`--set`) and when it's read (`--get`).

### 2.6 Optional: view information about the circuit

#### 2.6.1 Export R1CS to JSON

```
npx snarkjs r1cs export json circuit.r1cs circuit.r1cs.json
cat circuit.r1cs.json
```

We export r1cs to json format to make it human readable.

#### 2.6.2 Circuit stats

```
npx snarkjs r1cs info circuit.r1cs
```

The info command is used to print circuit stats.

You should see the following output:

```
[INFO]  snarkJS: Curve: bn-128
[INFO]  snarkJS: # of Wires: 1003
[INFO]  snarkJS: # of Constraints: 1000
[INFO]  snarkJS: # of Private Inputs: 2
[INFO]  snarkJS: # of Public Inputs: 0
[INFO]  snarkJS: # of Outputs: 1
```

This information fits with our mental map of the circuit we created: we had two private inputs a and b, one output c, and a thousand constraints of the form a * b = c.

#### 2.6.3 Constraints

```
npx snarkjs r1cs print circuit.r1cs circuit.sym
```

To double check, we print the R1CS constraints of the circuit.

You should see a thousand constraints of the form:

```
[ -main.int[i] ] * [ main.int[i] ] - [ main.b -main.int[i+1] ] = 0
```

## Keygen: Phase 1 (circuit-independent) Powers of Tau ceremony (skip if you already the intermediate Phase 1 `.ptau` file generated)

The Powers of Tau ceremony has two phases, circuit-independent and circuit-dependent. I have copied the full Phase 1 Powers-of-Tau key generation process from the [snarkjs github](https://github.com/iden3/snarkjs) into a [Phase 1 (circuit-independent) tutorial snippet](../ceremony_tutorial/README.md). If you are in the development phase (not actually security sensitive) you should not need to re-run this portion for each new circuit.

## Keygen: Phase 2 (circuit-dependent) Powers of Tau ceremony

Once you have compiled your circuit with Circom, follow the steps in [Phase 2 (circuit-dependent) tutorial snippet](../ceremony_tutorial/READMEp2.md) to generate the verification key.

Phase 2 only consumes the R1CS file, *not* the witness

## Generate a zero-knowledge proof with snarkjs and the generated keys

Here is the proof-generation process, copied from the [snarkjs github](https://github.com/iden3/snarkjs):

### Create the proof

```
npx snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json
```

We create the proof. `groth16 prove` generates the files `proof.json` and `public.json`: `proof.json` contains the actual proof, whereas `public.json` contains the values of the public inputs and output.

>Note that it's also possible to create the proof and calculate the witness in the same command by running:
>`snarkjs groth16 fullprove input.json circuit.wasm circuit_final.zkey proof.json public.json`

## Verify locally with the verification key

```
npx snarkjs groth16 verify verification_key.json public.json proof.json
```

We use the `groth16 verify` command to verify the proof, passing in the `verification_key` we exported earlier.

If all is well, you should see that `OK` has been outputted to your console. This signifies the proof is valid.

## Verify on-chain with Remix

In a real-world blockchain dApp you can run the verifier on-chain. In this example we will assume an Ethereum blockchain. [Remix](https://remix.ethereum.org/) provides a browser testnet interface and their [Deploy & Run](https://remix-ide.readthedocs.io/en/latest/udapp.html) documentation gives a pretty good guide to deploying and running smart contracts on the testnet and passing in arguments. I will overview the process here.

Again, much of this is quoted from [snarkjs github](https://github.com/iden3/snarkjs):

### Turn the verifier into a smart contract

```
npx snarkjs zkey export solidityverifier circuit_final.zkey verifier.sol
```

This exports a Solidity smart contract, `verifier.sol`, which you we will subsequently publish to the testchain.

Open `verifier.sol` and look toward the end of the file; you will see the `Verifier` smart contract

```
contract Verifier {
    using Pairing for *;
    struct VerifyingKey {
        Pairing.G1Point alfa1;
        Pairing.G2Point beta2;
        Pairing.G2Point gamma2;
        Pairing.G2Point delta2;
        Pairing.G1Point[] IC;
    }
    struct Proof {
        Pairing.G1Point A;
        Pairing.G2Point B;
        Pairing.G1Point C;
    }

...
```

which contains the method

```
    /// @return r  bool true if proof is valid
    function verifyProof(
            uint[2] memory a,
            uint[2][2] memory b,
            uint[2] memory c,
            uint[1] memory input
        ) public view returns (bool r) {
        Proof memory proof;

        ...
```

with arguments
* `a`: `uint[2]`
* `b`: `uint[2][2]`
* `c`: `uint[2]`
* `input`: `uint[1]`

In the next step, we will generate these arguments as JSON arrays.

### Generate arguments to the Verifier contract call

```
npx snarkjs zkey export soliditycalldata public.json proof.json
```

We use `soliditycalldata` to simulate a verification call; **the terminal output contains the arguments to the Verifier contract** and may look different from run-to-run depending on your verification key. Examining the terminal output

```
["0x0e3dd223dc23606759cc08edf0cec7de7b18c38135eedccd30840a0b360b2997", "0x1cfb7b2415971de040ece3b77c70bd917a728bb3e52d1dcfa382807300a05780"],[["0x058885b7df176f4ca21b17a5631d44d12a8183514e6128ff58bdd2029321f3f9", "0x17b9cf57090af9de8ac5157ae30f3194faff0740fa4bad37357e1b3dbb3ed0e6"],["0x1538d977850f8a7d8db34fc3f6587b9d1ca81f1c3b6e9b5d28a413274f411bec", "0x0b20925eba69a89773e748b0dd99574224160631d8d2a1e0db81995d2a36d3b8"]],["0x21d150fdea94cb327b14255877f857a0d1e951f96d7da55028a4fcfb00cda369", "0x10ddd98cef660e9710f0c7d0592ee4af90800f9ffd264862322eac1dd0c7f5c5"],["0x0000000000000000000000000000000000000000000000000000000000000021"]
```

note that
* The first, length-2 JSON array is `a`
* The second, 2-dimensional JSON array (containing two nested length-2 1-dimensional JSON arrays) is `b`
* The third, length-2 JSON array is `c`
* The fourth and final length-1 JSON array is `input`

### Execute the Verifier smart contract in Remix

Again, the best guide to deploy and run the smart contract in Remix is [here](https://remix-ide.readthedocs.io/en/latest/udapp.html), but it does leave out a few things which slowed me down. So I will give a high-level guide to deploying the Verifier smart contract

1. Navigate to [remix.ethereum.org](https://remix.ethereum.org/)
2. Under **Featured Plugins**, choose **Solidity**
3. You should see a **Solidity Compiler** panel appear. Under **Compiler** choose the **0.6.11...** option (or most similar). The solidity file we generated earlier specifies the compiler version and Remix will complain if the versions do not correspond.
4. Make sure the language is **Solidity**
5. Optional: Choose **Auto compile** to enable automatic compilation as you work in the editor

Now we will enter our smart contract into Remix.

6. Underneath **Featured Plugins** there is a **File** heading, underneath which there is a **New File** button. Click **New File**.
7. If prompted for a name, enter **verifier.sol**. You should see a blank text editor.
8. Now, open your local **verifier.sol** file that you generated earlier using **snarkjs**
9. Copy the file contents and paste them into the editor in your browser. You should now see the smart contract code in the editor.
10. Revisit the **Solidity Compiler** panel on the left. Near the bottom, you should see a **Contract** option; this specifies which smart contract in the `.sol` file will be compiled. We want to set this to `Verifier (verifier.sol)` but it may default to `Pairing (verifier.sol)` which is a library that `snarkjs` automatically generated when we created `verifier.sol`. So just make sure that `Verifier` is selected.
11. Click `Compile verifier.sol` (may be unnecessary if you have auto-compile enabled)
12. Next, at the bottom of the **Solidity Compiler** pane you should see a list of publishing options for the smart contract, one of which should be **Publish on Swarm** or **Publish on Ipfs**. I arbitrarily chose **Publish on Swarm** and that worked well for me, but feel free to choose whichever testnet you prefer. Click the button.
13. Your ZKP proof verifier smart contract is now deployed.

Finally, we will execute our verifier smart contract and see that verification is successful.

14. On the far left of the window, you should see a narrow panel with a list of icons corresponding to blades in the Remix UI. If you mouse over each icon, a tooltip tells you the purpose of each UI blade. We are currently in the **Solidity Compiler** blade.
15. Within the far left panel, navigate to the **Deploy & run transactions** blade. The icon looks a little like the Ethereum logo and it should be right underneath.
16. You should now see a **Deploy & run transactions** panel. Most of the default options should be acceptable. **Again, make sure the Verifier contract is selected under the Contract option.**
17. Click the **Deploy** button
18. You should see activity in the browser terminal. At the bottom of the **Deploy & run transactions** panel there is a section that says **Deployed Contracts** and underneath there should be an entry that says **VERIFIER AT 0X...**. 
19. The **VERIFIER AT 0X...** entry should have a dropdown. Click the dropdown to see a list of all methods in the smart contract. There should only be one - **verifyProof**.
20. The **verifyProof** entry should also have a dropdown. Click the dropdown and you will see a place to enter each of the arguments we generated above - `a`, `b`, `c`, and `input`. They are already in the form of JSON arrays so you can simply copy-paste from the terminal.
21. Once you have entered the arguments - find the **call** button underneath the argument textboxes. Click **call** and you should see activity in the browser terminal. 
22. Examine the terminal. If necessary, drag the top of the browser terminal to expand it. You should see a green checkmark indicating successful execution. Underneath you should see `[call] from: 0x...` and next to this terminal text, you should see another dropdown.
23. Expand the dropdown. A considerable amount of text will appear; scroll to the bottom and look for `decoded output`. Next to this field, you should see

```
{ "0": "bool: r true" }
```

If so, you have successfully verified a proof! Now you can visit [this blog](https://kndrck.co/posts/practical_guide_build_zk_dapps/) to try a more advanced circuit!
