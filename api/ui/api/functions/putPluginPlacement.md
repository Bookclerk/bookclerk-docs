[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putPluginPlacement

# Function: putPluginPlacement()

> **putPluginPlacement**(`instanceId`, `body`): `Promise`\<\{ `placement`: [`PluginPlacementView`](../interfaces/PluginPlacementView.md); `plugin_instance_id`: `string`; \}\>

Defined in: [api.ts:2102](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2102)

Hands an instance's deployments to the cluster scheduler, or changes its
placement (compare-and-swap on `expected_revision`; 0 creates).

## Parameters

### instanceId

`string`

Plugin instance id.

### body

Replica count, label constraints, and the revision last seen.

#### constraints

`string`[]

#### expected_revision

`number`

#### replicas

`number` \| `"all"`

## Returns

`Promise`\<\{ `placement`: [`PluginPlacementView`](../interfaces/PluginPlacementView.md); `plugin_instance_id`: `string`; \}\>

The stored placement.
