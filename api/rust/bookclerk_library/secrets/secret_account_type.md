**bookclerk_library > secrets > secret_account_type**

# Module: secrets::secret_account_type

## Contents

**Constants**

- [`INTEGRATION`](#integration) - Store / portal integration accounts (Audible, Libro, Chirp, GA, Widevine).
- [`OPERATOR`](#operator) - Operator-owned destination / control-plane secrets (S3, …).
- [`PLUGIN_INSTANCE`](#plugin_instance) - Secrets owned by one [`crate::control_plane::PluginInstanceId`](../control_plane/instances.md#bookclerk_librarycontrol_planeinstancesplugininstanceid).
- [`USER`](#user) - First-party user secrets (TOTP), purged when that user is deleted.

---

## bookclerk_library::secrets::secret_account_type::INTEGRATION

*Constant*: `&str`

Store / portal integration accounts (Audible, Libro, Chirp, GA, Widevine).



## bookclerk_library::secrets::secret_account_type::OPERATOR

*Constant*: `&str`

Operator-owned destination / control-plane secrets (S3, …).



## bookclerk_library::secrets::secret_account_type::PLUGIN_INSTANCE

*Constant*: `&str`

Secrets owned by one [`crate::control_plane::PluginInstanceId`](../control_plane/instances.md#bookclerk_librarycontrol_planeinstancesplugininstanceid).

Not purged with store accounts. `account_id` is the instance id.



## bookclerk_library::secrets::secret_account_type::USER

*Constant*: `&str`

First-party user secrets (TOTP), purged when that user is deleted.



