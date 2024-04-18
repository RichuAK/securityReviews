# First Flight #13: Baba Marta - Findings Report

# Table of contents
- ## [Contest Summary](#contest-summary)
- ## [Results Summary](#results-summary)
- ## High Risk Findings
    - ### [H-01. Users can get as many HealthTokens as they want by presenting to themselves](#H-01)
    - ### [H-02. Failure of token sale in marketplace](#H-02)
    - ### [H-03. Lack of access control for `updateCountMartenitsaTokensOwner`](#H-03)
- ## Medium Risk Findings
    - ### [M-01. `CancelListing` mentioned but not implemented](#M-01)
    - ### [M-02. Collected rewards getting a clean slate after each reward receival.](#M-02)
    - ### [M-03. _nextTokenId updation done prematurely](#M-03)
- ## Low Risk Findings
    - ### [L-01. Token Id pushed multiple times](#L-01)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: First Flight #13

### Dates: Apr 11th, 2024 - Apr 18th, 2024

[See more contest details here](https://www.codehawks.com/contests/cluseb1bf0001s4tjl2rzajup)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 3
   - Medium: 3
   - Low: 1


# High Risk Findings

## <a id='H-01'></a>H-01. Users can get as many HealthTokens as they want by presenting to themselves            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/blob/5eaab7b51774d1083b926bf5ef116732c5a35cfd/src/MartenitsaMarketplace.sol#L90

## Summary

HealthToken scarcity destroyed through gifting

## Vulnerability Details

By having multiple burner wallets that one controls, and then invoking `makePresent()` on each of them, user can collect as many HealthTokens as possible by invoking `collectReward()` with the address that holds the ERC721 tokens at the moment. 

## Impact

HealthToken scarcity will be compromised, if they're supposed to be valuable. 

## Tools Used

Manual review

## Recommendations

Have a different way to track reward eligibility, instead of token balance, since token balance is dynamic and users can use the same tokens to get rewards by using burner wallets.
## <a id='H-02'></a>H-02. Failure of token sale in marketplace            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/blob/5eaab7b51774d1083b926bf5ef116732c5a35cfd/src/MartenitsaMarketplace.sol#L82

## Summary
Listed token will not successfully transferred to the buyer

## Vulnerability Details
Since the marketplace was not approved by the seller at the time of listing, the marketplace contract won't be able to successfully transfer the token from the seller to the buyer.

## Impact

No sales will ever happen in the marketplace

## Tools Used

Manual review 

## Recommendations

Add an `approve` function in `listMartenitsaForSale` function so the marketplace contract is approved by the seller for the tokenId, which will enable the marketplace to do a transfer to the buyer.
## <a id='H-03'></a>H-03. Lack of access control for `updateCountMartenitsaTokensOwner`            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/blob/5eaab7b51774d1083b926bf5ef116732c5a35cfd/src/MartenitsaToken.sol#L62C74-L62C97

## Summary
`updateCountMartenitsaTokensOwner` in MartenitsaToken.sol should be properly access controlled for the protocol to work properly.

## Vulnerability Details
Since the function can be called anyone by passing in a proper `address` and `operation`, user balance of MartenitsaTokens are fully compromised.

## Impact

User balance of tokens compromised, further logic which is dependant on the logic fails.

## Tools Used

Manual Review

## Recommendations

Add proper access control for the `updateCountMartenitsaTokensOwner` function.
		
# Medium Risk Findings

## <a id='M-01'></a>M-01. `CancelListing` mentioned but not implemented            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/tree/main?tab=readme-ov-file#martenitsamarketplacesol

## Summary

Cancel listing is mentioned in the docs, but not implemented in the contract

## Vulnerability Details

No implementation of the function mentioned 

## Impact

A seller will be locked in on a price for a token until it's bought by someone. 

## Tools Used

Manual review 

## Recommendations

Implement the `cancelListing` function
## <a id='M-02'></a>M-02. Collected rewards getting a clean slate after each reward receival.            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/blob/5eaab7b51774d1083b926bf5ef116732c5a35cfd/src/MartenitsaMarketplace.sol#L106

## Summary

Collected rewards is getting reassigned instead of being updated 

## Vulnerability Details

`_collectedRewards[msg.sender]` only reflecting the previous rewards he's redeemed,  and not the whole rewards collected based on token balance.

## Impact

This would lead to the user being able to receive more HealthTokens than his proper share. 


## Tools Used

Manual review
## Recommendations

Update `_collectedRewards[msg.sender] = amountRewards;` with `_collectedRewards[msg.sender] += amountRewards;`
## <a id='M-03'></a>M-03. _nextTokenId updation done prematurely            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/blob/5eaab7b51774d1083b926bf5ef116732c5a35cfd/src/MartenitsaToken.sol#L39

## Summary

Variable `_nextTokenId` incremented before the current value is assigned to the token to be minted.

## Vulnerability Details

`uint256 tokenId = _nextTokenId++;` 


## Impact
View function `getDesign()` will throw an error on the latest minted token, and view function `getNextTokenId()` will return the tokenId of the latest minted token instead of the next token, leading to erratic behaviour on integration.

## Tools Used

Manual review 
## Recommendations

Update line 39 to two lines:
```
uint256 tokenId = _nextTokenId;
_nextTokenId++;
```

# Low Risk Findings

## <a id='L-01'></a>L-01. Token Id pushed multiple times            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-04-Baba-Marta/blob/5eaab7b51774d1083b926bf5ef116732c5a35cfd/src/MartenitsaVoting.sol#L51

## Summary

Token Id pushed possibly multiple times as part of the vote count
## Vulnerability Details
 Token ID being pushed into `_tokenIds` array whenever a vote for the token Id is counted

## Impact

Counting votes and deciding the winner will be very gas consuming since the `_tokenIds` array will be much longer than required

## Tools Used

Manual review 
## Recommendations

Do a check whether`tokenId` is already in the `_tokenIds` array before pushing

replace line 51 with:

```
if(voteCounts[tokenId]==1) {
_tokenIds.push(tokenId);
}

```


