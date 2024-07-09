# circom-circuit

# NFT Project README

## Overview
This project involves creating and deploying a Zero-Knowledge Proof (ZKP) using Circom and SnarkJS. We will:
1. Write a correct `circuit.circom` implementation.
2. Compile the circuit to generate circuit intermediaries.
3. Generate a proof using given inputs.
4. Deploy a Solidity verifier to the Sepolia or Mumbai Testnet.
5. Call the `verifyProof()` method on the verifier contract and assert the output is true.

## Prerequisites
- Node.js and npm
- Circom and SnarkJS
- Solidity development environment (e.g., Hardhat)
- An Ethereum wallet (e.g., MetaMask) configured for the Sepolia or Mumbai Testnet
- Enough test ETH or MATIC for deploying contracts and making transactions

## Setup

### Install Dependencies
1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-repo/nft-project.git
   cd nft-project
   ```

2. **Install Node.js dependencies:**
   ```bash
   npm install
   ```

3. **Install Circom and SnarkJS:**
   ```bash
   npm install -g circom snarkjs
   ```

## Circom Circuit

### Create the Circuit
1. **Create `circuit.circom`:**
   ```plaintext
   template LessThan() {
       signal input a;
       signal input b;
       signal output out;

       out <== a < b;
   }

   component main = LessThan();
   ```

### Compile the Circuit
2. **Compile the circuit:**
   ```bash
   circom circuit.circom --r1cs --wasm --sym
   ```

3. **Generate the zkey:**
   ```bash
   snarkjs groth16 setup circuit.r1cs pot12_final.ptau circuit_0000.zkey
   snarkjs zkey contribute circuit_0000.zkey circuit_final.zkey --name="1st Contributor" -v
   ```

## Generate Proof

1. **Generate the witness:**
   ```bash
   node circuit_js/generate_witness.js circuit_js/circuit.wasm input.json witness.wtns
   ```

   `input.json`:
   ```json
   {
     "a": 0,
     "b": 1
   }
   ```

2. **Generate the proof and public signals:**
   ```bash
   snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json
   ```

## Verifier Contract

### Deploy Verifier Contract

1. **Export the verifier:**
   ```bash
   snarkjs zkey export solidityverifier circuit_final.zkey Verifier.sol
   ```

2. **Deploy the verifier contract using Hardhat:**

   **Hardhat Deployment Script (`scripts/deploy.js`):**
   ```javascript
   async function main() {
       const Verifier = await ethers.getContractFactory("Verifier");
       const verifier = await Verifier.deploy();
       await verifier.deployed();
       console.log("Verifier deployed to:", verifier.address);
   }

   main().catch((error) => {
       console.error(error);
       process.exitCode = 1;
   });
   ```

   **Deploy the contract:**
   ```bash
   npx hardhat run scripts/deploy.js --network sepolia
   ```

## Verify Proof

1. **Call `verifyProof()` method:**
   ```javascript
   const proof = require('./proof.json');
   const publicSignals = require('./public.json');

   async function verifyProof() {
       const Verifier = await ethers.getContractFactory("Verifier");
       const verifier = await Verifier.attach("YOUR_DEPLOYED_CONTRACT_ADDRESS");

       const proofArray = [
           proof.pi_a[0], proof.pi_a[1],
           proof.pi_b[0][0], proof.pi_b[0][1], proof.pi_b[1][0], proof.pi_b[1][1],
           proof.pi_c[0], proof.pi_c[1]
       ];

       const publicSignalsArray = publicSignals;

       const result = await verifier.verifyProof(proofArray, publicSignalsArray);
       console.log("Verification result:", result);
       console.assert(result, "Proof verification failed!");
   }

   verifyProof().catch((error) => {
       console.error(error);
       process.exitCode = 1;
   });
   ```

## Conclusion
By following these steps, you will successfully create a zero-knowledge proof, deploy a verifier contract, and verify the proof on the Sepolia or Mumbai Testnet.
- - -
