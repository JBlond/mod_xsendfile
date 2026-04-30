# Changelog

## [1.0.0] - 2026-04-30

### Added
- Support for `X-SENDFILE-TEMPORARY` (serve a temporary file and delete it after sending, when enabled/allowed).
- New configuration directives:
  - `XSendFileUnescape` (URL-decode the header value).
  - `XSendFileUnsetContentEncoding` (optionally unset `Content-Encoding` for the replaced response).
- Added `LICENSE` (Apache License 2.0).
- Added `README.md` (replacing the old HTML documentation).
- Added `mod_xsendfile.spec` (RPM spec).
- Added Apache 2.4 Visual Studio project (`mod_xsendfile_24.vcproj`).

### Changed
- Bumped module version to 1.0.
- Internal configuration/path handling updated to support per-path permissions related to temporary file deletion.
- Updated copyright years and contributor credits.

### Removed
- Removed legacy `docs/Readme.html`.
- Removed older Visual Studio project files (`mod_xsendfile_20.vcproj`, `mod_xsendfile_22.vcproj`).

## [0.12] - 2023-02-01

### Changed
- Now incorrect headers will be dropped early.

## [0.11.1]
### Fixed
- Fixed some documentation bugs.
### Changed
- Built win32 binaries against latest httpd using MSVC9.
- Updated MSVC project files.

## [0.11] - 2023-01-15

### Fixed
- Fixed large file support.

## [0.10] - 2023-01-01

### Added
- New configuration directive: `XSendFileIgnoreEtag`.
- New configuration directive: `XSendFileIgnoreLastModified`.
- New configuration directive: `XSendFilePath`.

### Changed
- Won't override Etag/Last-Modified if already set.
- Improved header handling for FastCGI/CGI output (removing duplicate headers).

### Removed
- Removed configuration directive: `XSendFileAllowAbove` (use `XSendFilePath` instead).

## [0.9] - 2022-12-01

### Added
- New configuration directive: `XSendFileAllowAbove`.
- Initial FastCGI/CGI support.
- Filter only added when needed.

## [0.8] - 2022-11-01

### Added
- Initial public release.
