# receive

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

Contract: [`JBETHERC20SplitsPayer`](/api/contracts/or-utilities/jbetherc20splitspayer/README.md)

Interface: [`IJBSplitsPayer`](/api/interfaces/ijbsplitspayer.md)

<Tabs>
<TabItem value="Step by step" label="Step by step">

**Received funds are paid to the default split group using the stored default properties.**

_This function is called automatically when the contract receives an ETH payment._


#### Definition

```
receive() external payable virtual override { ... }
```

* The function is triggered when the contract receives ETH.
* The function can be accessed externally by anyone.
* The function doesn't return anything.

#### Body

1.  Pay the splits and get a reference to the leftover amount.

    ```solidity
    // Pay the splits and get a reference to the amount leftover.
    uint256 _leftoverAmount = _payToSplits(
      defaultSplitsProjectId,
      defaultSplitsDomain,
      defaultSplitsGroup,
      JBTokens.ETH,
      address(this).balance,
      18 // decimals.
    );
    ```

    _Internal references:_

    * [`_payToSplits`](/api/contracts/or-utilities/jbetherc20splitspayer/write/-_paytosplits.md)

2.  If there's no leftover amount, there's nothing left to do.

    ```solidity
    // If there is no leftover amount, nothing left to pay.
    if (_leftoverAmount == 0) return;
    ```

3.  Pay the leftover ETH to the default project ID using the default parameters. Use the `addToBalance` function if there's a preference to do so. If there's no default project ID, send to the default beneficiary if there is one, otherwise send to the message sender.

    ```
    // If there's a default project ID, try to pay it.
    if (defaultProjectId != 0)
      if (defaultPreferAddToBalance)
        // Pay the project by adding to its balance if prefered.
        _addToBalance(
          defaultProjectId,
          JBTokens.ETH,
          _leftoverAmount,
          18, // decimals.
          defaultMemo
        );
        // Otherwise, issue a payment to the project.
      else
        _pay(
          defaultProjectId,
          JBTokens.ETH,
          _leftoverAmount,
          18, // decimals.
          defaultBeneficiary != address(0) ? defaultBeneficiary : msg.sender,
          0, // min returned tokens.
          defaultPreferClaimedTokens,
          defaultMemo,
          defaultMetadata
        );
    // If no project was specified, send the funds directly to the beneficiary or the msg.sender.
    else
      Address.sendValue(
        defaultBeneficiary != address(0) ? payable(defaultBeneficiary) : payable(msg.sender),
        _leftoverAmount
      );
    ```

    _Library references:_

    * [`Address`](https://docs.openzeppelin.com/contracts/4.x/api/utils#Address)
      * `.sendValue(...)`
    * [`JBTokens`](/api/libraries/jbtokens.md)
      * `.ETH`
      
    _Internal references:_


    * [`defaultProjectId`](/api/contracts/or-utilities/jbetherc20projectpayer/properties/defaultprojectid.md)
    * [`defaultPreferClaimedTokens`](/api/contracts/or-utilities/jbetherc20projectpayer/properties/defaultpreferclaimedtokens.md)
    * [`defaultMemo`](/api/contracts/or-utilities/jbetherc20projectpayer/properties/defaultmemo.md)
    * [`defaultMetadata`](/api/contracts/or-utilities/jbetherc20projectpayer/properties/defaultmetadata.md)
    * [`defaultBeneficiary`](/api/contracts/or-utilities/jbetherc20projectpayer/properties/defaultbeneficiary.md)
    * [`_addToBalance`](/api/contracts/or-utilities/jbetherc20projectpayer/write/-_addtobalance.md)
    * [`_pay`](/api/contracts/or-utilities/jbetherc20projectpayer/write/-_pay.md)

</TabItem>

<TabItem value="Code" label="Code">

```
/** 
  @notice
  Received funds are paid to the default split group using the stored default properties.

  @dev
  This function is called automatically when the contract receives an ETH payment.
*/
receive() external payable virtual override nonReentrant {
  // Pay the splits and get a reference to the amount leftover.
  uint256 _leftoverAmount = _payToSplits(
    defaultSplitsProjectId,
    defaultSplitsDomain,
    defaultSplitsGroup,
    JBTokens.ETH,
    address(this).balance,
    18 // decimals.
  );

  // If there is no leftover amount, nothing left to pay.
  if (_leftoverAmount == 0) return;

  // If there's a default project ID, try to pay it.
  if (defaultProjectId != 0)
    if (defaultPreferAddToBalance)
      // Pay the project by adding to its balance if prefered.
      _addToBalance(
        defaultProjectId,
        JBTokens.ETH,
        _leftoverAmount,
        18, // decimals.
        defaultMemo
      );
      // Otherwise, issue a payment to the project.
    else
      _pay(
        defaultProjectId,
        JBTokens.ETH,
        _leftoverAmount,
        18, // decimals.
        defaultBeneficiary != address(0) ? defaultBeneficiary : msg.sender,
        0, // min returned tokens.
        defaultPreferClaimedTokens,
        defaultMemo,
        defaultMetadata
      );
  // If no project was specified, send the funds directly to the beneficiary or the msg.sender.
  else
    Address.sendValue(
      defaultBeneficiary != address(0) ? payable(defaultBeneficiary) : payable(msg.sender),
      _leftoverAmount
    );
}
```

</TabItem>

<TabItem value="Bug bounty" label="Bug bounty">

| Category          | Description                                                                                                                            | Reward |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| **Optimization**  | Help make this operation more efficient.                                                                                               | 0.5ETH |
| **Low severity**  | Identify a vulnerability in this operation that could lead to an inconvenience for a user of the protocol or for a protocol developer. | 1ETH   |
| **High severity** | Identify a vulnerability in this operation that could lead to data corruption or loss of funds.                                        | 5+ETH  |

</TabItem>
</Tabs>