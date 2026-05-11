# CPAN::Sysdeps Release Notes

## 1.0.0 - First Release

### Overview

`CPAN::Sysdeps` solves the long-standing problem of determining which
system library is required to build a given CPAN XS module. It derives
the answer automatically by cross-referencing two public package indexes,
with no manually maintained lists and no per-module API calls.

### How It Works

Two downloads, one lookup table:

1. **CPAN `02packages.details.txt.gz`** - the authoritative module name
   registry. Module names are indexed by their lowercased form, so
   `lc("Dbd::Pg")` and `lc("DBD::Pg")` both resolve to the canonical
   `DBD::Pg`. This eliminates the acronym casing problem inherent in
   converting distro package names to CPAN module names without any
   per-module API calls or manually maintained corrections.

2. **Distro source package index** - `Sources.gz` (Debian) or
   `repomd.xml` + `primary.xml.zst` (Fedora) - which lists build
   dependencies for every packaged Perl module. The system library
   entries are filtered from these and mapped to their CPAN modules.

### Commands

#### `cpan-sysdeps create-deps`

Fetches both indexes and writes a YAML mapping of system libraries to
the CPAN modules that require them:

```yaml
libssl-dev:
  - Crypt::OpenSSL::RSA
  - Net::SSLeay
  - IO::Socket::SSL

libxml2-devel:
  - XML::LibXML
  - XML::LibXSLT
```

Output filename defaults to `{type}-packages.yml` (`debian-packages.yml`
or `fedora-packages.yml`).

#### `cpan-sysdeps find-deps`

Inverts the mapping at query time to answer "what does this module need?":

```
$ cpan-sysdeps find-deps Net::SSLeay
libssl-dev
```

Returns `none` for pure-Perl modules with no system library requirements.

### Backends

| Backend | Source index | Compression | Parser |
|---|---|---|---|
| `Debian` | `Sources.gz` | gzip | Custom stanza parser |
| `Fedora` | `repomd.xml` → `primary.xml.zst` | Zstandard | `XML::Twig` streaming |

The Fedora backend performs a two-step fetch: `repomd.xml` is parsed to
locate the checksum-prefixed `primary.xml.zst`, which is then fetched
and stream-parsed with `XML::Twig` to keep the 49MB uncompressed payload
from loading into memory all at once.

New backends implement `CPAN::Sysdeps::Role::Backend`.

### Architecture

- `CLI::Simple` 2.x role-based command dispatch
- `Role::Tiny` backend interface with auto-detection of implementation status
- `Log::Log4perl` throughout - no bare `print STDERR`
- `Readonly` for all URL and version constants
- `YAML::Tiny` for output

### Known Issues

**`Fedora.pm` - operator precedence in arch filter**

```perl
return if !$arch eq 'src';   # wrong
return if $arch ne 'src';    # correct
```

`!$arch eq 'src'` is parsed as `(!$arch) eq 'src'`, which is always
false. The filter never fires. Benign in practice because the Fedora
SRPM `primary.xml` only contains source packages, so all entries have
`arch=src`. Fixed in 1.0.1.

**`Fedora.pm` - `next` in twig handler closure**

```perl
next if !@deps;    # wrong inside anonymous sub
return if !@deps;  # correct
```

`next` inside an anonymous sub is not a loop `next`. Correct behaviour
by accident due to `XML::Twig`'s internal call mechanism, but not valid
Perl. Fixed in 1.0.1.

**`CreateDeps.pm` - `grep` with side effects in dedup**

```perl
$mapping{$lib} = [ sort grep { !$seen{$_}++ } @{ $mapping{$lib} } ];
```

Side-effect `grep` will be flagged by `perlcritic`. Replace with an
explicit dedup loop in 1.0.1.

### Dependencies

**Core:**
- `HTTP::Tiny`
- `IO::Uncompress::Gunzip`

**Non-core (Debian backend):**
- `YAML::Tiny`
- `Role::Tiny`
- `Readonly`
- `CLI::Simple` 2.x
- `Log::Log4perl`

**Additional (Fedora backend):**
- `IO::Uncompress::UnZstd` (`IO-Compress-Zstd` on CPAN)
- `XML::Twig`

### Roadmap

- **Override table** - `share/overrides.yml` for known package-to-module
  name divergences (e.g. `libdbd-sqlite3-perl` → `DBD::SQLite`)
- **`--cpanfile` input for `find-deps`** - scan a whole dependency list
  at once and produce a ready-to-use `debian-packages` file for the
  Lambda builder
- **Debian `bookworm` / Fedora version defaults** - make defaults
  configurable without code changes
