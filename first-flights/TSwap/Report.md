# Unused deadline in `deposit` function (Severity: Medium, Valid)

## Relevant GitHub Links
https://github.com/Cyfrin/2024-06-t-swap/blob/d1783a0ae66f4f43f47cb045e51eca822cd059be/src/TSwapPool.sol#L117
## Summary
No deadline check in `deposit` function
## Vulnerability Details
The `uint64 deadline` param of the `deposit` function is not used by the modifiers, therefore a deposit with no time limit could be executed.
## Impact
Alice wants to deposit some amount of tokens and sends a transaction to the mempool, but with a very low has fee. Validators see the transaction but the fee is not attractive, so the transaction will be pending for a long time of period. Let's say that after a week the average gas fees drop low enough for the validators to execute the transaction but the price of the assets will has changed drastically.
## Tools Used
Manual Review
## Recommendations
Add the `revertIfDeadlinePassed(deadline)` modifier.

# `swapExactInput` function returns wrong variable (Severity: Low, Valid)

## Relevant GitHub Links
https://github.com/Cyfrin/2024-06-t-swap/blob/d1783a0ae66f4f43f47cb045e51eca822cd059be/src/TSwapPool.sol#L306
## Summary
`swapExactInput` function returns wrong variable
## Vulnerability Details
Function `swapExactInput` returns a uint256 variable `output` which is never used. If we were to follow the example of the other function - `swapExactOutput`, the function should return the variable that is used by `getOutputAmountBasedOnInput`.
## Impact

## Tools Used
Manual Review
## Recommendations
The new function should look like this:
```solidity
function swapExactInput(
        IERC20 inputToken,
        uint256 inputAmount,
        IERC20 outputToken,
        uint256 minOutputAmount,
        uint64 deadline
    )
        public
        revertIfZero(inputAmount)
        revertIfDeadlinePassed(deadline)
        returns (uint256 outputAmount)
    {
        uint256 inputReserves = inputToken.balanceOf(address(this));
        uint256 outputReserves = outputToken.balanceOf(address(this));

        outputAmount = getOutputAmountBasedOnInput(
            inputAmount,
            inputReserves,
            outputReserves
        );

        if (outputAmount < minOutputAmount) {
            revert TSwapPool__OutputTooLow(outputAmount, minOutputAmount);
        }

        _swap(inputToken, inputAmount, outputToken, outputAmount);
    }
```
