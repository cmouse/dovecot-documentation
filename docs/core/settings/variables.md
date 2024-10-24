---
layout: doc
title: Settings Variables
dovecotlinks:
  settings_variables: Settings Variables
  settings_variables_distribution_variables:
    hash: distribution-variables
    text: Distribution Variables
  settings_variables_mail_user_variables:
    hash: mail-user-variables
    text: Mail User Variables
  login_variables:
    hash: login-variables
    text: Login Variables
  variables_modifiers:
    hash: modifiers
    text: variable modifiers
  conditionals:
    hash: conditionals
    text: conditionals
---

# Settings Variables

You can use special variables in several places:

* All settings, except of type [[link,settings_types_string_novar]]. Most
  commonly used by [[link,mail_location]].
* Static [[link,userdb]] and [[link,auth_passwd_file]] template strings
* [[link,auth_ldap]], [[link,auth_sql]], and [[link,userdb]] query strings
* Log prefix for imap/pop3 process

## Variable expansion syntax

In Dovecot 2.4/3.1 we have introduced a totally new variable expansion syntax.
See [[link,var_expand]] for in-depth details how it works.

Basic syntax is `%{variable (| filter | filter ...)}`, which means that most existing
variables work, but there are some changes, so check them carefully.

The simple case of just getting a value of variable is `%{variable}`.
These can be in middle of strings.

Another way to get variables is `%{provider:variable}`, where the value is provided by
a provider. There are global providers, and context-specific providers.

Variable can be then filtered with various filters, such as `%{variable | upper}` to get
uppercase representation of variable. You can add as many filters as you need.

Filters can accept parameters, both positional and named. E.g. `%{literal('\r\n\)}` will expand
to CR LF. '%{user | substr(1)}` will take first character of username.

You can use `%%{variable}` to escape this, and emit `%{variable}`.

Filters accept either direct value or variable as a parameter. Key names cannot be variables.
The left side of pipe character (`|`) is provided as input to a filter. Some filters can be used
in place of variables, e.g. lookup, literal and if.

When value is missing or empty, you can use `default` filter to provide value. Missing variables
will cause errors and must be negated with defaults. This does not apply to all providers, some
providers return empty when value is missing.

The new syntax also supports simple mathematics, you can do one operation. E.g. `%{port + 1000}`.
Addition, substraction, multiplication, division and modulo are supported for now.

All strings must be encapsulated with " or ', and you can escape them using \ within string.
Numbers when used as parameters must be provided without quotes. 

## List of filters

All parameters are strings unless stated otherwise.
If parameter is any, it accepts both numbers and strings.
Boolean type is 0 = false or 1 = true.

