# bookclerk_media_worker

Runs exactly one media job inside a jail derived from that job.

The process reads a JSON [`MediaJob`](../bookclerk_media/job.md#bookclerk_mediajobmediajob) on stdin, confines itself to the paths
the job declared, runs it, and writes a JSON [`MediaJobReply`](../bookclerk_media/job.md#bookclerk_mediajobmediajobreply) to stdout.
One job per process, so the allowlist is only ever as wide as a single
book's inputs and one output directory.

On Linux and macOS confinement happens here, before any media is touched and
while the process is still single-threaded — which is why this is a separate
binary rather than a thread in the host.

On Windows the host launches this binary through `bookclerk-jail`, which
applies an AppContainer at `CreateProcess`. This process then skips
self-confine (unsupported) and relies on that spawn-time jail.

Everything this binary needs is already resolved by the time it confines
itself, so the jail can deny the rest of the filesystem — including the
host's `master.key` and `library.db`.

## Modules

