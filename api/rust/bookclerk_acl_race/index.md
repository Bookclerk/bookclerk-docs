# bookclerk_acl_race

Cross-process ACL grant/revoke helper for Windows integration tests.

Repeatedly grants then revokes a Package SID ACE on a shared directory so
two independent processes can race the DACL RMW path.

## Modules

