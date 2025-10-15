deployed contract address: 0xBaAEF1E47DEdc209E0B80D6606624e97146d1d6B 

Build a dynamic whitelist with staking requirements, write this smart contract in Solidity, no imports and constructors, with no input fields
# WhitelistStaking

A single-file Solidity smart contract that provides a **dynamic whitelist** based on native ETH staking. No imports and no constructor parameters — fully self-contained.

## Key ideas

* Users stake native ETH (via `stake()` or by sending ETH directly) to accumulate balance in the contract.
* If a user's stake is **≥ `stakeRequirement`**, they become automatically whitelisted.
* If they withdraw and their stake falls **below** the requirement, they are removed from the whitelist.
* The owner (deployer) can change the stake requirement and perform batch updates or manual whitelist overrides.
* Reentrancy guard included (simple, internal).

## Files

* `WhitelistStaking.sol` — the Solidity contract (single file; no imports; no constructor).

## Deployment

* Solidity compiler version: `^0.8.20`.
* There is **no constructor**. Owner is set to the deployer (the address that sends the transaction creating the contract).
* Deploy normally using your preferred tooling (Hardhat, Remix, Foundry, etc).

## Important public functions

* `stake()` — payable. Stake ETH (adds `msg.value` to caller's stake; auto-whitelists if stake ≥ requirement).
* `withdraw(uint256 amount)` — withdraw part/all of your stake (ETH returned to caller). If resulting stake < requirement, auto-removes from whitelist.
* `ownerCreditStake(address user)` — payable owner function to credit a user's stake from owner funds.
* `isWhitelisted(address account)` — view: returns boolean.
* `getStake(address account)` — view: returns stake in wei.
* `setStakeRequirement(uint256 newRequirement)` — owner-only: change required stake (in wei).
* `batchUpdateWhitelist(address[] calldata addrs)` — owner-only: evaluate a list of addresses and add/remove them based on current stakes.
* `ownerSetWhitelist(address account, bool status)` — owner-only: manually override whitelist status.
* `transferOwnership(address newOwner)` — owner-only.
* `emergencyWithdraw(address payable to, uint256 amount)` — owner-only emergency ETH withdraw from contract balance (use with caution).

Also supports `receive()` and `fallback()` to treat direct ETH transfers as stakes.

## Events

* `StakeDeposited(address user, uint256 amount)`
* `StakeWithdrawn(address user, uint256 amount)`
* `Whitelisted(address user)`
* `Dewhitelisted(address user)`
* `StakeRequirementChanged(uint256 oldRequirement, uint256 newRequirement)`
* `OwnerTransferred(address oldOwner, address newOwner)`
* `EmergencyWithdraw(address to, uint256 amount)`

## Example usage

1. Deploy contract (deployer becomes owner).
2. Users:

   * Call `stake()` with `msg.value` (e.g. 1 ether) → if stake ≥ requirement they'll be whitelisted.
   * Or send ETH directly to contract address → same effect (receive/fallback treat it as stake).
3. Users can call `withdraw(amount)` to take out ETH; if lowered below requirement they will be dewhitelisted automatically.
4. Owner can change `stakeRequirement` (e.g. to increase minimum), and then call `batchUpdateWhitelist` with a list of addresses to immediately enforce the new rule.

## Security notes & recommendations

* **Native ETH only**: This contract uses native ETH for staking. If you prefer ERC-20 token staking, the contract must be expanded (adds ERC-20 transfer/approve handling) — that would require adding interface code (still doable without imports).
* **EmergencyWithdraw**: Owner can withdraw ETH from the contract with `emergencyWithdraw`. This withdraws from contract balance and may include staked funds (use with extreme caution). You could disable this in a production release or add clearer separation between user stakes and operational funds.
* **No builtin rewards**: This contract does not pay interest or rewards—just tracks stakes.
* **Batch operations cost gas**: `batchUpdateWhitelist` iterates; keep arrays modest to avoid hitting block gas limits.
* **Reentrancy guard**: Implemented for deposit/withdrawal flows.
* **Owner powers**: Owner can manually set/clear whitelist entries. Consider multisig ownership or governance for production.
* **Testing**: Thoroughly test staking/withdraw and edge cases (e.g. partial withdraws that exactly meet requirement, direct sends, fallback behaviour).

## Possible improvements

* Add explicit separation of user-staked funds vs. protocol funds (escrow pattern).
* Add ERC-20 staking option.
* Add on-chain snapshotting / event logs that index whitelisted users.
* Add a timelock or multisig for owner actions (transferOwnership, setStakeRequirement, emergencyWithdraw).
* Add feature to lock stake for a duration (to prevent flash stake/un-stake to manipulate access).

## Example values

* Default `stakeRequirement` = **1 ether** at deployment. Owner can change it.
