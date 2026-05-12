# CPAN::Sysdeps Release Notes

## 1.0.1

### Bug Fixes

**YAML output format corrected.** The 1.0.0 packages files used an incorrect
YAML scalar format for module lists. Modules are now properly emitted as array
elements:

```yaml
# 1.0.0 (wrong)
libssl-dev:
- Net::SSLeay
- IO::Socket::SSL

# 1.0.1 (correct)
libssl-dev:
  - Net::SSLeay
  - IO::Socket::SSL
```

**`canonical_module` no longer returns incorrect guesses.** Modules that could
not be resolved via the `02packages` lookup now return `undef` and are excluded
from the packages output file entirely. Previously the uncorrected guess (e.g.
`Dbd::Sqlite3`) was written to the packages file.

**`libsqlite3-dev` corrected to `DBD::SQLite`.** The `lc()` normalization
correctly matches `Dbd::Sqlite3` to `DBD::SQLite` at lookup time, but the
Debian package name suffix (`3`) caused the initial guess to diverge. Corrected
via the new override mechanism and shipped in `debian-overrides.yml`.

**Unresolvable entries removed from packages files.** Several entries that
could not be confirmed against `02packages` and have no override have been
removed from the distributed `debian-packages.yml` and `fedora-packages.yml`.
These now appear in the corresponding `*-overrides-todo.yml` files for
community review.

**`grep` side-effect dedup replaced.** The `perlcritic`-flagged
`grep { !$seen{$_}++ }` pattern in `CreateDeps` has been replaced with a clean
`map { $_ => 1 }` hash-based deduplication.

---

### New Features

**Override system.** A new three-file convention handles modules that cannot be
auto-resolved:

| File | Purpose |
|---|---|
| `<type>-packages.yml` | Auto-generated output - do not edit |
| `<type>-overrides.yml` | User-maintained corrections, merged in on each run |
| `<type>-overrides-todo.yml` | Staged unresolved entries for review |

All three files share the same `lib => [modules]` format. The workflow:

1. Run `create-deps` - generates packages file and todo file
2. Review `<type>-overrides-todo.yml` for unresolved entries
3. Add corrected entries to `<type>-overrides.yml` in the same format
4. Rerun `create-deps` - overrides merge in, resolved entries disappear from todo

Override files are looked up in the current working directory first, then fall
back to the distribution's `share/` directory, matching the existing behaviour
for packages files.

**Type-prefixed file naming.** All three files are prefixed by backend type
(`debian-` or `fedora-`), eliminating ambiguity when both backends are used
in the same directory.

**Logger passed to backend classes.** `Backend.pm` now owns the `new`
constructor and `get_logger` accessor, requiring a `logger` argument at
construction. Individual backend classes (`Debian.pm`, `Fedora.pm`) no longer
define their own `new`. The main `init` method passes the CLI logger to the
backend, giving all backends consistent access to structured logging.

**`fedora-overrides.yml` shipped.** An empty `fedora-overrides.yml` is now
included in the distribution as a placeholder for community-contributed
corrections.

---

### Removed Entries

The following entries were removed from the distributed packages files as
unresolvable without overrides. They appear in the `*-overrides-todo.yml`
files shipped with this release:

**Debian:** `libapr1-dev` (Apache::Ssslookup), `libbam-dev` (Bio::Samtools),
`libfreecontact-dev` (Freecontact), `liblmdb-dev` (Lmdb::File),
`libldap2-dev` (Mozilla::Ldap), `libnss3-dev` (Mozilla::Ldap),
`librostlab-*-dev` (Rg::Blast::Parser), `libsolv-dev` (Bssolv),
`libtokyocabinet-dev` (Tokyocabinet), `libzerg0-dev` (Zerg)

**Fedora:** `flow-tools-devel` (Cflow), `grpc-devel` (Grpc::Xs),
`mecab-devel` (Mecab), `libsolv-devel` (BSSolv), `libpq-devel`
(Pgsql_perl5), `gdbm-devel` (Verilog::Perl)

---

## 1.0.0 - Initial Release

See `release-notes-1.0.0.md`.
