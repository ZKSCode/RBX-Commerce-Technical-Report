# From inventory management to a counter sale

[Português](https://github.com/ZKSCode/RBX-Commerce-Relatorio-Tecnico/blob/main/docs/estudo-de-caso.md) · [Technical report home](../README.md)

A retailer sells the same T-shirt in several sizes. Staff need to know how many units remain for each variant and match every sale to the corresponding stock movement. At the end of the shift, they reconcile the cash drawer against the recorded movements.

[Follow the workflow in screenshots](gallery.md)

## From product setup to physical counts

Staff assign a color, size, EAN barcode, and minimum stock level to each variant. During a physical count, they compare the quantity on hand with the recorded balance and can enter a reason for any discrepancy before applying an adjustment. The movement history explains how the balance changed.

## From barcode scan to cash reconciliation

The cashier opens a register session with a starting balance. For a numeric barcode, the search tries an exact EAN match first. Prefix and text searches support partial codes and product names. The sale ties each line item to a variant and records the outgoing stock. Discounts and price overrides have separate authorization and reason fields.

At closing, the controller calculates how much physical cash should be in the drawer. It totals the eligible inflows and outflows before applying this calculation.

```php
$expectedBalance += $entradas;
$expectedBalance -= $saidas;

$difference = $request->closing_balance - $expectedBalance;
```

An opening balance of BRL 100, a BRL 40 cash sale, a BRL 10 cash deposit, and a BRL 20 cash withdrawal produce an expected balance of BRL 130. Counting BRL 128 yields a shortage of BRL 2. These figures come from the demo scenario.

## Implementation details for technical interviews

The POS controller locks the variant row within the transaction that writes the sale. A physical count uses the difference between the recorded balance and the count to determine the adjustment. Both workflows can add movements to the history of the same variant.

The [design notes](design.md) describe the interaction. The [validation notes](demonstrations.md) show what was reproduced and identify the environment-specific checks.
