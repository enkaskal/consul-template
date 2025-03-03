Consul Template Changelog
=========================

## v0.15.0.dev (Unreleased)

IMPROVEMENTS:

  * Allow specifying per-template delimiters [GH-615, GH-389]
  * Allow specifying per-template wait parameters [GH-589, GH-618]

BUG FIXES:

  * Close open connections when reloading configuration [GH-591, GH-595]
  * Do not share catalog nodes [GH-611, GH-572, GH-603]
  * Properly handle empty string in ParseUint [GH-610, GH-609]
  * Cache Vault's _original_ lease duration [5b955a8]
  * Use decimal division for calculating Vault lease durations [87d61d9]
  * Load VAULT_TOKEN environment variable [2431448]
  * Properly clean up quiescence timers when using multiple templates [GH-616]
 * Print a nice error if K/V cannot be exploded [GH-617, GH-596]

## v0.14.0 (March 7, 2016)

DEPRECATIONS:

  * The `vault` template API function has been renamed to `secret` to be in line
    with other tooling. The `vault` API function will continue to work but will
    print a warning to the log file. A future release of Consul Template will
    remove the `vault` API.

NEW FEATURES:

  * Add `secrets` template API for listing secrets in Vault. Please note this
    requires Vault 0.5+ and the secret backend must support listing. Please see
    the Vault documentation for more information [GH-270]

IMPROVEMENTS:

  * Allow passing any kind of object to `toJSON` in the template. Previously
    this was restricted to key-value maps, but that restriction is now removed.
    [GH-553]

BUG FIXES:

  * Parse file permissions as a string in JSON [GH-548]
  * Document how to reload config with signals [GH-522]
  * Stop all dependencies when reloading the running/watcher [GH-534, GH-568]

## v0.13.0 (February 18, 2016)

BUG FIXES:

  * Compile with go1.6 to avoid race [GH-442]
  * Switch to using a pooled transport [GH-546]

## v0.12.2 (January 15, 2016)

BUG FIXES:

  * Fixed an issue when running as PID 1 in a Docker container where Consul
    Template could consume CPU and spuriously think its spwaned sub-processes
    had failed [GH-511]

## v0.12.1 (January 7, 2016)

IMPROVEMENTS:

  * Add support for math operations on uint types [GH-483, GH-484]
  * Make check information available through health service [GH-490]

BUG FIXES:

  * Store vault data on the dependency and handle an error where a failed
    lease renewal would result in `<no data>` in the rendered template. Please
    note, there is a bug in Vault 0.4 with respect to lease renewals that makes
    it inoperable with Consul Template. Please either use Vault 0.3 or wait
    until Vault 0.5 is released (the bug has already been fixed on master).
    [GH-468, GH-493, GH-504]


## v0.12.0 (December 10, 2015)

BREAKING CHANGES:

  * Add support for checking if a node is in maintenance mode [GH-477, GH-455]

    Previously, Consul Template would report nodes in maintenance mode as
    "critical". They will now report as "maintenance" so users can perform more
    detailed filtering. It is unlikely, but if you were filtering critical
    services, nodes/services in maintenance mode will no longer be included.


FEATURES:

  * Add support for de-duplication mode. In de-duplication mode, Consul Template
    uses leader election to elect one Consul Template process to render a
    template. The results of this template are rendered into Consul's key-value
    store, and other templates pull from the pre-rendered template. This option
    is off by default, but it is highly recommended that the option is enabled
    for clusters with a high load factor (number of templates x number of
    dependencies per template). [GH-465]
  * Add support for automatically reaping child processes. This is very useful
    when running Consul Template as PID 1 (like in a Docker container) when no
    init system is present. The option is configurable, but it defaults to "on"
    when the Consul Template process is PID 1. [GH-428, GH-479]


IMPROVEMENTS:

  * Use the `renew-self` endpoint instead of `renew` for renewing the token
    [GH-450]
  * Allow existing templates to be backed up before writing the new one [GH-464]
  * Add support for TLS/SSL mutual authentication [GH-448]
  * Add support for checking if a node is in maintenance mode [GH-477, GH-455]


