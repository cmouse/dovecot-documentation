---
layout: doc
title: Spamtest/Virustest
dovecotlinks:
  sieve_spamtest: Sieve spamtest extension
  sieve_virustest: Sieve virustest extension
---

# Sieve: Spamtest and Virustest Extensions

Using the spamtest and virustest extensions ([[rfc,5235]]), the Sieve language
provides a uniform and standardized command interface for evaluating
spam and virus tests performed on the message. Users no longer need to
know what headers need to be checked and how the scanner's verdict is
represented in the header field value. They only need to know how to use
the spamtest (spamtestplus) and virustest extensions.

This also gives GUI-based Sieve editors the means to provide a portable and
easy to install interface for spam and virus filter configuration.

The burden of specifying which headers need to be checked and how the
scanner output is represented falls onto the Sieve administrator.

## Configuration

The spamtest, spamtestplus, and virustest extensions are not
enabled by default and thus need to be enabled explicitly using
[[setting,sieve_extensions]].

### Settings: Spamtest

<SettingsComponent tag="sieve-spamtest" level="3" />

### Settings: Virustest

<SettingsComponent tag="sieve-virustest" level="3" />

## Sieve Example

```sieve
require ["spamtest", "fileinto", "relational", "comparator-i;ascii-numeric"];

if spamtest :value "ge" :comparator "i;ascii-numeric" "5" {
  fileinto "Spam";
}
```

This example files messages into the Spam folder when the spam score
meets or exceeds the configured threshold.
