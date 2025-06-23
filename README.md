### Introduction

An example repo to verify Noir circuits (with bb backend) using a Solidity verifier.

- `/circuits` - contains the Noir circuits.
- `/contract` - Foundry project with a Solidity verifier and a Test contract that reads proof from a file and verifies it.
- `/js` - JS code to generate proof and save as a file.


Tested with Noir 1.0.0-beta.3 and bb 0.82.2
## Circuit Logic

The Noir circuit checks if `x * 2 + y == expected`, where:
- `x` is a private input
- `y` and `expected` are public inputs

### Installation / Setup
```ssh
# Foundry
git submodule update

# Build circuits, generate verifier contract
(cd circuits && ./build.sh)

# Install JS dependencies
(cd js && yarn)

```  

### Proof generation in JS


```ssh
# Use bb.js to generate proof and save to a file
(cd js && yarn generate-proof)

# Run foundry test to read generated proof and verify
(cd contract && forge test --optimize --optimizer-runs 5000 --gas-report -vvv)

```

### Proof generation with bb cli

```ssh
cd circuits

# Generate witness
nargo execute

# Generate proof
bb prove -b ./target/noir_solidity.json -w target/noir_solidity.gz -o ./target --oracle_hash keccak

# Split this proof and public inputs into two files

# Convert proof to hex, and slice first 4 bytes of metadata
PROOF_HEX=$(cat ./target/proof | od -An -v -t x1 | tr -d $' \n' | sed 's/^.\{8\}//')

NUM_PUBLIC_INPUTS=2
HEX_PUBLIC_INPUTS=${PROOF_HEX:0:$((32 * $NUM_PUBLIC_INPUTS * 2))}
SPLIT_HEX_PUBLIC_INPUTS=$(sed -e 's/.\{64\}/0x&,/g' <<<$HEX_PUBLIC_INPUTS)
PROOF_WITHOUT_PUBLIC_INPUTS="${PROOF_HEX:$((NUM_PUBLIC_INPUTS * 32 * 2))}"

# Save proof and public inputs to files
echo $PROOF_WITHOUT_PUBLIC_INPUTS | xxd -r -p > ./target/proof
echo "[\"$SPLIT_HEX_PUBLIC_INPUTS\"]" > ./target/public-inputs

# Run foundry test to read generated proof and verify
cd ..
(cd contract && forge test --optimize --optimizer-runs 5000 --gas-report -vvv)
```

### 🔁 Dual Workflow Support (CLI and JS)

The project supports two approaches for generating proofs:

- **JavaScript-based workflow** using `bb.js`
- **CLI-based workflow** using `nargo` and `bb` directly

### ▶️ Running the Circuit and Tests

Use the `run.sh` script with an argument to select the approach:

```bash
# JavaScript workflow using bb.js
./circuits/run.sh js

# CLI workflow using nargo + bb
./circuits/run.sh cli
```
This script will:
1. Build the circuit
2. Generate a proof
3. Run Solidity tests using Foundry
### 🛠 Building the Solidity Verifier
Use the `build.sh` script to compile the circuit and generate the Solidity verifier:
```bash
./build.sh
```
This will:
- Compile the Noir circuit
- Generate the verification key (`vk`)
- Export the Solidity verifier to `contract/Verifier.sol`