## v0.11.1 (October 26, 2015)

FEATURES:

  * Accept "unix" as an argument to `timestamp` to generate a unix
    timestamp [GH-422]

IMPROVEMENTS:

  * Make `Path` a public field on the vault secret dependency so other libraries
    can access it

BUG FIXES:

  * Ensure there is a newline at the end of the version output
  * Update README development instructions [GH-423]
  * Adjust error messages so that data does not always "come from Consul"
  * Fix race conditions in tests
  * Update the `LastContact` value for non-Consul dependencies to always
    return 0 [GH-432, GH-433]
  * Always use `DefaultConfig()` in tests to find issues
  * Fix broken math functions - previously add, subtract, multiply, and divide
    for integers would perform the operation on only the first operand
    [GH-430, GH-435]
  * Renew the vault token based off of the auth, not the secret [GH-443]
  * Remove noisy log message [GH-445]


## v0.11.0 (October 9, 2015)

BREAKING CHANGES:

  * Allow configuration of destination file permissions [GH-415, GH-358]

    Previously, Consul Template would inspect the file at the destination path
    and mirror those file permissions, if a file existed. If a file did not
    exist, Consul Template would render the file with 0644 permissions. This was
    acceptable behavior in a pre-Vault world, but now that Consul Template is
    capable of rendering secrets, there is a desire for increased security. As
    such, Consul Template **no longer mirrors existing destination file
    permissions**. Instead, users can specify the file permissions in the
    configuration file. Please see the README for examples. If you were
    previously relying on an existing file's file permissions to enfore the
    destination file permissions, you must switch to specifying the file
    permissions in the configuration file. If you were not dependent on this
    behavior, nothing has changed; the default value is still 0644.

FEATURES:

  * Add `in` and `contains` functions for checking if a slice or array contains
    a given value [GH-366]
  * Add `add` function for calculating the sum of integers/floats
  * Add `subtract` function for calculating the difference of integers/floats
  * Add `multiply` function for calculating the product of integers/floats
  * Add `divide` function for calculating the division of integers/floats

IMPROVEMENTS:

  * Sort serivces by ID as well
  * Add a mechanism for renewing the given Vault token [GH-359, GH-367]
  * Default max-stale to 1s - this severely reduces the load on the Consul
    leader by allowing followers to respond to API requests [GH-386, GH-397]
  * Add GPG signing for SHASUMS on new releases
  * Push watcher errors down to the client in `once` mode [GH-361, GH-418]

BUG FIXES:

  * Set ssl in the CLI [GH-321]
  * **Regression** - Reload configuration on SIGHUP [GH-332]
  * Remove port option from `service` query and documentation - it was unused
    and legacy, but was causing issues and confusion [GH-333]
  * Return the empty value when no parsable value is given [GH-353]
  * Start with a blank configuration when reloading via SIGHUP [GH-393, GH-394]
  * Use an int64 instead of an int to loop function [GH-401, GH-402]
  * Do not remove the Windows file if it exists [GH-378]

## v0.10.0 (June 9, 2015)

FEATURES:

  * Add `plugin` and plugin ecosystem
  * Add `parseBool` function for parsing strings into booleans (GH-312)
  * Add `parseFloat` function for parsing strings into float64 (GH-312)
  * Add `parseInt` function for parsing strings into int64 (GH-312)
  * Add `parseUint` function for parsing strings into uint64 (GH-312)
  * Add `explode` function for exploding the result of `tree` or `ls` into a
    deeply nested  hash (GH-311)
  * Add `toJSON` and `toJSONPretty` function for exporting the result of `tree`
    or `ls`  into a JSON hash (GH-311)
  * Add `toYAML` function for exporting the result of `tree` or `ls` into a
    YAML document (GH-311)
  * Add `node` function for querying nodes (GH-306, GH-309)
  * Add `split` function for splitting a string on a separator (GH-285)
  * Add `join` function for joining a string slice on a given key (GH-285)
  * Add `pid_file` configuration and command line option for specifying the
    location of a pid file on disk (GH-281, GH-286)

