[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / AdapterReceipt

# Interface: AdapterReceipt

Defined in: [generated.ts:2606](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2606)

Replay receipt the host asks the adapter to persist with the batch.

## Properties

### guestHash

> **guestHash**: `string`

Defined in: [generated.ts:2610](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2610)

Guest `requestHash` compared on replay.

***

### guestLen

> **guestLen**: `number`

Defined in: [generated.ts:2608](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/generated.ts#L2608)

Guest statement count inside the receipt wrap (excluding host prune/select).
