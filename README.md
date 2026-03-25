# 996 Coin

996 Coin is a cryptocurrency developed by the HCC Community (see hashcashfaucet.com)
This repository contains the core node, CLI tools, wallet tooling, and Qt desktop wallet sources.

## 996-Coin Parameters

### General
- **Coin name:** 996-Coin
- **Ticker / unit shown in wallet:** NNS / 996-Coin network
- **Bech32 HRP:** `996`

### Network
- **Mainnet P2P port:** `49969`
- **Mainnet RPC port:** `48931`
- **Message start:** `99 6c 01 33`

### Address prefixes (mainnet)
- **P2PKH / public key address prefix:** `53`
- **P2SH / script address prefix:** `18`
- **Private key prefix:** `128`

This means normal mainnet addresses currently begin with **N**.

### Genesis
- **Genesis reward:** `50 COIN`
- **Genesis timestamp message:** `Why are we so rich? Because 996!`
- **Genesis hash:** `48b00c93e8a20fb6b39f7d99b85066da2820cc3830ca223fc3e04d1a5b0dcbb7`


## Consensus

### Hybrid launch / transition
- The chain is designed to bootstrap with PoW and then continue with PoS.
- **Last PoW block before PoS:** `500`

### Block reward schedule
- **Reward reduction interval:** `996,000 blocks` (about 5.7 years)
- At each reduction point, the block subsidy is reduced to **1/10** of the previous value.
- 500 to 996,000 blocks: 996 NNS 
- 996,001 - 1,992,000 blocks 99.6 NNS
- 1,992,000 - 2.988.000 blocks 9.96 NNS
- ...

### Target timing
- **Target block spacing:** `3 minutes`
- **Target timespan:** `10 blocks = 30 minutes`

## Proof of Stake

### Stake maturity
- Staking uses the chain’s `COINBASE_MATURITY` logic.
- In practice, stakeable outputs must:
  - be confirmed,
  - not be spent,
  - not be locked,
  - have reached maturity,
  - and belong to supported output types.

### Supported staking output types
The wallet currently considers these output types for staking:
- `PUBKEY`
- `PUBKEYHASH`
- optionally `COLDSTAKE` if cold staking is enabled

### Excluded from direct staking
These outputs are **not** used as direct staking inputs:
- `ISMINE_SPENDABLE_DELEGATED`

### Stake split / combine behavior
From `pos.cpp`:
- **Max combine inputs:** `40`
- **Combine threshold:** `50 COIN`
- **Stake split outputs:** `2`
- **Split threshold:** `500 COIN`

This means:
- small same-script inputs may be combined into one coinstake,
- larger stake rewards can be split into 2 outputs,
- staking UTXOs above the split threshold may be split automatically.

### Effective per-UTXO staking cap
Consensus-side kernel weight is capped per UTXO:
- **Max effective stake weight per UTXO:** `125,000 COIN`

This means a UTXO larger than `125,000` coins does **not** gain additional staking probability above that cap.

### Practical consequence
- Splitting funds into multiple UTXOs of around **125k** can be advantageous for staking frequency.
- A single 400k UTXO does **not** stake like 400k effective weight; it is capped to 125k in the kernel.

## MPoS / Multi PoS Rewards

### MPoS parameters
- **MPoS reward recipients:** `10`
- **First MPoS block:**  
  `nLastPOWBlock + nMPoSRewardRecipients + COINBASE_MATURITY`
- **Mainnet MPoS end:** currently  
  `nLastMPoSBlock = consensus.NNSColdStakeEnableHeight`
- **Cold staking activation height:** `260,000`

### Current reward logic
During the MPoS window:
- block reward is split into **10 shares**
- the staker keeps:
  - **1 share + remainder**
- the other **9 shares** are distributed to previous PoS stakers via MPoS outputs

Outside the MPoS window:
- the staking node keeps the normal full PoS reward
- unless cold staking applies

  
## Cold Staking

### Cold staking activation
- **Mainnet enable height:** `260,000`

### Behavior
Cold staking is supported in the codebase and can coexist with normal staking.
When cold staking is active:
- the owner and cold staker can split rewards,
- `COLDSTAKE` outputs can be recognized by wallet and consensus code,
- wallet inclusion depends on the `-coldstaking` setting.



## Wallet staking behavior

### Wallet stake selection
The wallet currently:
- scans all mature, unspent, unlocked eligible staking outputs
- excludes delegated outputs from direct staking
- optionally includes cold staking outputs
- no longer applies an artificial wallet-side truncation of staking coins before selection
- consensus kernel weight is capped per UTXO at **125,000 COIN**


### PoS probability
PoS selection is proportional to stake weight, but with:
- **per-UTXO effective cap = 125,000 COIN**

This is the most important staking-specific economic parameter.

### Block production safety
If very few or no mature coins exist after the PoW→PoS switch, the chain can temporarily stall until some mature eligible staking outputs are available.


## Recommended operator guidance

- Keep at least one node with mature staking outputs online after PoW ends.
- Do not consolidate everything into one giant UTXO.
- For best staking efficiency, split large balances into chunks near **125,000 coins**.
- Ensure the wallet is unlocked and staking-enabled.
- Check `getstakingstatus` and `listunspent` if staking does not start.

## Build Targets

The repository currently contains source/build logic for:

- daemon
- CLI RPC client
- transaction utility
- wallet utility
- Qt GUI wallet

Depending on platform and build method, output binaries may still use inherited upstream naming until renamed in code and packaging.

## Building

Build instructions are not yet fully refreshed for 996-Coin.

Until project-specific instructions are finalized, review the existing platform build files and upstream-style documentation in:

- `doc/`
- `depends/`
- `build_msvc/`
- `contrib/`

If you are contributing build instructions, prefer documenting:

- tested OS version
- compiler/toolchain version
- dependency installation steps
- exact build commands
- resulting binary names
- known warnings or failures

## Configuration

Default configuration naming and datadir conventions may still reflect upstream ancestry in parts of the codebase. These should be verified before release packaging or public distribution.

## Contributing

Contributions should be made through branches and pull requests unless direct branch coordination has been explicitly agreed by the project maintainers.

Recommended contribution flow:

1. create a feature or audit branch
2. make scoped changes
3. test locally where possible
4. open a pull request with a clear summary
5. include notes on build/test status

## Project Status

This project is in active fork/customization state. Some codebase and build-system elements still reflect upstream Bitcoin PoS ancestry while chain parameters and network identity have been customized for 996-Coin.

That means this repository should currently be treated as a development fork, not as a polished end-user release.

## Important Note

This project is derived from upstream Bitcoin Core / Bitcoin PoS lineage. As with many cryptocurrency forks, inherited naming, comments, scripts, and documentation may still exist in parts of the codebase. Ongoing cleanup and verification are expected parts of development.

Do not assume all documentation, filenames, or build artifacts are fully rebranded until they have been reviewed.


## Security

This repository may contain inherited security contact or upstream project references that are no longer valid for 996-Coin. Verify all security/reporting contact paths before publishing releases.

## License

This project inherits from Bitcoin Core lineage and is distributed under the MIT software license unless otherwise noted in specific files.

See `COPYING` for details.
