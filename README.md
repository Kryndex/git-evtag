# git-evtag

`git-evtag` can be used as a drop-in replacement for `git-tag -s`.  It
will generate a strong checksum over the commit, tree, and blobs it
references.

Git mailing list thread: http://permalink.gmane.org/gmane.comp.version-control.git/264533

### Using git-evtag

Create a new `v2015.10` tag, covering the `HEAD` revision with GPG
signature and `Git-EVTag-SHA512`:

```
$ git-evtag v2015.10
 ( type your tag message, note a Git-EVTag-SHA512 line in the message )
$ git show v2015.10
 ( Note signature covered by PGP signature )
```

Verify a tag:

```
$ git-evtag --verify v2015.10
gpg: Signature made Sun 28 Jun 2015 10:49:11 AM EDT
gpg:                using RSA key 0xDC45FD5921C13F0B
gpg: Good signature from "Colin Walters <walters@redhat.com>" [ultimate]
gpg:                 aka "Colin Walters <walters@verbum.org>" [ultimate]
Primary key fingerprint: 1CEC 7A9D F7DA 85AB EF84  3DC0 A866 D7CC AE08 7291
     Subkey fingerprint: AB92 8A9C F8DD 0629 09C3  7BBD DC45 FD59 21C1 3F0B
Successfully verified: Git-EVTag-SHA512: b05f10f9adb0eff352d90938588834508d33fdfcedbcfc332999ee397efa321d1f49a539f1b82f024111a281c1f441002e7f536b06eb04d41857b01636f6f268
```

### Replacing tarballs

This is similar to what project distributors often accomplish by using
`git archive` or `make dist` to generate a tarball, and then
checksumming that, and usually providing a GPG signature.

The advantage `git-evtag` has over this is that no out of band
distribution mechanism is necessary - git already supports GPG
signatures, and with this project, we now have a single checksum over
the complete source objects for the target commit (+ trees + blobs).

(And if you want to avoid downloading the entire history, that's what
 `git clone --depth=1` is for.)

### Git and SHA1

Git uses a modified Merkle tree with SHA1, which means that if an
attacker managed to create a SHA1 collision for a source file object,
it would affect all revisions and checkouts.

In contrast, `Git-EVTag-SHA512` covers the entirety of a single
commit.  The algorithm is:

 - Add commit object to checksum
 - Add commit root tree to checksum
 - Walk tree recursively, checksumming each tree and blob referenced

Unlike `git archive`, which might change format in the future and
break checksums, the core git object format is fixed.

This strong checksum then helps obviate the SHA1 weakness concerns of
git for source distribution.  The author of this tool believes that
today, GPG signed git tags are fairly secure, especially if one is
careful to ensure transport integrity (e.g. pinned TLS certificates
from the origin).

But while at the time of this writing, no public SHA1 collision is
known, there are attacks against reduced round SHA1.  We expect git
repositories to be used for many, many years to come.  It makes a lot
of sense to take additional steps now to add security.

And most importantly, it's quite inexpensive to compute an additional
(strong SHA512) checksum at `git tag` time that covers all of a
commit's contents at once.  The cost/benefit of other approaches such
as forking git with a different checksum algorithm don't appear to be
worth it.
