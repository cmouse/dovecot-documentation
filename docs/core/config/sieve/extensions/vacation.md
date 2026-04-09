---
layout: doc
title: Vacation
dovecotlinks:
  sieve_vacation: Sieve vacation extension
---

# Sieve: Vacation Extension

The Sieve vacation extension ([[rfc,5230]]) defines a mechanism
to generate automatic replies to incoming email messages. It takes
various precautions to make sure replies are only sent when appropriate.

Script authors can specify how often replies can be sent to a particular
contact. In the original vacation extension, this interval is specified
in days with a minimum of one day. When more granularity is necessary
and particularly when replies must be sent more frequently than one day,
the vacation-seconds extension ([[rfc,6131]]) can be used. This
allows specifying the minimum reply interval in seconds with a minimum
of zero (a reply is then always sent), depending on administrator
configuration.

## Configuration

The vacation extension is available by default.

In contrast, the vacation-seconds extension - which implies the vacation
extension when used - is not available by default and needs to be enabled
explicitly by adding it to [[setting,sieve_extensions]].

The configuration also needs to be adjusted accordingly to allow a non-reply
period of less than a day.

### Settings

<SettingsComponent tag="sieve-vacation" level="3" />

## Sieve Example

```sieve
require ["vacation"];

vacation :days 7
  :subject "Out of office"
  "I am currently away and will reply when I return.";
```

This example sends an automatic reply at most once every 7 days per sender.
