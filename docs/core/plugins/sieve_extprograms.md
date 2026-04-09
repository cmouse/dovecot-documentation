---
layout: doc
title: sieve-extprograms
---

# Pigeonhole Sieve Extprograms Plugin (`sieve-extprograms`)

The "sieve_extprograms" plugin provides an extension to the Sieve
filtering language, adding new action commands for invoking a predefined
set of external programs.

Messages can be piped to or filtered through those programs and string
data can be input to and retrieved from those programs.

To mitigate the security concerns, the external programs cannot be chosen
arbitrarily; the available programs are restricted through administrator
configuration.

## Configuration

The plugin is activated by adding it to the [[setting,sieve_plugins]]
setting:

```doveconf[dovecot.conf]
sieve_plugins {
  sieve_extprograms = yes
}
```

## Sieve Example

```sieve
require ["vnd.dovecot.pipe"];

pipe "process-message";
```

This example pipes the message to an external program named
`process-message`, which must be configured and available via the plugin.