IMPROVEMENTS:

  * Allow setting log_level via the configuration file (CLI still take
    precedence if specified)
  * Improve error reporting when loading multiple configs by including the path
    on the configuration file that had an error (GH-275)
  * Add a timeout around command execution to prevent hanging (GH-283)
  * Read Vault/Consul environment variables for the config (GH-307, GH-308)

BUG FIXES:

  * Properly merge "default" config values with user-supplied values (GH-271)


## v0.9.0 (April 29, 2015)

FEATURES:

  * Add Vault functionality for querying secrets from Vault (GH-264)
  * Add `regexMatch` template helper to determine if a result matches the given
    regular expressions (GH-246)
  * Add support for `ssl-cert` and `ss-ca-cert` options (GH-255)

IMPROVEMENTS:

  * Expand `byTag` to accept catalog services as well (GH-249, GH-250)
  * Allow catalog service tags to use the `.Contains` function (GH-261)

BUG FIXES:

  * Send the standard error of commands back over the standard error of
    Consul Template (GH-253, GH-254)
  * Allow specifying `-v` in addition to `-version` to get the version output

## v0.8.0 (March 30, 2015)

FEATURES:

  * Add `.Size()` so the watcher can report its size (GH-206)
  * Add `byKey` template helper to group the results of a `tree` function by
    their containing directory (GH-207, GH-209, GH-241)
  * Add `timestamp` template function for returning the current timestamp with
    the ability to add custom formatting (GH-225, GH-230)
  * Add `loop` template function for iteration (GH-238, GH-221)

IMPROVEMENTS:

  * Expose `LastIndex` and `ReceivedData` from the Watcher
  * Add unimplemented KV fields (GH-203)
  * Warn the user if there are a large number of dependencies (GH-205)
  * Extend documentation on how health service dependencies are downloaded from
    Consul (GH-212)
  * Allow empty configuration directories (GH-217)
  * Document caveats around using `parseJSON` during multi-evaluation
  * Print the final configuration as JSON in debug mode (GH-231)
  * Export certain environment variables when executing commands that are read
    by other Consul tooling or in your scripts (GH-232) - see the README for
    more information
  * Adjust logging to be less noisy without compromising information (GH-242)

BUG FIXES:

  * Properly filter services by their type (GH-210, GH-212)
  * Return an error if extra arguments are given on the command line (GH-227)
  * Do not overwrite given configuration with the default options (GH-228, GH-219)
  * Check for the correct conditions when using basic authentication (GH-220)
  * Remove unused code paths for clarity (GH-242)
  * Remove race condition in templates when called concurrently (GH-242)
  * Remove race condition in test suite (GH-242)
  * Force a refresh if Consul's WaitIndex is less than our current value (GH-242)
  * Avoid pushing data onto the watcher when the view has been stopped (GH-242)
  * Do not accept data in the runner for an unwatched dependency (GH-198, GH-242)

## v0.7.0 (February 19, 2015)

BREAKING CHANGES:

  * Remove `ssl` configuration option from templates - use an `ssl`
    configuration block with `enabled = true` instead
  * Remove `ssl_no_verify` configuration option from templates - use an `ssl`
    configuration block with `verify = false` instead
  * Restructure CLI `-ssl-no-verify` to `-ssl-verify` - to disable SSL
    certification validation on the command line, use `-ssl-verify=false`
  * Remove `auth` configuration option from templates - use an `auth`
    configuration block with `enabled = true` combined with `username = ...` and
    `password = ...` inside the block instead

FEATURES:

  * Add support for logging to syslog (GH-163)
  * Add `log_level` as a configuration file option
  * Add `-log-level` as a CLI option

