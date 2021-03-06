## A password entry

**is formatted information**

A password entry file contains a password entry. The "entry" is the block of
formatted data, not the file itself.

**consists of lines**

Each of which is taken to be a key:value pair, where the first colon is taken to
be a field separator.

## A key

**approved characters**

This definitely will not include the colon, and almost as certainly will not
include the newline. The character whitelist may be expanded in the future, but
for the moment will be defined as:

- upper
- lower
- digits
- hyphen
- underscore (special meaning)

**may have a prefix**

A prefix is simply a string reserved to indicate something about a key. The
only prefix currently defined is "user_". This merely indicates that a key was
created and set by a user. If another program has heavy interactions with this
one, a prefix might be established to simulate a namespace for the settings
saved by that other program.

## Defined keys

**login-url** | Page where credentials should be applied.

**username** | The username.

**password** | Password to be used with username.

A complete password entry consists at a minimum of:

- A context field. (Login URL, SSH host, etc.)
- An identity field. (Username, address, etc.)
- An authentication field. (Password, private key, etc.)
