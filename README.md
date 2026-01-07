# Blockchain Implementation

A Python implementation of a blockchain system supporting both Proof of Work (PoW) and Proof of Authority (PoA) consensus mechanisms. This project implements core blockchain functionality including block validation, transaction handling, Merkle tree computation, and chain management.

## Project Structure

```
p1/
├── blockchain/          # Core blockchain package
│   ├── __init__.py     # Package initialization and database setup
│   ├── block.py        # Abstract Block class with validation logic
│   ├── chain.py        # Blockchain data structure and chain operations
│   ├── pow_block.py    # Proof of Work block implementation
│   ├── poa_block.py    # Proof of Authority block implementation
│   ├── transaction.py  # Transaction and UTXO model
│   └── util.py         # Utility functions (hashing, encoding, etc.)
├── tests/              # Test suite
│   ├── hash.py         # Hash function tests
│   ├── weight.py       # Block weight calculation tests
│   ├── get_chain.py    # Chain traversal tests
│   ├── validity.py    # Block validation tests
│   ├── poa.py         # PoA mining tests
│   └── merkle.py      # Merkle root calculation tests
├── config.py           # Configuration (database path, keys, peers)
├── run_all_tests.py    # Test runner script
└── add_random_pow_blockchain.py  # Script to generate test blockchain
```

## Features

- **Block Validation**: Comprehensive validation including:
  - Merkle root verification
  - Hash integrity checks
  - Transaction limit enforcement (max 900 per block)
  - Genesis block validation
  - Parent block existence and height verification
  - Timestamp validation
  - Seal validation (PoW/PoA)
  - Double-spend prevention
  - Transaction validity checks
  - Money creation prevention

- **Consensus Mechanisms**:
  - **Proof of Work (PoW)**: Mining with difficulty targets and nonce searching
  - **Proof of Authority (PoA)**: Block signing with ECDSA signatures

- **Transaction System**: UTXO (Unspent Transaction Output) model with:
  - Transaction inputs and outputs
  - User consistency validation
  - Input/output lookup and validation

- **Merkle Tree**: Merkle root calculation for transaction integrity

## Prerequisites

- Python 3.6 or higher
- pip (Python package manager)

## Installation

1. **Clone the repository** (or navigate to the project directory):
   ```bash
   cd p1
   ```

2. **Create a virtual environment** (recommended):
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

   The required packages are:
   - `zodb` - ZODB database for persistent storage
   - `ecdsa` - Elliptic curve cryptography for PoA signatures
   - `flask` - Web framework (if using web interface)
   - `matplotlib` - Plotting library (if using visualization)

## Running the Project

### Running Tests

To run all tests:
```bash
python run_all_tests.py
```

This will execute all test suites:
- Hash function tests (`tests.hash`)
- Block weight tests (`tests.weight`)
- Chain traversal tests (`tests.get_chain`)
- Block validation tests (`tests.validity`)
- PoA mining tests (`tests.poa`)
- Merkle root tests (`tests.merkle`)

### Running Individual Test Suites

You can also run individual test files:
```bash
python -m unittest tests.hash
python -m unittest tests.weight
# etc.
```

### Generating a Test Blockchain

To generate a random PoW blockchain for testing:
```bash
python add_random_pow_blockchain.py
```

**Note**: This will fail if a blockchain database already exists. Delete files in the `database/` directory to regenerate.

## Known Issues and Troubleshooting

### Issue: `ModuleNotFoundError: No module named 'ecdsa'` (or other modules)

**Solution**: Make sure you've installed all dependencies:
```bash
pip install -r requirements.txt
```

If you're using a virtual environment, ensure it's activated before installing packages.

### Issue: `ModuleNotFoundError: No module named 'blockchain'`

**Solution**: Make sure you're running scripts from the `p1/` directory, not from a parent directory. The `blockchain` package must be in your Python path.

### Issue: `AttributeError: module 'blockchain' has no attribute 'chaindb'`

**Solution**: This has been fixed. The code now uses `blockchain.chain` instead of `blockchain.chaindb.chain`. If you see this error, make sure you have the latest version of `blockchain/block.py` and `blockchain/pow_block.py`.

### Issue: `TypeError: Block.__init__() got an unexpected keyword argument 'include_merkle_root'`

**Status**: **Known Issue** - The test `tests.hash.HashTest.test_blockchain_hash` passes `include_merkle_root=False` to the `Block.__init__()` method, but the current implementation may not fully handle this parameter correctly.

**Details**: 
- The `Block.__init__()` method accepts the `include_merkle_root` parameter (defaults to `True`)
- When `include_merkle_root=False`, the merkle root should not be calculated
- The test expects a specific hash when the merkle root is excluded
- This may cause the `test_blockchain_hash` test to fail with a hash mismatch

**Workaround**: The test may fail, but other functionality should work correctly. The `include_merkle_root` parameter is primarily used for testing block hashing without Merkle root computation.

### Issue: `error: externally-managed-environment` when installing packages

**Solution**: On macOS with Homebrew Python, you need to use a virtual environment:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Issue: Database Lock Errors

**Solution**: If you see database lock errors, make sure no other Python processes are accessing the database. Delete the lock file:
```bash
rm database/chain.db.lock
```

## Project Components

### Block Structure

Each block contains:
- `height`: Block height in the chain
- `timestamp`: Unix timestamp
- `target`: Difficulty target (for PoW) or 0 (for PoA)
- `parent_hash`: Hash of the parent block
- `is_genesis`: Boolean indicating if this is the genesis block
- `merkle`: Merkle root hash of transactions
- `seal_data`: Nonce (PoW) or signature (PoA)
- `transactions`: List of transactions in the block

### Transaction Model

Transactions use the UTXO model:
- **Inputs**: References to previous transaction outputs (`tx_hash:output_index`)
- **Outputs**: New transaction outputs with sender, receiver, and amount
- Each transaction must:
  - Have valid inputs (existing outputs)
  - Not create money (inputs ≥ outputs)
  - Have consistent users (all inputs from same user, all outputs from same user)
  - Not double-spend inputs

## Testing

The test suite covers:
1. **Hash Functions**: Double SHA256 hashing
2. **Block Weight**: PoW difficulty and weight calculations
3. **Chain Traversal**: Following parent pointers to build chain paths
4. **Block Validation**: All validation rules and edge cases
5. **PoA Mining**: Signature generation and verification
6. **Merkle Roots**: Merkle tree computation for transaction lists