IMPROVEMENTS:

  * Use a default retry interval of 5s (GH-190) - this value has been (and will
    remain) configurable since v0.5.0, but the default value has changed from 0
    to 5s
  * Use a service's reported address if given (GH-185, GH-186)
  * Add new `NodeAddress` field to health services to always include the node's
    address
  * Return errors up the watcher's error channel so other libraries can
    determine what to do with the error instead of swallowing it (GH-196)
  * Move SSL and authentication options into their own configuration blocks in
    the HCL
  * Add new `watch.WaitVar` for parsing Wait structs via Go's flag parsing
    library.
  * Extract logging components into their own library for sharing (GH-199)

BUG FIXES:

  * Return errors instead of nil in catalog nodes and key prefix dependencies
    (GH-192)
  * Allow Consul Template to exit when running in `once` mode and templates have
    not changed (GH-188)
  * Raise an error when specifying a non-existent option in the configuration
    file (GH-197)
  * Use an RWLock when accessing information in the Brain to improve performance
  * Improve debugging output and consistency
  * Remove unused Brain functions
  * Remove unused documentation items
  * Use the correct default values for `-ssl` and `-retry` on the CLI

## v0.6.5 (February 5, 2015)

FEATURES:

  * Add `-max-stale` to specify Consul Template may talk to non-leader Consul
    nodes if they are less than the maximum stale value (GH-183)

BUG FIXES:

  * Fix a concurrency bug in the Brain (GH-180)
  * Add a better queue-draining mechanism for templates that have a large number
    of dependencies (GH-184)

## v0.6.1 (February 2, 2015)

IMPROVEMENTS:

  * Allow watcher to use buffered channels so we do not block when multiple
    dependencies return data (GH-176)
  * Buffer results from the watcher to reduce the number of CPU cycles (GH-168
    and GH-178)

BUG FIXES:

  * Handle the case where reloading via SIGHUP would cause an error (GH-175 and
    GH-177)
  * Return errors to the template when parsing a key fails (GH-170)
  * Expand the list of possible values for keys to non-ASCII fields (the `@` is
    still a restricted character because it denotes the datacenter) (GH-170)
  * Diff missing dependencies during the template render to avoid creating
    extra watchers (GH-169)
  * Improve debugging output (GH-169)

## v0.6.0 (January 20, 2015)

FEATURES:

  * Implement n-pass evaluation (GH-64) - templates are now evaluated N+1 times
    to properly accumulate dependencies and build the graph properly

BREAKING CHANGES:

  * Remove `storeKeyPrefix` template function - it has been replaced with `ls`
    and/or `tree` and was deprecated in 0.2.0
  * Remove `Key()` from dependency interface

IMPROVEMENTS:

  * Switch to using `hashicorp/consul/api` instead of `armon/consul-api`
  * Add support for communicating with Consul via HTTPS/SSL (GH-143)
  * Add support for communicating with Consul via BasicAuth (GH-147)
  * Quiesce on a per-template basis

BUG FIXES:

  * Reduce memory footprint when running with a large number of templates by
    using a single context instead of separate template contexts for each
    template
  * Improve test coverage
  * Improve debugging output
  * Correct tag deep copy that could result in 2N-1 tags (GH-155)
  * Return an empty slice when parsing an empty JSON file
  * Update README documentation

## v0.5.1 (December 25, 2014)

BUG FIXES:

  * Parse Retry values in the config (GH-136)
  * Remove `util` package as it is a code smell and separate `Watcher` and
    `Dependency` structs and functions into their own packages for re-use
    (GH-137)

## v0.5.0 (December 19, 2014)

FEATURES:

  * Reload configuration on `SIGHUP`
  * Add `services` template function for listing all services and associated
    tags in the Consul catalog (GH-77)

BUG FIXES:

  * Do not execute the same command more than once in one run (GH-112)
  * Do not exit when Consul is unavailable (GH-103)
  * Accept configuration files as a valid option to `-config` (GH-126)
  * Accept Windows drive letters in template paths (GH-78)
  * Deep copy and sort data returned from Consul API (specifically tags)
  * Run commands even if not all templates have received data (GH-119)

