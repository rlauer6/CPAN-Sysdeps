# Table of Contents

* [NAME](#name)
* [SYNOPSIS](#synopsis)
* [DESCRIPTION](#description)
* [COMMANDS](#commands)
  * [create-deps](#create-deps)
  * [find-deps](#find-deps)
* [OPTIONS](#options)
* [SEE ALSO](#see-also)
* [AUTHOR](#author)
* [POD ERRORS](#pod-errors)
# NAME

CPAN::Sysdeps - map CPAN modules to system library build dependencies

# SYNOPSIS

    # build and cache the mapping for the current distro
    cpan-sysdeps create-deps

    # look up what a module needs
    cpan-sysdeps find-deps Net::SSLeay

    # use a different distro or backend
    cpan-sysdeps create-deps --type fedora --distro 41

# DESCRIPTION

`CPAN::Sysdeps` answers the question "which system library do I need
to install before `cpm` can build this XS module?"

It derives the answer automatically by cross-referencing two public
indexes:

- The distro's source package index (Debian `Sources.gz`, Fedora
SRPM repodata) — which lists `Build-Depends`/`BuildRequires` for every
Perl package.
- CPAN's `02packages.details.txt.gz` — the authoritative list of
module names, used to correct casing (`DBD::Pg` not `Dbd::Pg`).

No per-module API calls.  No manually maintained lists.

# COMMANDS

## create-deps

Fetch the source package index and `02packages`, build the mapping, and
write it to the output file (default: `debian-packages.yml`).

    cpan-sysdeps create-deps [--type debian|fedora] [--distro trixie]
                              [--output packages.yml] [--verbose]

## find-deps

Look up the system library dependencies for a CPAN module.  Reads the
output file written by `create-deps`.

    cpan-sysdeps find-deps [--output packages.yml] <Module::Name>
    cpan-sysdeps find-deps --module Module::Name

Prints one library name per line, or `none` if no system deps are
required.

# OPTIONS

    --type    -t   Backend: debian (default) or fedora
    --distro  -d   Distro codename/release (default: trixie / 41)
    --output  -o   Mapping file path (default: debian-packages.yml)
    --module  -m   Module name for find-deps (alternative to positional arg)
    --verbose  -v  Show per-package resolution progress
    --help    -h   This help

# SEE ALSO

[CPAN::Sysdeps::Debian](https://metacpan.org/pod/CPAN%3A%3ASysdeps%3A%3ADebian), [CPAN::Sysdeps::Fedora](https://metacpan.org/pod/CPAN%3A%3ASysdeps%3A%3AFedora),
[CPAN::Sysdeps::Role::Backend](https://metacpan.org/pod/CPAN%3A%3ASysdeps%3A%3ARole%3A%3ABackend)

# AUTHOR

Rob Lauer <rlauer@treasurersbriefcase.com>

# POD ERRORS

Hey! **The above document had some coding errors, which are explained below:**

- Around line 139:

    Non-ASCII character seen before =encoding in '—'. Assuming UTF-8
