# Leverj Gluon Plasma Dapp Audit Report

## Preamble
This audit report was undertaken by **BlockchainLabs.nz** for the purpose of providing feedback to **Leverj**.

It has subsequently been shared publicly without any express or implied warranty.

Solidity contracts were provided by the Leverj development team at this commit [leverj/gluon-plasma @ a44c9cd476539ed9d68df369cfef5b410ed1afa3](https://github.com/leverj/gluon-plasma/tree/a44c9cd476539ed9d68df369cfef5b410ed1afa3) and the most recent commit we have audited is this [41a78e6ea77c17cf06e9180e52e1848dfc168c7e](https://github.com/leverj/gluon-plasma/tree/41a78e6ea77c17cf06e9180e52e1848dfc168c7e) - we would encourage all community members and token holders to make their own assessment of the contracts.

## Scope
All smart contracts, test files, migration and deployment scripts from [this Github repo](https://github.com/leverj/gluon-plasma).
Configuration files, documents and other assets were **out of the scope** of this audit. 

- [Custodian.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/custodian/Custodian.sol)
- [Depositing.sol(interface)](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/custodian/Depositing.sol)
- [Ledger.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/custodian/Ledger.sol)
- [Withdrawing.sol(interface)](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/custodian/Withdrawing.sol)
- [ApiKeyRegistry.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/registry/ApiKeyRegistry.sol)
- [Registry.sol(interface)](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/registry/Registry.sol)
- [Fee.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/staking/Fee.sol)
- [Stake.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/staking/Stake.sol)
- [HasOwners.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/HasOwners.sol)
- [Stoppable.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/Stoppable.sol)
- [Switchable.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/Switchable.sol)
- [Validating.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/Validating.sol)
- [Versioned.sol](https://github.com/BlockchainLabsNZ/Leverj-gluon-plasma-audit/blob/master/packages/on-chain/contracts/Versioned.sol)

## Focus areas
The audit report is focused on the following key areas - though this is not an exhaustive list.

### Correctness
- No correctness defects uncovered during static analysis?
- No implemented contract violations uncovered during execution?
- No other generic incorrect behaviour detected during execution?
- Adherence to adopted standards such as ERC20?

### Testability
- Test coverage across all functions and events?
- Test cases for both expected behaviour and failure modes?
- Settings for easy testing of a range of parameters?
- No reliance on nested callback functions or console logs?
- Avoidance of test scenarios calling other test scenarios?

### Security
- No presence of known security weaknesses?
- No funds at risk of malicious attempts to withdraw/transfer?
- No funds at risk of control fraud?
- Prevention of Integer Overflow or Underflow?

### Best Practice
- Explicit labeling for the visibility of functions and state variables?
- Proper management of gas limits and nested execution?
- Latest version of the Solidity compiler?

## Issues

### Severity Description
<table>
<tr>
  <td>Minor</td>
  <td>A defect that does not have a material impact on the contract execution and is likely to be subjective.</td>
</tr>
<tr>
  <td>Moderate</td>
  <td>A defect that could impact the desired outcome of the contract execution in a specific scenario.</td>
</tr>
<tr>
  <td>Major</td>
  <td> A defect that impacts the desired outcome of the contract execution or introduces a weakness that may be exploited.</td>
</tr>
<tr>
  <td>Critical</td>
  <td>A defect that presents a significant security vulnerability or failure of the contract across a range of scenarios.</td>
</tr>
</table>

### Minor

- **canExit() can be used in exit()** -`Best practice`<br>
```javascript
235  function canExit(bytes32 entryHash) public view returns (bool) {
236    return
237      exitClaims[entryHash].confirmationThreshold != 0 &&  // exists
238      currentGblockNumber >= exitClaims[entryHash].confirmationThreshold;
239  }
```
`canExit()` at line 235 of `Custodian.sol` verifies if the user can exit. It is a public function and it can be used in `exit()` to replace line 207-208.
- [x] Fixed at [41a78e6e](https://github.com/leverj/gluon-plasma/blob/41a78e6ea77c17cf06e9180e52e1848dfc168c7e/packages/on-chain/contracts/custodian/Custodian.sol#L231)
<br>

- **canSubmit() is always true** - `Correctness`<br>
```javascript
282  function canSubmit() public view returns (bool) { return block.number >= submissionBlock; }
``` 
The `canSubmit()` always returns `true` because the variable `submissionBlock` is set to equal to `block.number` at contract deployment. From here forwards, `block.number` will always be equal to or greater than `submissionBlock`, which makes the returned value `true` for all time.<br>
- [x] Fixed at [41a78e6e](https://github.com/leverj/gluon-plasma/blob/41a78e6ea77c17cf06e9180e52e1848dfc168c7e/packages/on-chain/contracts/custodian/Custodian.sol#L294)<br>
<br>

- **Gas optimisation of Deposit Commitment Record** - `Enhancement`<br>
```javascript
86  function deposit(address account, address asset, uint quantity) private whenOn {
87    uint nonce = ++nonceGenerator;
88    uint designatedGblock = currentGblockNumber + visibilityDelay;
89    DepositCommitmentRecord memory record = toDepositCommitmentRecord(account,                           asset, quantity, nonce, designatedGblock);
90    deposits[record.hash] = true;
91    emit Deposited(address(this), account, asset, quantity, nonce, designatedGblock);
92  }
```
`DepositCommitmentRecord` is generated here, but only `record.hash` is being used. To save some gas (for storing variable in memory), you could replace with below:
`hash = keccak256(abi.encodePacked(address(this), account, asset, quantity, nonce, designatedGblock));`<br>
- [ ] Fix not required - works as designed <br>
<br>

- **submissionInterval is never used** - `Correctness`<br>
`submissionInterval` is set at constructor but never used.<br>
- [x] Fixed at [41a78e6e](https://github.com/leverj/gluon-plasma/blob/41a78e6ea77c17cf06e9180e52e1848dfc168c7e/packages/on-chain/contracts/custodian/Custodian.sol#L294)<br>
<br>

- **double ternary operators are used** - `Best practice` <br>
```javascript
111  uint withdrawLev = signedQuantity >= 0 ?
112        0 :
113        uint(signedQuantity * -1) >= stake.quantity ?
114          stake.quantity :
115          uint(signedQuantity * -1);
```
Ternary operators are use twice here, which makes the readability and maintainability harder.
There are many better approaches to improve it. 
- [x] Fixed at [41a78e6e](https://github.com/leverj/gluon-plasma/blob/41a78e6ea77c17cf06e9180e52e1848dfc168c7e/packages/on-chain/contracts/staking/Stake.sol#L110)<br>
<br>

- **Any one can withdraw() other people's asset** - `Enhancement` <br>
```javascript
182  function withdraw(address[] addresses, uint[] uints, bytes signature, bytes proof, bytes32 root) external {
183    Entry memory entry = extractEntry(addresses, uints);
184    verifySignedBy(entry.hash, signature, operator);
185    require(entry.entryType == EntryType.Withdrawal, "entry must be of type Withdrawal");
186    require(proveInConfirmedWithdrawals(proof, root, entry.hash), "invalid entry proof");
187    require(!withdrawn[entry.hash], "entry already withdrawn");
188    withdrawn[entry.hash] = true;
189    transfer(entry.quantity, entry.asset, entry.account);
190    emit Withdrawn(entry.hash, entry.account, entry.asset, entry.quantity);
191  }
```
e.g. There are two users - A and B. User A deposits some asset in the ledger. Once user B get the proof/signature from the operator, he can call `withdraw()` function to transfer the asset back to user A.
In this case, user A doesn't actually lose anything, but it is kind of unexpected that the balance is not in his ledger account.<br>
- [x] Fixed at [41a78e6e](https://github.com/leverj/gluon-plasma/blob/41a78e6ea77c17cf06e9180e52e1848dfc168c7e/packages/on-chain/contracts/custodian/Custodian.sol#L202)<br>
<br>

- **Variables could benefit from better naming and docstring** - `Best practice` <br>
Some params are sharing the name but not intrinsic meaning.  
e.g. `root` in `claimExit()` means balances root, but in `withdraw()` it means withdrawal root.
Recommend to have a better naming, or in order to keep variable's name short, add docstring in front of functions to clarify those variables(like what them mean and where are they from if necessary). That's a good practice for both community and your future maintainability.<br>
- [ ] Fix not required - Future enhancement

### Moderate

- None found

### Major

- **Don't trust outside contract** - `Security` <br>
```javascript
81  function depositToken(address token, uint quantity) external validToken(token) {
82    require(Token(token).transferFrom(msg.sender, this, quantity), "failure to transfer quantity from token");
83    deposit(msg.sender, token, quantity);
84  }
```
When a contract is calling functions in an outside contract, we need to verify if the action is really being done instead of trust the return value because the return value can be fake.
In a ERC20 complaint contract there is a `transferFrom()`. We are expecting it increase Custodian's token balance while decrease executor's balance, and then return `true` if transfer is success. However, if a DummyToken contract designed on purpose with the `transferFrom()` like below, Custodian cannot get transferred tokens in DummyToken but still increase executor’s token amount generating from nowhere in Custodian. <br>
- [x] Fixed at [41a78e6e](https://github.com/leverj/gluon-plasma/blob/41a78e6ea77c17cf06e9180e52e1848dfc168c7e/packages/on-chain/contracts/custodian/Custodian.sol#L81-L87)<br>

### Critical

- None found

## Observations

It is documented that `if the root hashes do not match the values committed to the plasma contract, there is a potential data unavailability issue and the participant can vote to halt`. Currently there are no specified functions implementing the logic in the contracts. When the exception happens, there is one method less to protect the users by themselves. Considering the DApp is still under development, we are expecting more related code to be added in future releases to fulfill the objective.<br>

When declaring the inheritance, it is very important to be aware that `HasOwners` contract needs to be always after `Stoppable` contract. Otherwise, the logic of `onlyOwner` modifier will be overwritten. At the moment of this audit, the code has been tested for success and failures of the `onlyOwner` modifier. In case future maintainers rewrite the code, they should ensure the tests for correct inheritance of `onlyOwner` are passing. <br>

`MerkleProof.sol` is an imported external contract based on [this repo](https://github.com/ameensol/merkle-tree-solidity/blob/master/src/MerkleProof.sol). Leverj made some tweaks and added functions in the contract for convenience. However, these are only minor and do not affect the correctness of the merkle root functions.<br>

## Conclusion

We are satisfied that these Smart Contracts do not exhibit serious security vulnerabilities. Overall the code is well written and the developers have been responsive and active throughout the audit process. The contracts show care taken by the developers to follow best practices and a strong knowledge of Solidity. There is very high test coverage which should increase confidence in the security of these contracts, and their maintainability/development in the future.
___

### Disclaimer

Our team uses our current understanding of the best practises for Solidity and Smart Contracts. Development in Solidity and for Blockchain is an emergering area of software engineering which still has a lot of room to grow, hence our current understanding of best practise may not find all of the issues in this code and design.

We have not analysed any of the assembly code generated by the Solidity compiler. We have not verified the deployment process and configurations of the contracts. We have only analysed the code outlined in the scope. We have not verified any of the claims made by any of the organisations behind this code.

Security audits do not warrant bug-free code. We encourge all users interacting with smart contract code to continue to analyse and inform themselves of any risks before interacting with any smart contracts.
