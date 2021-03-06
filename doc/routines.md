## create identity

Accept a master password and generate an identity file structure. This is
initially placed in a staging area, isolated from torrent area.

## accept credentials

Provide a mechanism for external entities (browser plugin, eg) to pass a set of
credentials to our program.

This routine should create a data structure that can be passed around to other
functions.

## record credentials

This should accept the data structure produced by "accept credentials", convert
it to the expected on-disk format, encrypt it, and write it as a password entry.

Only valid for identities in the staging area.

## delete credentials

Remove a password entry from an identity.

Only valid for identities in the staging area.

## upload identity

Make available the identity we just generated. I (currently) envision an
always-running torrent thread/process, to which we'll IPC the identity we want
to make available.

A stored identity must be specified. A torrent file is generated if necessary.
The files are moved from staging to the torrent area. The torrent file is handed
to the torrent process. The torrent is made available on DHT. Other clients
should find it automatically.

## update entry to net

Upload a new version of a previously uploaded identity. This implicitly expires
the old version.

## gather identities

Search DHT for identity files and (randomly? intelligently?) select some to
download. Pass the torrent to the torrent process.

This should be called on a regular basis, to find new identities quickly. (It
would be nice if a new user doesn't have to wait all day to get his identity
uploaded.)

Depending on user base size, every desktop system using this function once in a
24 hour period might be sufficient. Once an hour still seems quite conservative
for resources.

## update entry from net

This should do a search for all versions of a specified entry, and download the
newest if it's newer than the current version.

## download my identity

Search for the identity associated with a given master password, and download
it. It may make sense to implement this as a mode of operation for "download
identity".

## retrieve credentials

Retrieve all password entries from an identity, using the master password. They
get sucked into some data structure, which can be kept around in memory as
needed or appropriate.

This is where we start to need to think about certain aspects of design. Right
now, I'm thinking daemon mode, where this program keeps these plain text data
structures sitting around, and makes them available to authorized programs.
There's the thing, though...

The simplest way to declare a program "authorized" is to require it to supply
the master password. (I'm picturing a browser plugin that prompts for and then
keeps in memory the master password, although that might also be a bit unwise.)
At that point, why daemonize? This program could be invoked from a cold start,
be given the master password, and recover the requested passwords from storage.

Alternately, we could daemonize and make the passwords available to any program
running as the same user.

We'll need to work out the API before we really get anywhere.

## torrent

"To torrent", as a verb, that is. This amounts to a feature-poor torrent client.

This is where we could really become a nuisance to the user. Don't:

- hog the bandwidth
- use up limited (expensive) data
- drain the battery

The front-end should probably provide some options at initialization time. I see
modes of operation:

### selfish

Seems sensible for cell phones. Upload and download files we care about, then
exit. Save that battery.

In this mode, we don't bother downloading other people's identities, because we
won't seed them anyway.

### seeder

Allocate a (relatively) big ol' chunk of storage, and download as many
strangers' identities as we can. Make them all available all the time.

A few nice guys running this mode on desktops could probably support thousands
of mobile clients.


## generate password

This may not be a function of the service code. This might go better with the
UI element. (Browser plugin.)
