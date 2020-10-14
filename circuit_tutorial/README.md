# End-to-end circuit tutorial: From Powers-of-Tau Ceremony to Proof Verification in Remix

The purpose of this tutorial is to provide a guide that works with circom and snarkjs as they are currently implemented as of October 2020. I find many tutorials online are out-of-date with changes made in the last several months. Or else, I find existing tutorials have great coverage on one aspect of ZK proof generation but poor coverage of others. This tutorial strives to give equally poor coverage of all areas.

This tutorial is intended to be performed in the `iden3tutorial/circuit_tutorial` directory

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

Despite what the tutorials currently say it appears Circom and snarkjs are upgraded to use features only available in the latest Node.js. I recommend you install the [latest current build](https://nodejs.org/en/download/current/) (v14 at time of writing) so that you can run this tutorial.

## Setup & test Circom and snarkjs (skip if the provided local install works for you)

I have pre-installed Circom & snarkjs in this repo. If these installs work for you, jump to the next step ("ZKP hello world")

**To test Circom and snarkjs:**

```
npx circom
npx snarkjs
```

>*For Circom:* it is alright if you see `ENOENT: no such file or directory, open '/home/abf149/codeHome/iden3tutorial/circuit_tutorial/circuit.circom'`. As long as you don't see a dependency error.

>*For snarkjs:* you should see a Usage guide locally.

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

1. Create an empty directory called `factor` within the current directory (`circuit_tutorial` )where you will put all the files that you will use to write the circuit:

```
mkdir factor
cd factor
```

> In a real circuit, you will probably want to create a `git` repository with a `circuits` directory and a `test` directory with all your tests, and the needed scripts to build all the circuits.

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

## Prover/verifier keygen: Powers of Tau ceremony

This is the full multi-party key-generation process.

## Generate a zero-knowledge proof with snarkjs and the prover key

## Verify locally with the verifier key

## Verify on-chain with Remix
