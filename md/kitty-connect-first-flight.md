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
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings](#findings)
- [High](#high)
  - [H-01. `KittyConnect::_updateOwnershipInfo` function doesn't update the ownership info of the kitty's previous owner, leads to confusion in managing and querying the ownership of NFTs.](#h-01-kittyconnect_updateownershipinfo-function-doesnt-update-the-ownership-info-of-the-kittys-previous-owner-leads-to-confusion-in-managing-and-querying-the-ownership-of-nfts)
    - [Relevant GitHub Links](#relevant-github-links)
  - [Vulnerability Details](#vulnerability-details)
  - [Impact](#impact)
  - [Tools Used](#tools-used)
  - [Proof of Code](#proof-of-code)
  - [Recommendations](#recommendations)
- [Gas](#gas)
  - [G-01. Using `require` instead of custom errors, leads to gas inefficiency.](#g-01-using-require-instead-of-custom-errors-leads-to-gas-inefficiency)
    - [Relevant GitHub Links](#relevant-github-links-1)
  - [Description](#description)
  - [Impact](#impact-1)
  - [Tools Used](#tools-used-1)
  - [Recommendations](#recommendations-1)
  - [G-02. Reading from storage in `KittyConnect::constructor` every iteration in the for-loop, which is gas inefficient.](#g-02-reading-from-storage-in-kittyconnectconstructor-every-iteration-in-the-for-loop-which-is-gas-inefficient)
    - [Relevant GitHub Links](#relevant-github-links-2)
  - [Vulnerability Details](#vulnerability-details-1)
  - [Impact](#impact-2)
  - [Tools Used](#tools-used-2)
  - [Recommendations](#recommendations-2)

# Protocol Summary

Kitty Connect allows users to buy a cute cat from our branches and mint NFT for buying a cat. The NFT will be used to track the cat info and all related data for a particular cat corresponding to their token ids. Kitty Owner can also Bridge their NFT from one chain to another chain via Chainlink CCIP.

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

# Audit Details
The findings described in this document correspond to the following github repo:

```
https://github.com/Cyfrin/2024-03-kitty-connect
```

## Scope

```
- contracts/
  * KittyConnect.sol
  * KittyBridge.sol
```
## Roles
- `Cat Owner`: User who buy the cat from our branches and mint NFT for buying a cat.
- `Shop Partner` - Shop partner provide services to the cat owner to buy cat.
- `KittyConnect Owner` - Owner of the contract who can transfer the ownership of the contract to another address.


# Executive Summary
## Issues found
| Severtity | Number of issues found |
| --------- | ---------------------- |
| High      | 1                      |
| Medium    | 0                      |
| Low       | 0                      |
| Gas       | 2                      |
| Info      | 0                      |
| Total     | 3                      |

# Findings
# High
## <a id='H-01'></a>H-01. `KittyConnect::_updateOwnershipInfo` function doesn't update the ownership info of the kitty's previous owner, leads to confusion in managing and querying the ownership of NFTs.            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L181

## Vulnerability Details
When `KittyConnect::safeTransferFrom` function is called, it updates the ownership information of the NFT by calling `KittyConnect::_updateOwnershipInfo`. However, the function does not update the `KittyConnect::s_ownerToCatsTokenId` mapping, where the owner of the NFT is stored.
## Impact
This could lead to confusion or inefficiencies in managing and querying the ownership of NFTs.
## Tools Used
Manual Review

## Proof of Code
The proof of concept is this test, which was already written:
<details>
<summary>PoC</summary>

```javascript
function test_safetransferCatToNewOwner() public {
    string memory catImageIpfsHash = "ipfs://QmbxwGgBGrNdXPm84kqYskmcMT3jrzBN8LzQjixvkz4c62";
    uint256 tokenId = kittyConnect.getTokenCounter();
    address newOwner = makeAddr("newOwner");

    vm.prank(partnerA);
    kittyConnect.mintCatToNewOwner(user, catImageIpfsHash, "Meowdy", "Ragdoll", block.timestamp);

    vm.prank(user);
    kittyConnect.approve(newOwner, tokenId);

    vm.expectEmit(false, false, false, true, address(kittyConnect));
    emit CatTransferredToNewOwner(user, newOwner, tokenId);
    vm.prank(partnerA);
    kittyConnect.safeTransferFrom(user, newOwner, tokenId);

    assertEq(kittyConnect.ownerOf(tokenId), newOwner);
    assertEq(kittyConnect.getCatsTokenIdOwnedBy(user).length, 0);
    assertEq(kittyConnect.getCatsTokenIdOwnedBy(newOwner).length, 1);
    assertEq(kittyConnect.getCatsTokenIdOwnedBy(newOwner)[0], tokenId);
    assertEq(kittyConnect.getCatInfo(tokenId).prevOwner[0], user);
    assertEq(kittyConnect.getCatInfo(tokenId).prevOwner.length, 1);
    assertEq(kittyConnect.getCatInfo(tokenId).idx, 0);
}

```
</details>

By running this test, we can see that this assertion fails:

```javascript
assertEq(kittyConnect.getCatsTokenIdOwnedBy(user).length, 0);
```

## Recommendations

To prevent this, we can remove the tokenId from the array of the previous owner in `KittyConnect::s_ownerToCatsTokenId` mapping. This can be done by adding the following line of code in the `KittyConnect::_updateOwnershipInfo` function:

```diff
function _updateOwnershipInfo(
    address currCatOwner,
    address newOwner,
    uint256 tokenId
) internal {
+   // Get the index of the token ID in the array
+   uint256 tokenIndex = s_ownerToCatsTokenId[currCatOwner].length;
+   for (uint256 i = 0; i < tokenIndex; i++) {
+       if (s_ownerToCatsTokenId[currCatOwner][i] == tokenId) {
+           tokenIndex = i;
+           break;
+       }
+   }
+
+   // Swap the token ID to be removed with the last element in the array
+   s_ownerToCatsTokenId[currCatOwner][tokenIndex] = s_ownerToCatsTokenId[
+        currCatOwner
+   ][s_ownerToCatsTokenId[currCatOwner].length - 1];

+   // Pop the last element from the array
+   s_ownerToCatsTokenId[currCatOwner].pop();
    s_catInfo[tokenId].prevOwner.push(currCatOwner);
    s_catInfo[tokenId].idx = s_ownerToCatsTokenId[newOwner].length;
    s_ownerToCatsTokenId[newOwner].push(tokenId);
}
```
# Gas
## <a id='G-01'></a>G-01. Using `require` instead of custom errors, leads to gas inefficiency.            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L46

https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L51

https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L56

https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L93

https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L118

https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L129

## Description
All of the error handling in the `KittyConnect` contract is done using the require function. Since Solidity v0.8.4, custom reverts were presented, which are more gas efficient than using `require`.

## Impact
Leads to spending more gas than necessary.

## Tools Used
Manual Review

## Recommendations
Read about custom errors in this [solidity blog](https://soliditylang.org/blog/2021/04/21/custom-errors/), and rewrite them with the new syntax.


## <a id='G-02'></a>G-02. Reading from storage in `KittyConnect::constructor` every iteration in the for-loop, which is gas inefficient.            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-03-kitty-connect/blob/c0a6f2bb5c853d7a470eb684e1954dba261fb167/src/KittyConnect.sol#L62

## Vulnerability Details
In `KittyConnect::constructor`, the function reads from storage in every iteration of the for-loop. Reading from storage is more expensive than reading from memory.

## Impact
Leads to spending more gas than necessary.

## Tools Used
Manual Review

## Recommendations
Consider storing the length of the array in a local variable and using it in the for-loop. This will reduce the gas cost of the function.

```diff
constructor(
    address[] memory initShops,
    address router,
    address link
) ERC721("KittyConnect", "KC") {
+   uint256 initialShopsArray = initShops.length;
+   for (uint256 i = 0; i < initialShopsArray; i++) {
-   for (uint256 i = 0; i < initShops.length; i++) {
        s_kittyShops.push(initShops[i]);
        s_isKittyShop[initShops[i]] = true;
    }

    i_kittyConnectOwner = msg.sender;
    i_kittyBridge = new KittyBridge(router, link, msg.sender);
}
```