| Filter | Description |
| ------ | ----------- |
| lookup(name) | Lookup var from table. If var is variable, the name is taken from variable's contents. |
| literal(string) | Expands into literally the value. If variable is used, works like lookup. | 
| concat(any, any...) | Concatenates input with value(s). Numbers are coerced to strings. Input is optional. |
| upper | Uppercases input. |
| lower | Lowercases input. |
| hash(method, rounds=number, salt=string) | Returns raw hash from input using given hash method. Rounds and salt are optional. | 
| md5(rounds=number, salt=string) | Alias for hash with method md5. |
| sha1(rounds=number, salt=string) | Alias for hash with method sha1. |
| sha256(rounds=number, salt=string) | Alias for hash with method sha256. |
| sha384(rounds=number, salt=string) | Alias for hash with method sha384. |
| sha512(rounds=number, salt=string) | Alias for hash with method sha512. |
| base64(pad=boolean, url=boolean) | Base64 encode given input, defaults to pad and not url scheme. |
| unbase64(pad=boolean, url=boolean) | Base64 decode given input, defaults to pad and not url scheme. |
| hex(number) | Hex encode input, with optional width. |
| unhex | Hex decode input. |
| default(value) | Replace empty or missing input with value. Clears missing variable error. If no value is provided, "" is used. |
| reverse | Reverse input bytes. |
| truncate(len, bits=number) | Truncate to len bytes, or number of bits. |
| substr(start, end) | Substring bytes of input from start to end. Start and end can be negative numbers, in which case they are counted from right. End is optional. |
| ldap_dn | Converts domain.com to dc=domain,dc=com. |
| if(left,operator,right,true,false) | Evaluates given comparison and returns true or false value. See [conditionals](#conditionals). |
| if(operator,right,true,false | Evaluates given comparison against input value and retuns true or false value. |
| regexp(expression, replacement) | Performs regular expression replacement using PCRE2 syntax. Supports up to 9 capture groups. |
| number | Converts input number into input bytes. |
| index(separator, nth) | Returns nth element from separator separated string. |
| username | Provides local part of user@domain value. |
| domain | Provides domain part of user@domain value. |
| list(separator) | Converts tab-escaped list into separator separated list. Defaults to `,`. |
| lfill(filler, width) | Pads value from left with filler until length is width. |
| rfill(filler, width) | Pads value to right with filler until length is width. |
| fold(modulo) | Folds value to 64-bit unsigned integer with given modulo. Modulo is optional. |

If [[plugin,var-expand-crypt]] is loaded, these filters are registered as well.

| Filter | Description |
| ------ | ----------- |
| encrypt(algorithm=string,key=string,iv=string,raw=boolean) | Encrypts input with given parameters. |
| decrypt(algorithm=string,key=string,iv=string,raw=boolean) | Decrypts input with given parameters. |

## Global providers

Global providers that work everywhere are:

| Long Name | Description |
| --------- | ----------- |
| `env:<name>` | Environment variable \<name\>. Returns "" if unset. |
| `system:<name>` | Get a system variable, see [below](#system-variables) for list of supported names. |
| `process:<name>` | Get a process variable, see [below](#process-variables) for list of supported names. |
| `dovecot:<name>` | Get a distribution variable, see [below](#distribution-variables) for a list of supported names. |
| `event:<name>`   | Get an event field. Returns "" if no such field is found from event. |

## System Variables

### `cpu_count`

Number of CPUs available. Works only on Linux and FreeBSD-like systems.

Can be overridden with `NCPU` environment variable.

This needs to be included in [[setting,import_environment]].

### `hostname`

Hostname (without domain). Can be overridden with `DOVECOT_HOSTNAME`
environment variable.

This needs to be included in [[setting,import_environment]].

### `os`

OS name reported by `uname()` call. (Similar to `uname -s` output.)

### `os-version`

OS version reported by `uname()` call. (Similar to `uname -r` output.)

## Process Variables

### `pid`

Current process ID.

### `uid`

Effective user ID of the current process.

### `gid`

Effective group ID of the current process.

## Distribution Variables

### `name`

Name of distributed package. (Default: `Dovecot`)

### `version`

Dovecot version.

### `support-url`

Support webpage set in Dovecot distribution. (Default:
https://www.dovecot.org/)

### `support-email`

Support email set in Dovecot distribution. (Default: `dovecot@dovecot.org`)

### `revision`

Short commit hash of Dovecot git source tree HEAD. (Same as the commit hash
reported in `dovecot --version`.)

## User Variables

::: tip
See also:
* [Global Variables](#global-variables)
:::

Variables that work nearly everywhere where there is a username:

| Variable | Description |
| -------- | ----------- |
| `%{user}` | Full username (e.g. user@domain) |
| `%{user | username}` | User part in user@domain, same as `%{user}` if there's no domain |
| `%{user | domain}` | Domain part in user@domain, empty if user with no domain |
| `session` | Session ID for this client connection (unique for 9 years) |
| `auth_user` | SASL authentication ID (e.g. if master user login is done, this contains the master username). If username changes during authentication, this value contains the original username. Otherwise the same as `%{user}`. |

## Mail Service User Variables

::: tip
See also:
* [Global Variables](#global-variables), and
* [User Variables](#user-variables).
:::

| Variable | Description |
| -------- | ----------- |
| `service` | imap, pop3, smtp, lda (and doveadm, etc.) |
| `local_ip` | local IP address |
| `remote_ip` | remote IP address |
| `userdb:<name>` | Return userdb extra field "name". |

## Mail User Variables

::: tip
See also:
* [Global Variables](#global-variables),
* [User Variables](#user-variables), and
* [Mail Service User Variables](#mail-service-user-variables).
:::

| Variable | Description |
| -------- | ----------- |
| `%{home}` | home directory. Use of `~/` is better whenever possible. |
| `%{hostname}` | Expands to the hostname setting. Overrides the global `%{hostname}`. |

## Login Variables

::: tip
See also:
* [Global Variables](#global-variables), and
* [User Variables](#user-variables).
:::

| Variable | Description |
| -------- | ----------- |
| `protocol` | imap, pop3, smtp, lda (and doveadm, etc.)<br/>[[added,variables_login_variables_protocol]] Renamed from `%{service}` variable. |
| `local_name` | TLS SNI hostname, if given |
| `local_ip` | local IP address |
| `remote_ip` | remote IP address |
| `local_port` | local port |
| `remote_port` | remote port |
| `real_remote_ip` | Same as `%{remote_ip}`, except in proxy setups contains the remote proxy's IP instead of the client's IP |
| `real_local_ip` | Same as `%{local_ip}`, except in proxy setups contains the local proxy's IP instead of the remote proxy's IP |
| `real_remote_port` | Similar to `%{real_rip}` except for port instead of IP |
| `real_local_port` | Similar to `%{real_lip}` except for port instead of IP |
| `mechanism` | [[link,sasl]], e.g., PLAIN |
| `secured` | "TLS" with established SSL/TLS connections, "TLS handshaking", or "TLS [handshaking]: error text" if disconnecting due to TLS error. "secured" with secured connections (see: [[setting,ssl]]). Otherwise empty. |
| `ssl_security` | TLS session security string. If HAProxy is configured and it terminated the TLS connection, contains "(proxied)". |
| `ssl_ja3` | [[link,ssl_ja3]] composed from TLS Client Hello. |
| `ssl_ja3_hash` | MD5 hash from [[link,ssl_ja3]] composed from TLS Client Hello. |
| `mail_pid` | PID for process that handles the mail session post-login |
| `original_user` | Same as `%{user}`, except using the original username the client sent before any changes by auth process. With master user logins (also with [[setting,auth_master_user_separator]] based logins),this contains only the original master username. |
| `listener` | Socket listener name as specified in config file, which accepted the client connection. |
| `owner_user` | For shared storage this is the `%{user}` variable of the owner, otherwise it is the same as `%{user}`.<br />[[added,variables_owner_user_added]] |
| `passdb:<name>` | Return passdb extra field "name". |
| `passdb:forward_<name>` | Used by proxies to pass on extra fields to the next hop, see [[link,auth_forward_fields]]. |

## Authentication Variables

::: tip
See also:
* [Global Variables](#global-variables), and
* [User Variables](#user-variables).
:::

| Variable | Description |
| -------- | ----------- |
| `protocol` | imap, pop3, smtp, lda (and doveadm, etc.)<br/>[[added,variables_auth_variables_protocol]] Renamed from `%{service}` variable. |
| `domain_first` | For "username@domain_first@domain_last" style usernames |
| `domain_last` | For "username@domain_first@domain_last" style usernames |
| `local_name` | TLS SNI hostname, if given |
| `local_ip` | local IP address |
| `remote_ip` | remote IP address |
| `local_port` | local port |
| `remote_port` | remote port |
| `real_remote_ip` | Same as `%{remote_ip}`, except in proxy setups contains the remote proxy's IP instead of the client's IP |
| `real_local_ip` | Same as `%{local_ip}`, except in proxy setups contains the local proxy's IP instead of the remote proxy's IP |
| `real_remote_port` | Similar to `%{real_rip}` except for port instead of IP |
| `real_local_port` | Similar to `%{real_lip}` except for port instead of IP |
| `client_pid` | process ID of the authentication client |
| `session_pid` | For user logins: The PID of the IMAP/POP3 process handling the session. |
| `mechanism` | [[link,sasl]], e.g., PLAIN |
| `password` | cleartext password from cleartext authentication mechanism |
| `secured` | "TLS" with established SSL/TLS connections, "secured" with secured connections (see: [[setting,ssl]]). Otherwise empty. |
| `ssl_ja3_hash` | MD5 hash from JA3 string composed from TLS Client Hello. |
| `cert` | "valid" if client had sent a valid client certificate, otherwise empty. |
| `login_user` | For master user logins: Logged in user@domain |
| `master_user` | For master user logins: The master username |
| `original_user` | Same as `%{user}`, except using the original username the client sent before any changes by auth process |
| `passdb:<name>` | Return passdb extra field "name". |
| `userdb:<name>` | Return userdb extra field "name". Note that this can also be used in passdbs to access any userdb_\* extra fields added by previous passdb lookups. |
| `client_id` | If [[setting,imap_id_retain]] is enabled this variable is populated with the client ID request as IMAP arglist. For directly logging the ID see the [[event,imap_id_received]] event. |
| `passdb:forward_<name>` | Used by proxies to pass on extra fields to the next hop, see [[link,auth_forward_fields]]. |
| `id` | Internal ID number of the current passdb/userdb. |

## Conditionals

The following operators are supported:

| Operator | Explanation |
| -------- | ----------- |
| `==` | NUMERIC equality |
| `!=` | NUMERIC inequality |
| `<` | NUMERIC less than |
| `<=` | NUMERIC less or equal |
| `>` | NUMERIC greater than |
| `>=` | NUMERIC greater or equal |
| `eq` | String equality |
| `ne` | String inequality |
| `lt` | String inequality |
| `le` | String inequality |
| `gt` | String inequality |
| `ge` | String inequality |
| `*` | Wildcard match (mask on value2) |
| `!*` | Wildcard non-match (mask on value2) |
| `~` | Regular expression match (pattern on value2, extended POSIX) |
| `!~` | String inequality (pattern on value2, extended POSIX) |

Examples:

```
# If %{user} is "testuser", return "INVALID". Otherwise return %{user} uppercased.
%{user | if ("=", "testuser, "invalid", user) | upper }
```
