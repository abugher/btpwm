# Design

This document shall serve as a general overview of the project's design
concepts.

## Files

### Password Entry

These entries form a chain. One entry is one file, and this is the atomic unit
dealt with by the distribution side of the program.

Each entry contains a reference to the next entry, until the last one which is
marked as terminal.

The file name is composed of several parts. First is a fixed string identifying
this as an identity bundle for our program.

Next is a unique identifier. This is constructed by cryptographic means from
the master password and a second value, to which I will refer, probably somewhat
inaccurately, as a salt.

The salt for the first entry is a widely known fixed value. This way the first
entry in ones own identity can always be found. Each entry contains a new salt
within the encrypted text. The new salt is used to form the unique identifier
for the next entry.

Thus, entries form a chain, and a user should always be able to find his first
entry, and from there find all his other entries, creating a complete copy of
his identity, from only a copy of the program and knowledge of his master
password.

Datestamping is not quite worked out, yet.

Collisions: We'll use a good hash algorithm, or possibly just encrypt instead,
and make collisions either unlikely or impossible. It's up to the user to pick a
unique password.

It needs to be made clear to the user that using a bad master password, even for
a short while, exposes all passwords in that identity to risk, permanently.

Each password entry will be formatted as name:value pairs.

I think text will be fine, as long as there's a consistent format. (No need for
a database.) We can treat the first colon on a line as the delimiter. That lets
us allow pretty much whatever characters we want before the delimiter, for
names, and any non-line-ending character for the value. We might even implement
backslash escaping to remove the newline limitation.

The other fields should probably include site, username, and any other metadata
we like. We could conceivably allow arbitrary user-defined metadata fields.
Perhaps with a prefix, like "user-".

Side idea: We could have a "password" field, which gets whatever special
handling is called for with a password, and an alternative private key field,
which gets appropriate handling, such as never sending it over the wire.

I'm not sure it would be beneficial to step on ssh-agent's toes, though. This
could be a good way to always keep your private key available to yourself, but
we might just punt to the user when it comes to installing and using the key.

### Password Files

The password file is encrypted with the master key. Each file contains a
password entry. A separate torrent exists for each password file.

## Application

Cross platform availability is obviously desirable. For starters, we should have
Windows, Linux, Mac, Android, and iOS. Interpreted languages sound good to me,
off the cuff.

### Things we want to do:

**Store Passwords**

We want tie-in to environments where users use a lot of passwords. Basically,
that means browsers. A firefox extension might be in order.

We should also allow manual entry, of course.

**Upload Identity**

The client should regularly (or constantly) make the identity bundle available
on bittorrent, discoverable over DHT.

**Download Identities**

The client should keep a certain quorum of other identities (10 seems like a
reasonable default). The fact that clients do this out of the box makes the
previous thing work.

Just search for the fixed string we chose to identify identity bundles.

**Retrieve Specific Identity**

Given only the master password from bittorrent/DHT, we should be able to
retrieve a precise identity. See the Password Entry description for the
mechanics.

**Password Availability**

Make passwords available, given the master password. In addition to letting a
user manually invoke the program and retrieve a password, it would be nice to
feed it to autocomplete, or hand it off to a local browser password history, if
that's what the user wants. (That one's a little sketchy. Maybe off-by-default.)

A stand-alone program could just put the retrieved password on the clipboard.
I'd feel a lot better about this option if we can make it clear the clipboard
after the next paste action.

**Generate Passwords**

This is best-effort, but that should handle most situations.

We should provide controls for password restrictions. Basically, we should have
a checkbox buffet so you can build a set of restrictions to match your obstinate
website of choice.

I'd also like to see an automated diceware option.

The user should always have the option to enter a password of his choosing.

**Manage entropy**

If the program uses system entropy at all, it needs to at least evaluate it and
make sure it looks like decent random data. More likely we'll end up keeping an
application-specific entropy pool.

Anyway, if we're generating passwords, they need to be random.

## Anonymity

The design is already such that everything uploaded is encrypted, and filenames
are random. In other words, the identity bundle itself does not make it clear to
whom it belongs.

A network observer might still be able to work something out, but I'm not
worried about the ability to map one specific user to an identity bundle, given
unlimited time and resources.

I want it to be difficult to discriminate among bundles based on content. For
example, I wouldn't want someone to be able to set up a bunch of clients, and
have them refuse service (or somehow interfere with service) to bundles based on
usernames, or site names. (Think unethical government ops.)

I also wouldn't really want someone to be able to hassle clients based on the
number of password entries they maintain. It seems somewhat less likely, and
would require some special handling.

We'd have to set fixed sizes for files, and pad them out to fill the space.
Files that would normally grow beyond that size could perhaps be split into
parts not easily associated with each other.

No password list. No identity directory. Each torrent contains one file, with
one password entry. The file also contains a field with a random value. Hash
this value with the master password to get a new identifier. Search for it to
find the next entry in the chain. The last entry has a terminator value in the
daisy-chain field.

## Defense

I'd like to whole system to be intrinsically robust, hard to corrupt.

### Threats:

**flood**

Someone could generate craploads of fake identities, flood the network, and make
it hard to find enough bandwidth to shuttle around real identities. This seems
like it would take continuous resources by the attacker.

I don't think we can reasonably stop it from happening, but we can design the
system so it will recover quickly. The only way I can think to do this is with
automatic expiration of password entries.

That presents the problem that if you don't use the manager for a long time (and
it's not silently doing its thing in the background), then you sit down to a new
device, you won't be able to get your passwords unless you can find an old copy
on a device you were previously using and seed them yourself. In other words:
Once they're gone, they're gone.

On the other hand, resources are limited. Theoretically, even without an attack,
we'll eventually ...

**spoof**

If someone downloads someone else's identity, he might be able to change some
attribute to make it look newer, fill it with garbage, and seed it to
intentionally replace the legit version, thus screwing over some poor stranger.
Ideally, this shouldn't be possible.

The timestamp needs to be plaintext, so that anyone can check for updated
versions of entries. We also need to prove that this entry, including the
timestamp, was authorized by the same person who authorized any past versions
of the same entry. To me, this suggests PKI and signatures.

Each credential entry can be accompanied by two extra files:

- A unique (to this entry, but not to this version) public key.

- A signature of the entry file, verifiable with the same public key.

- A signature of the name of the file, which includes the timestamp.

Now when Bob uploads an entry, if Eve uploads a fake new version of Bob's entry,
Alice (or our program on her behalf) can easily check that the entries were not
signed with the same public key.

Alice won't know how to tell which of these entries is legitimate. If Eve
included the same public key, of course, she won't have valid signatures, which
is a dead giveaway. If she was smart enough to just generate a new key pair, she
could generate a blob of data that looks like an entry, give it a name that
indicates it was encrypted with the same master password Bob uses (even though
it wasn't), sign it with her private key, sign the name, and include the public
key. I don't see a good way to check that the same master password was used to
encrypt the entry and generate the identifier, without knowing the password.

We could check that the same key was used:

1. To encrypt the entry.
2. To sign the entry.
3. To sign the name.
...
