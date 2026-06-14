# xz scripts

`xz` v1 does not require runtime helper scripts.

The skill relies on MCP tools configured by the host agent environment using the required names:

- `elk` for log queries
- `sk` for trace and call-chain queries
- `sql` for read-only SQL queries
- `arthas` for JVM diagnostics (Java/JVM only, read-only)

If a future environment needs adapter scripts for MCP protocol differences, place them in this directory and document their invocation here.
