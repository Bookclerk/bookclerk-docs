# bookclerk_ac_probe

Windows AppContainer probe used by jail integration tests.

Reports TokenIsAppContainer, path read/write, cwd/temp, optional TEMP
create/read/delete, and child-spawn helpers for Job Object tests. On
non-Windows hosts this binary exits with an error.

## Modules