IMPROVEMENTS:

  * Add support for more complex service health filtering (GH-116)
  * Add support for specifying a `-retry` interval for Consul timeouts and
    connection errors (GH-22)
  * Use official HashiCorp multierror package for errors
  * Gracefully stop watchers on interrupt
  * Add support for Go 1.4
  * Improve test coverage around retrying failures

## v0.4.0 (December 10, 2014)

FEATURES:

  * Add `env` template function for reading an environment variable in the
    current process into the template
  * Add `regexReplaceAll` template function

BUG FIXES:

  * Fix documentation examples
  * Fix `golint` and `go vet` errors
  * Fix a panic when Consul returned empty query metadata
  * Allow colons in key prefixes (`ls` and `tree` receive this by proxy)
  * Allow `parseJSON` to handle top-level JSON objects
  * Filter empty keys in `tree` and `ls` (folder nodes)

IMPROVEMENTS:

  * Merge multiple configuration template definitions when a configuration
    directory is specified

## v0.3.1 (November 24, 2014)

BUG FIXES:

  * Allow colons in key names (GH-67)
  * Fix a documentation bug in the README in the Varnish example (GH-82)
  * Attempt to render templates before starting the watcher - this fixes an
    issue where a template that declared no Consul dependencies would never be
    rendered (GH-85)
  * Update inline Go documentation for better clarity

IMPROVEMENTS:

  * Fix all issues raised by `go vet`
  * Update packaging script to fix ZSHisms and use awk for clarity

## v0.3.0 (November 13, 2014)

FEATURES:

  * Added a `Contains` method to `Service.Tags`
  * Added support for specifying a configuration directory in `-config`, in
    addition to a file
  * Added support for querying all nodes in Consul's catalog with the `nodes`
    template function

BUG FIXES:

  * Update README documentation to clarify that `service` dependencies default
    to the current datacenter if one is not explicitly given
  * Ignore empty keys that are returned from an `ls` call (GH-54)
  * When writing a file atomicly, ensure the drive is the same (GH-58)
  * Run all commands before exiting - previously if a single command failed in
    a multi-template environment, the other commands would not execute, but
    Consul Template would return

IMPROVEMENTS:

  * Added support for querying all `service` nodes by passing an additional
    parameter to `service`

## v0.2.0 (November 4, 2014)

FEATURES:

  * Added helper for decoding a result as JSON using the `parseJSON` pipe
    function
  * Added support for reading and watching changes from a file using the `file`
    template function
  * Added helper for sorting service entires by a particular tag
  * Added helper function `toLower()` for converting a string to lowercase
  * Added helper function `toTitle()` for converting a string to titlecase
  * Added helper function `toUpper()` for converting a string to uppercase
  * Added helper function `replaceAll()` for replacing occurrences of a
    substring with a new string
  * Added `tree` function for returning all key prefixes recursively
  * Added `ls` function for returning all keys in the top-level prefix (but not
    deeply nested ones)

BUG FIXES:

  * Remove prefixes from paths when querying a key prefix

IMPROVEMENTS:

  * Moved shareable functions into a util module so other libraries can benefit
  * Make Path a public field on Template
  * Added more examples and documentation to the README

DEPRECATIONS:

  * `keyPrefix` is deprecated in favor or `tree` and `ls` and will be removed in
  the next major release


## v0.1.1 (October 28, 2014)

BUG FIXES:

  * Fixed an issue where help output was displayed twice when specifying the
    `-h` flag
  * Added support for specifyiny forward slashes (`/`) in service names
  * Added support for specifying underscores (`_`) in service names
  * Added support for specifying dots (`.`) in tag names

IMPROVEMENTS:

  * Added support for Travis CI
  * Fixed numerous typographical errors
  * Added more documentation, including an FAQ in the README
  * Do not return an error when a template has no dependencies. See GH-31 for
    more background and information
  * Do not render templates if they have the same content
  * Do not execute commands if the template on disk would not be changed

## v0.1.0 (October 21, 2014)

  * Initial release
