# Third-party licenses for the ReadyBase binary

Every third-party Go module **compiled into the released `readybase` binary**
(`cmd/readybase`), with its exact version, license, and a pointer to the full,
verbatim license text bundled in [`LICENSES/`](LICENSES/). Scope was resolved
from the binary's own dependency graph (`go list -deps ./cmd/readybase`), so
test-only and tooling dependencies are excluded. ReadyBase's own source is not
listed here.

This satisfies the redistribution terms of the MIT and BSD-3-Clause licenses
below, which require the copyright notice and permission text to travel with the
binary. The full text of each is reproduced verbatim under `LICENSES/`.

| Module | Version | License | Full text |
|--------|---------|---------|-----------|
| `github.com/dustin/go-humanize` | v1.0.1 | MIT | [LICENSES/github.com_dustin_go-humanize.txt](LICENSES/github.com_dustin_go-humanize.txt) |
| `github.com/google/uuid` | v1.6.0 | BSD-3-Clause | [LICENSES/github.com_google_uuid.txt](LICENSES/github.com_google_uuid.txt) |
| `github.com/mattn/go-isatty` | v0.0.20 | MIT | [LICENSES/github.com_mattn_go-isatty.txt](LICENSES/github.com_mattn_go-isatty.txt) |
| `github.com/ncruces/go-strftime` | v1.0.0 | MIT | [LICENSES/github.com_ncruces_go-strftime.txt](LICENSES/github.com_ncruces_go-strftime.txt) |
| `github.com/odvcencio/gotreesitter` | v0.20.2 | MIT | [LICENSES/github.com_odvcencio_gotreesitter.txt](LICENSES/github.com_odvcencio_gotreesitter.txt) |
| `github.com/remyoudompheng/bigfft` | v0.0.0-20230129092748-24d4a6f8daec | BSD-3-Clause | [LICENSES/github.com_remyoudompheng_bigfft.txt](LICENSES/github.com_remyoudompheng_bigfft.txt) |
| `golang.org/x/sys` | v0.46.0 | BSD-3-Clause | [LICENSES/golang.org_x_sys.txt](LICENSES/golang.org_x_sys.txt) (+ [PATENTS](LICENSES/golang.org_x_sys.PATENTS.txt)) |
| `modernc.org/libc` | v1.70.0 | BSD-3-Clause | [LICENSES/modernc.org_libc.txt](LICENSES/modernc.org_libc.txt) (+ [3rd-party C](LICENSES/modernc.org_libc.LICENSE-3RD-PARTY.md.txt)) |
| `modernc.org/mathutil` | v1.7.1 | BSD-3-Clause | [LICENSES/modernc.org_mathutil.txt](LICENSES/modernc.org_mathutil.txt) |
| `modernc.org/memory` | v1.11.0 | BSD-3-Clause | [LICENSES/modernc.org_memory.txt](LICENSES/modernc.org_memory.txt) (+ [Go](LICENSES/modernc.org_memory.LICENSE-GO.txt), [mmap-go](LICENSES/modernc.org_memory.LICENSE-MMAP-GO.txt), [logo](LICENSES/modernc.org_memory.LICENSE-LOGO.txt)) |
| `modernc.org/sqlite` | v1.48.2 | BSD-3-Clause | [LICENSES/modernc.org_sqlite.txt](LICENSES/modernc.org_sqlite.txt) |

_11 third-party modules._

## Notes

- **`golang.org/x/sys`** ships an additional Go patent grant (`PATENTS`),
  reproduced alongside its BSD-3-Clause license.
- **`modernc.org/libc`** is a Go port of C library code and carries an
  upstream third-party license file (`LICENSE-3RD-PARTY.md`) covering the
  original C sources; it is reproduced in full.
- **`modernc.org/memory`** bundles small amounts of vendored code under their
  own terms (the Go Authors BSD license and Evan Shaw's mmap-go BSD license),
  plus a logo attribution; all are reproduced.
- The texts under `LICENSES/` are copied verbatim from each module at the
  pinned version in `go.mod`. To regenerate after a dependency bump, re-run the
  collection against `go list -deps ./cmd/readybase`, or use
  [`go-licenses`](https://github.com/google/go-licenses):
  `go-licenses save ./cmd/readybase --save_path=LICENSES/`.

If anything here is miscredited or mislabeled, please open an issue and we will
correct it.
