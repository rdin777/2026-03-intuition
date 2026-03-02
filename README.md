# Intuition audit details
- Total Prize Pool: $17,500 in USDC
    - HM awards: up to $14,400 in USDC
        - If no valid Highs or Mediums are found, the HM pool is $0
    - QA awards: $600 in USDC
    - Judge awards: $2,000 in USDC
    - Scout awards: $500 USDC
- [Read our guidelines for more details](https://docs.code4rena.com/competitions)
- Starts March 4, 2026 20:00 UTC
- Ends March 9, 2026 20:00 UTC

### ❗ Important notes for wardens
1. Since this audit includes live/deployed code, **all submissions will be treated as sensitive**:
    - Wardens are encouraged to submit High-risk submissions affecting live code promptly, to ensure timely disclosure of such vulnerabilities to the sponsor and guarantee payout in the case where a sponsor patches a live critical during the audit.
    - Submissions will be hidden from all wardens (SR and non-SR alike) by default, to ensure that no sensitive issues are erroneously shared.
    - If the submissions include findings affecting live code, there will be no post-judging QA phase. This ensures that awards can be distributed in a timely fashion, without compromising the security of the project. (Senior members of C4 staff will review the judges’ decisions per usual.)
    - By default, submissions will not be made public until the report is published.
    - Exception: if the sponsor indicates that no submissions affect live code, then we’ll make submissions visible to all authenticated wardens, and open PJQA to SR wardens per the usual C4 process.
    - [The "live criticals" exception](https://docs.code4rena.com/awarding#the-live-criticals-exception) therefore applies.
2. A coded, runnable PoC is required for all High/Medium submissions to this audit. 
    - This repo includes a basic template to run the test suite.
    - PoCs must use the test suite provided in this repo.
    - Your submission will be marked as Insufficient if the POC is not runnable and working with the provided test suite.
    - Exception: PoC is optional (though recommended) for wardens with signal ≥ 0.4.
3. Judging phase risk adjustments (upgrades/downgrades):
    - High- or Medium-risk submissions downgraded by the judge to Low-risk (QA) will be ineligible for awards.
    - Upgrading a Low-risk finding from a QA report to a Medium- or High-risk finding is not supported.
    - As such, wardens are encouraged to select the appropriate risk level carefully during the submission phase.

## V12 findings (🐺 C4 staff: remove this section for non-Solidity/EVM audits)

[V12](https://v12.zellic.io/) is [Zellic](https://zellic.io)'s in-house AI auditing tool. It is the only autonomous Solidity auditor that [reliably finds Highs and Criticals](https://www.zellic.io/blog/introducing-v12/). All issues found by V12 will be judged as out of scope and ineligible for awards.

V12 findings will typically be posted in this section within the first two days of the competition.  

## Publicly known issues

_Anything included in this section is considered a publicly known issue and is therefore ineligible for awards._

- Production config assumptions: `minDeposit = 1e16` and `minShare = 1e6`, and `MultiVault` enforces these floors. Findings that rely on `1 wei`style deposits or leaving vaults below minimum share floor should be treated as invalid under production params.
- `TrustBonding` intentionally has a single-epoch claim window; older rewards become unclaimable by users and are handled as unclaimed emissions.
- `AtomWallet` signature format is intentionally strict: only 65-byte raw ECDSA or 77-byte ECDSA+time-window suffix.
- Router behavior is intentional: ERC20 flow refunds excess ETH sent with the transaction (`msg.value - bridgeFee`), while the ETH flow consumes `msg.value - bridgeFee` as swap input (no separate excess-refund path).

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

# Overview

[ ⭐️ SPONSORS: add info here ]

## Links

- **Previous audits:**  https://github.com/0xIntuition/intuition-contracts-v2/tree/main/audits
  - ✅ SCOUTS: If there are multiple report links, please format them in a list.
- **Documentation:** https://www.docs.intuition.systems/
- **Website:** https://www.intuition.systems/
- **X/Twitter:** https://x.com/0xIntuition

---

# Scope

[ ✅ SCOUTS: add scoping and technical details here ]

### Files in scope
- ✅ This should be completed using the `metrics.md` file
- ✅ Last row of the table should be Total: SLOC
- ✅ SCOUTS: Have the sponsor review and and confirm in text the details in the section titled "Scoping Q amp; A"

*For sponsors that don't use the scoping tool: list all files in scope in the table below (along with hyperlinks) -- and feel free to add notes to emphasize areas of focus.*

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| [contracts/folder/sample.sol](https://github.com/code-423n4/repo-name/blob/contracts/folder/sample.sol) | 123 | This contract does XYZ | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

### Files out of scope
✅ SCOUTS: List files/directories out of scope

# Additional context

## Areas of concern (where to focus for bugs)
- `TrustSwapAndBridgeRouter`: packed path parsing/pool validation, bridge fee handling, and external call sequencing (swap router + bridge hub).
- `AtomWallet`: signature decoding/validation semantics (PR #135), owner/entrypoint authorization boundaries, and ownership claim transition logic.
- `TrustBonding`: epoch boundary accounting, utilization-ratio math, and unclaimed-emissions accounting semantics (PR [#134](https://github.com/0xIntuition/intuition-contracts-v2/pull/134)).
- `ProgressiveCurve` + `OffsetProgressiveCurve`: redeem/withdraw rounding correctness and edge-case behavior at low-share states (PR #136).

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Main invariants

- Router path invariants: ETH path must start with WETH and all paths must end with TRUST; all hops must map to existing pools before swap.
- Router fee/value invariants: swap/bridge reverts if bridge fee is insufficient; ERC20 flow refunds all ETH above required bridge fee.

- AtomWallet auth invariant: only `owner()` or `EntryPoint` can execute wallet actions; deposit withdrawal is only by owner or self-call.
- `AtomWallet` signature invariant: validation only accepts 65/77-byte signatures; time-window metadata is interpreted from signature suffix (PR [#135](https://github.com/0xIntuition/intuition-contracts-v2/pull/135) behavior).
- `TrustBonding` claim-window invariant: only previous epoch is user-claimable; cannot double-claim an epoch.
- `TrustBonding` unclaimed invariant: for epochs outside claim window, unclaimed amount is based on max epoch emissions minus claimed (PR [#134](https://github.com/0xIntuition/intuition-contracts-v2/pull/134) behavior).
- Curve math invariant: redeem/convert-to-assets path uses consistent conservative rounding and must not revert in low-share edge cases where result should be zero (PR #136 behavior).
- System config invariant: effective deployment assumptions include `minDeposit`/`minShare` floors mentioned above (no dust-state vault behavior below floor).

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## All trusted roles in the protocol

- `TrustSwapAndBridgeRouter`: contract `owner` (`Ownable2Step`) can set router/factory/quoter/bridge config
- `TrustBonding`: `timelock` (parameter updates; min delay is 3 days), `DEFAULT_ADMIN_ROLE` (admin/unpause + inherited `VotingEscrow` admin actions), `PAUSER_ROLE` (can only pause the contract).
- `AtomWallet`: wallet `owner` (initially `AtomWarden` via `MultiVault` until claimed, then user), and trusted `EntryPoint` contract for AA (account abstraction) execution path.
- Curves (`ProgressiveCurve`, `OffsetProgressiveCurve`): no runtime privileged role in normal operation; only upgrade/initialization authority at proxy/governance layer, which is handled via timelock for all of our contracts (with a min delay of 7 days).

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Owner                          | Has superpowers                |
| Administrator                             | Can change fees                       |

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Running tests

- If Foundry is not already installed: `curl -L https://foundry.paradigm.xyz | bash && foundryup`
- `git clone <repo> && cd <repo>`
- Install dependencies: `forge install && bun install`
- Build contracts: `forge build`
- Run the tests: `forge test`
    - Optional: Fork tests require Alchemy RPC config, so please make sure to set `API_KEY_ALCHEMY` in your local `.env` file (or skip fork tests when running locally without RPC).

✅ SCOUTS: Please format the response above 👆 using the template below👇

```bash
git clone https://github.com/code-423n4/2023-08-arbitrum
git submodule update --init --recursive
cd governance
foundryup
make install
make build
make sc-election-test
```
To run code coverage
```bash
make coverage
```

✅ SCOUTS: Add a screenshot of your terminal showing the test coverage

## Miscellaneous
Employees of Intuition and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.





# Scope

*See [scope.txt](https://github.com/code-423n4/2026-03-intuition/blob/main/scope.txt)*

### Files in scope


| File   | Logic Contracts | Interfaces | nSLOC | Purpose | Libraries used |
| ------ | --------------- | ---------- | ----- | -----   | ------------ |
| /src/protocol/emissions/TrustBonding.sol | 1| **** | 359 | |@openzeppelin/contracts-upgradeable/utils/PausableUpgradeable.sol<br>src/interfaces/ICoreEmissionsController.sol<br>src/interfaces/IMultiVault.sol<br>src/interfaces/ITrustBonding.sol<br>src/interfaces/ISatelliteEmissionsController.sol<br>src/external/curve/VotingEscrow.sol|
| /src/protocol/curves/ProgressiveCurve.sol | 1| **** | 83 | |@prb/math/src/UD60x18.sol<br>src/protocol/curves/BaseCurve.sol<br>src/libraries/ProgressiveCurveMathLib.sol|
| /src/protocol/curves/OffsetProgressiveCurve.sol | 1| **** | 86 | |@prb/math/src/UD60x18.sol<br>src/protocol/curves/BaseCurve.sol<br>src/libraries/ProgressiveCurveMathLib.sol|
| /src/protocol/wallet/AtomWallet.sol | 1| **** | 158 | |@account-abstraction/core/BaseAccount.sol<br>@account-abstraction/interfaces/PackedUserOperation.sol<br>@openzeppelin/contracts/utils/cryptography/ECDSA.sol<br>@account-abstraction/interfaces/IEntryPoint.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/access/Ownable2StepUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol<br>src/interfaces/IMultiVault.sol<br>@account-abstraction/core/Helpers.sol|
| **Totals** | **4** | **** | **686** | | |

### Files out of scope

*See [out_of_scope.txt](https://github.com/code-423n4/2026-03-intuition/blob/main/out_of_scope.txt)*

| File         |
| ------------ |
| ./script/SetupScript.s.sol |
| ./script/base/BaseEmissionsControllerDeploy.s.sol |
| ./script/base/BaseEmissionsControllerSetup.s.sol |
| ./script/base/LegacyTrustTokenDeploy.s.sol |
| ./script/base/TrustDeploy.s.sol |
| ./script/e2e/HubBridgeDeploy.s.sol |
| ./script/e2e/SpokeBridgeDeploy.s.sol |
| ./script/intuition/DeployOffsetProgressiveCurve.s.sol |
| ./script/intuition/IntuitionDeployAndSetup.s.sol |
| ./script/intuition/MultiVaultDeploy.s.sol |
| ./script/intuition/MultiVaultMigrationModeDeploy.s.sol |
| ./script/intuition/MultiVaultMigrationUpgrade.s.sol |
| ./script/intuition/TrustBondingDeployAndSetup.s.sol |
| ./script/intuition/WrappedTrustDeploy.s.sol |
| ./script/periphery/DeployEntryPoint.s.sol |
| ./src/Trust.sol |
| ./src/WrappedTrust.sol |
| ./src/external/curve/VotingEscrow.sol |
| ./src/interfaces/IAtomWallet.sol |
| ./src/interfaces/IAtomWalletFactory.sol |
| ./src/interfaces/IAtomWarden.sol |
| ./src/interfaces/IBaseCurve.sol |
| ./src/interfaces/IBaseEmissionsController.sol |
| ./src/interfaces/IBondingCurveRegistry.sol |
| ./src/interfaces/ICoreEmissionsController.sol |
| ./src/interfaces/IMetaLayer.sol |
| ./src/interfaces/IMultiVault.sol |
| ./src/interfaces/IMultiVaultCore.sol |
| ./src/interfaces/ISatelliteEmissionsController.sol |
| ./src/interfaces/ITrust.sol |
| ./src/interfaces/ITrustBonding.sol |
| ./src/interfaces/ITrustUnlock.sol |
| ./src/interfaces/ITrustUnlockFactory.sol |
| ./src/legacy/TrustToken.sol |
| ./src/libraries/ProgressiveCurveMathLib.sol |
| ./src/protocol/MultiVault.sol |
| ./src/protocol/MultiVaultCore.sol |
| ./src/protocol/MultiVaultMigrationMode.sol |
| ./src/protocol/curves/BaseCurve.sol |
| ./src/protocol/curves/BondingCurveRegistry.sol |
| ./src/protocol/curves/LinearCurve.sol |
| ./src/protocol/emissions/BaseEmissionsController.sol |
| ./src/protocol/emissions/CoreEmissionsController.sol |
| ./src/protocol/emissions/MetaERC20Dispatcher.sol |
| ./src/protocol/emissions/SatelliteEmissionsController.sol |
| ./src/protocol/wallet/AtomWalletFactory.sol |
| ./src/protocol/wallet/AtomWarden.sol |
| ./tests/BaseTest.t.sol |
| ./tests/mocks/BaseEmissionsControllerMock.sol |
| ./tests/mocks/CoreEmissionsControllerMock.sol |
| ./tests/mocks/ERC20Mock.sol |
| ./tests/mocks/MetalayerRouterMock.sol |
| ./tests/mocks/TestTrust.sol |
| ./tests/mocks/TrustBondingMock.sol |
| ./tests/mocks/VotingEscrowHarness.sol |
| ./tests/testnet/HubBridge.sol |
| ./tests/testnet/SpokeBridge.sol |
| ./tests/unit/AtomWallet/AtomWallet.t.sol |
| ./tests/unit/AtomWarden/AtomWarden.t.sol |
| ./tests/unit/BaseEmissionsController/AccessControl.t.sol |
| ./tests/unit/BaseEmissionsController/MintAndBridge.t.sol |
| ./tests/unit/BaseEmissionsController/MintAndBridgeCurrentEpoch.t.sol |
| ./tests/unit/BaseEmissionsController/Reads.t.sol |
| ./tests/unit/CoreEmissionsController/CoreEmissionsControllerBase.t.sol |
| ./tests/unit/CoreEmissionsController/Reads.t.sol |
| ./tests/unit/MultiVault/AdminFunctions.t.sol |
| ./tests/unit/MultiVault/ClaimFees.t.sol |
| ./tests/unit/MultiVault/CounterStakeGuard.t.sol |
| ./tests/unit/MultiVault/CreateAtoms.t.sol |
| ./tests/unit/MultiVault/CreateTriples.t.sol |
| ./tests/unit/MultiVault/Deposit.t.sol |
| ./tests/unit/MultiVault/DepositBatch.t.sol |
| ./tests/unit/MultiVault/FeeFlows.t.sol |
| ./tests/unit/MultiVault/Helpers.t.sol |
| ./tests/unit/MultiVault/Redeem.t.sol |
| ./tests/unit/MultiVault/RedeemBatch.t.sol |
| ./tests/unit/MultiVault/UtilizationTest.t.sol |
| ./tests/unit/MultiVaultCore/MultiVaultCore.t.sol |
| ./tests/unit/MultiVaultMigrationMode/MultiVaultMigrationMode.t.sol |
| ./tests/unit/SatelliteEmissionsController/AccessControl.t.sol |
| ./tests/unit/SatelliteEmissionsController/BridgeUnclaimedEmissions.t.sol |
| ./tests/unit/SatelliteEmissionsController/Reads.t.sol |
| ./tests/unit/SatelliteEmissionsController/WithdrawUnclaimedEmissions.t.sol |
| ./tests/unit/Trust/LegacyTrustToken.t.sol |
| ./tests/unit/Trust/Trust.t.sol |
| ./tests/unit/Trust/TrustUpgradeIntegrationTest.t.sol |
| ./tests/unit/Trust/WrappedTrust.t.sol |
| ./tests/unit/TrustBonding/AccessControl.t.sol |
| ./tests/unit/TrustBonding/ClaimRewards.t.sol |
| ./tests/unit/TrustBonding/Events.t.sol |
| ./tests/unit/TrustBonding/Getters.t.sol |
| ./tests/unit/TrustBonding/NormalizedUtilizationRatio.t.sol |
| ./tests/unit/TrustBonding/Reads.t.sol |
| ./tests/unit/TrustBonding/TrustBonding.t.sol |
| ./tests/unit/TrustBonding/TrustBondingBase.t.sol |
| ./tests/unit/TrustBonding/TrustBondingIntegration.t.sol |
| ./tests/unit/TrustBonding/TrustBondingRegressionTest.t.sol |
| ./tests/unit/TrustBonding/UserAndSystemUtilizationRatio.t.sol |
| ./tests/unit/TrustBonding/reads/BalanceOf.t.sol |
| ./tests/unit/TrustBonding/reads/GetUserCurrentClaimableRewards.t.sol |
| ./tests/unit/TrustBonding/reads/GetUserInfo.t.sol |
| ./tests/unit/TrustBonding/reads/GetUserRewardsForEpoch.t.sol |
| ./tests/unit/TrustBonding/reads/SystemApy.t.sol |
| ./tests/unit/TrustBonding/reads/UserApy.t.sol |
| ./tests/unit/TrustBonding/reads/UserClaimedRewardsForEpoch.t.sol |
| ./tests/unit/VotingEscrow/VotingEscrow.t.sol |
| ./tests/unit/VotingEscrow/VotingEscrowBinarySearch.t.sol |
| ./tests/unit/VotingEscrow/VotingEscrowVievHelpersIntegration.t.sol |
| ./tests/unit/curves/BondingCurveRegistry.t.sol |
| ./tests/unit/curves/LinearCurve.t.sol |
| ./tests/unit/curves/OffsetProgressiveCurve.t.sol |
| ./tests/unit/curves/OffsetProgressiveCurveConfiguration.t.sol |
| ./tests/unit/curves/ProgressiveCurve.t.sol |
| ./tests/utils/Constants.sol |
| ./tests/utils/Modifiers.sol |
| ./tests/utils/Types.sol |
| ./tests/utils/Utils.sol |
| Totals: 117 |

