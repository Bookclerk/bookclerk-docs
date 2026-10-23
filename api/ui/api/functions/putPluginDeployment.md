[**bookclerk-ui**](../../README.md)

***

[bookclerk-ui](../../modules.md) / [api](../README.md) / putPluginDeployment

# Function: putPluginDeployment()

> **putPluginDeployment**(`instanceId`, `nodeId`, `desired`, `expectedRevision?`): `Promise`\<[`PluginDeploymentView`](../interfaces/PluginDeploymentView.md)\>

Defined in: [api.ts:2076](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/ui/src/lib/api.ts#L2076)

Sets whether a plugin instance should run on a host (the enable switch).

## Parameters

### instanceId

`string`

Plugin instance id.

### nodeId

`string`

Target node id.

### desired

`"present"` \| `"absent"`

`present` to run it there, `absent` to retire it.

### expectedRevision?

`number`

Deployment revision last observed, for compare-and-swap.

## Returns

`Promise`\<[`PluginDeploymentView`](../interfaces/PluginDeploymentView.md)\>

The deployment row after the write.
