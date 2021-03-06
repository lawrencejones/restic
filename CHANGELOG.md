Changelog for restic 0.8.1 (2017-12-27)
=======================================

The following sections list the changes in restic 0.8.1 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Fix #1457: Improve s3 backend with DigitalOcean Spaces
 * Fix #1454: Correct cache dir location for Windows and Darwin
 * Fix #1459: Disable handling SIGPIPE
 * Chg #1452: Do not save atime by default
 * Enh #1436: Add code to detect old cache directories
 * Enh #1439: Improve cancellation logic
 * Enh #11: Add the `diff` command

Details
-------

 * Bugfix #1457: Improve s3 backend with DigitalOcean Spaces

   https://github.com/restic/restic/issues/1457
   https://github.com/restic/restic/pull/1459

 * Bugfix #1454: Correct cache dir location for Windows and Darwin

   The cache directory on Windows and Darwin was not correct, instead the directory `.cache` was
   used.

   https://github.com/restic/restic/pull/1454

 * Bugfix #1459: Disable handling SIGPIPE

   We've disabled handling SIGPIPE again. Turns out, writing to broken TCP connections also
   raised SIGPIPE, so restic exits on the first write to a broken connection. Instead, restic
   should retry the request.

   https://github.com/restic/restic/issues/1457
   https://github.com/restic/restic/issues/1466
   https://github.com/restic/restic/pull/1459

 * Change #1452: Do not save atime by default

   By default, the access time for files and dirs is not saved any more. It is not possible to
   reliably disable updating the access time during a backup, so for the next backup the access
   time is different again. This means a lot of metadata is saved. If you want to save the access time
   anyway, pass `--with-atime` to the `backup` command.

   https://github.com/restic/restic/pull/1452

 * Enhancement #1436: Add code to detect old cache directories

   We've added code to detect old cache directories of repositories that haven't been used in a
   long time, restic now prints a note when it detects that such dirs exist. Also, the option
   `--cleanup-cache` was added to automatically remove such directories. That's not a problem
   because the cache will be rebuild once a repo is accessed again.

   https://github.com/restic/restic/pull/1436

 * Enhancement #1439: Improve cancellation logic

   The cancellation logic was improved, restic can now shut down cleanly when requested to do so
   (e.g. via ctrl+c).

   https://github.com/restic/restic/pull/1439

 * Enhancement #11: Add the `diff` command

   The command `diff` was added, it allows comparing two snapshots and listing all differences.

   https://github.com/restic/restic/issues/11
   https://github.com/restic/restic/issues/1460
   https://github.com/restic/restic/pull/1462


Changelog for restic 0.8.0 (2017-11-26)
=======================================

The following sections list the changes in restic 0.8.0 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Sec #1445: Prevent writing outside the target directory during restore
 * Fix #1256: Re-enable workaround for S3 backend
 * Fix #1291: Reuse backend TCP connections to BackBlaze B2
 * Fix #1317: Run prune when `forget --prune` is called with just snapshot IDs
 * Fix #1437: Remove implicit path `/restic` for the s3 backend
 * Enh #1102: Add subdirectory `ids` to fuse mount
 * Enh #1114: Add `--cacert` to specify TLS certificates to check against
 * Enh #1216: Add upload/download limiting
 * Enh #1271: Cache results for excludes for `backup`
 * Enh #1274: Add `generate` command, replaces `manpage` and `autocomplete`
 * Enh #1367: Allow comments in files read from via `--file-from`
 * Enh #448: Sftp backend prompts for password
 * Enh #510: Add `dump` command
 * Enh #1040: Add local metadata cache
 * Enh #1249: Add `latest` symlink in fuse mount
 * Enh #1269: Add `--compact` to `forget` command
 * Enh #1281: Google Cloud Storage backend needs less permissions
 * Enh #1319: Make `check` print `no errors found` explicitly
 * Enh #1353: Retry failed backend requests

Details
-------

 * Security #1445: Prevent writing outside the target directory during restore

   A vulnerability was found in the restic restorer, which allowed attackers in special
   circumstances to restore files to a location outside of the target directory. Due to the
   circumstances we estimate this to be a low-risk vulnerability, but urge all users to upgrade to
   the latest version of restic.

   Exploiting the vulnerability requires a Linux/Unix system which saves backups via restic and
   a Windows systems which restores files from the repo. In addition, the attackers need to be able
   to create create files with arbitrary names which are then saved to the restic repo. For
   example, by creating a file named "..\test.txt" (which is a perfectly legal filename on Linux)
   and restoring a snapshot containing this file on Windows, it would be written to the parent of
   the target directory.

   We'd like to thank Tyler Spivey for reporting this responsibly!

   https://github.com/restic/restic/pull/1445

 * Bugfix #1256: Re-enable workaround for S3 backend

   We've re-enabled a workaround for `minio-go` (the library we're using to access s3 backends),
   this reduces memory usage.

   https://github.com/restic/restic/issues/1256
   https://github.com/restic/restic/pull/1267

 * Bugfix #1291: Reuse backend TCP connections to BackBlaze B2

   A bug was discovered in the library we're using to access Backblaze, it now reuses already
   established TCP connections which should be a lot faster and not cause network failures any
   more.

   https://github.com/restic/restic/issues/1291
   https://github.com/restic/restic/pull/1301

 * Bugfix #1317: Run prune when `forget --prune` is called with just snapshot IDs

   A bug in the `forget` command caused `prune` not to be run when `--prune` was specified without a
   policy, e.g. when only snapshot IDs that should be forgotten are listed manually.

   https://github.com/restic/restic/pull/1317

 * Bugfix #1437: Remove implicit path `/restic` for the s3 backend

   The s3 backend used the subdir `restic` within a bucket if no explicit path after the bucket name
   was specified. Since this version, restic does not use this default path any more. If you
   created a repo on s3 in a bucket without specifying a path within the bucket, you need to add
   `/restic` at the end of the repository specification to access your repo:
   `s3:s3.amazonaws.com/bucket/restic`

   https://github.com/restic/restic/issues/1292
   https://github.com/restic/restic/pull/1437

 * Enhancement #1102: Add subdirectory `ids` to fuse mount

   The fuse mount now has an `ids` subdirectory which contains the snapshots below their (short)
   IDs.

   https://github.com/restic/restic/issues/1102
   https://github.com/restic/restic/pull/1299
   https://github.com/restic/restic/pull/1320

 * Enhancement #1114: Add `--cacert` to specify TLS certificates to check against

   We've added the `--cacert` option which can be used to pass one (or more) CA certificates to
   restic. These are used in addition to the system CA certificates to verify HTTPS certificates
   (e.g. for the REST backend).

   https://github.com/restic/restic/issues/1114
   https://github.com/restic/restic/pull/1276

 * Enhancement #1216: Add upload/download limiting

   We've added support for rate limiting through `--limit-upload` and `--limit-download`
   flags.

   https://github.com/restic/restic/issues/1216
   https://github.com/restic/restic/pull/1336
   https://github.com/restic/restic/pull/1358

 * Enhancement #1271: Cache results for excludes for `backup`

   The `backup` command now caches the result of excludes for a directory.

   https://github.com/restic/restic/issues/1271
   https://github.com/restic/restic/pull/1326

 * Enhancement #1274: Add `generate` command, replaces `manpage` and `autocomplete`

   The `generate` command has been added, which replaces the now removed commands `manpage` and
   `autocomplete`. This release of restic contains the most recent manpages in `doc/man` and the
   auto-completion files for bash and zsh in `doc/bash-completion.sh` and
   `doc/zsh-completion.zsh`

   https://github.com/restic/restic/issues/1274
   https://github.com/restic/restic/pull/1282

 * Enhancement #1367: Allow comments in files read from via `--file-from`

   When the list of files/dirs to be saved is read from a file with `--files-from`, comment lines
   (starting with `#`) are now ignored.

   https://github.com/restic/restic/issues/1367
   https://github.com/restic/restic/pull/1368

 * Enhancement #448: Sftp backend prompts for password

   The sftp backend now prompts for the password if a password is necessary for login.

   https://github.com/restic/restic/issues/448
   https://github.com/restic/restic/pull/1270

 * Enhancement #510: Add `dump` command

   We've added the `dump` command which prints a file from a snapshot to stdout. This can e.g. be
   used to restore files read with `backup --stdin`.

   https://github.com/restic/restic/issues/510
   https://github.com/restic/restic/pull/1346

 * Enhancement #1040: Add local metadata cache

   We've added a local cache for metadata so that restic doesn't need to load all metadata
   (snapshots, indexes, ...) from the repo each time it starts. By default the cache is active, but
   there's a new global option `--no-cache` that can be used to disable the cache. By deafult, the
   cache a standard cache folder for the OS, which can be overridden with `--cache-dir`. The cache
   will automatically populate, indexes and snapshots are saved as they are loaded. Cache
   directories for repos that haven't been used recently can automatically be removed by restic
   with the `--cleanup-cache` option.

   A related change was to by default create pack files in the repo that contain either data or
   metadata, not both mixed together. This allows easy caching of only the metadata files. The
   next run of `restic prune` will untangle mixed files automatically.

   https://github.com/restic/restic/issues/29
   https://github.com/restic/restic/issues/738
   https://github.com/restic/restic/issues/282
   https://github.com/restic/restic/pull/1040
   https://github.com/restic/restic/pull/1287
   https://github.com/restic/restic/pull/1436
   https://github.com/restic/restic/pull/1265

 * Enhancement #1249: Add `latest` symlink in fuse mount

   The directory structure in the fuse mount now exposes a symlink `latest` which points to the
   latest snapshot in that particular directory.

   https://github.com/restic/restic/pull/1249

 * Enhancement #1269: Add `--compact` to `forget` command

   The option `--compact` was added to the `forget` command to provide the same compact view as the
   `snapshots` command.

   https://github.com/restic/restic/pull/1269

 * Enhancement #1281: Google Cloud Storage backend needs less permissions

   The Google Cloud Storage backend no longer requires the service account to have the
   `storage.buckets.get` permission ("Storage Admin" role) in `restic init` if the bucket
   already exists.

   https://github.com/restic/restic/pull/1281

 * Enhancement #1319: Make `check` print `no errors found` explicitly

   The `check` command now explicetly prints `No errors were found` when no errors could be found.

   https://github.com/restic/restic/issues/1303
   https://github.com/restic/restic/pull/1319

 * Enhancement #1353: Retry failed backend requests

   https://github.com/restic/restic/pull/1353


Changelog for restic 0.7.3 (2017-09-20)
=======================================

The following sections list the changes in restic 0.7.3 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Fix #1246: List all files stored in Google Cloud Storage

Details
-------

 * Bugfix #1246: List all files stored in Google Cloud Storage

   For large backups stored in Google Cloud Storage, the `prune` command fails because listing
   only returns the first 1000 files. This has been corrected, no data is lost in the process. In
   addition, a plausibility check was added to `prune`.

   https://github.com/restic/restic/issues/1246
   https://github.com/restic/restic/pull/1247


Changelog for restic 0.7.2 (2017-09-13)
=======================================

The following sections list the changes in restic 0.7.2 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Fix #1167: Do not create a local repo unless `init` is used
 * Fix #1164: Make the `key remove` command behave as documented
 * Fix #1191: Make sure to write profiling files on interrupt
 * Enh #1132: Make `key` command always prompt for a password
 * Enh #1179: Resolve name conflicts, append a counter
 * Enh #1218: Add `--compact` to `snapshots` command
 * Enh #317: Add `--exclude-caches` and `--exclude-if-present`
 * Enh #697: Automatically generate man pages for all restic commands
 * Enh #1044: Improve `restore`, do not traverse/load excluded directories
 * Enh #1061: Add Dockerfile and official Docker image
 * Enh #1126: Use the standard Go git repository layout, use `dep` for vendoring
 * Enh #1134: Add support for storing backups on Google Cloud Storage
 * Enh #1144: Properly report errors when reading files with exclude patterns.
 * Enh #1149: Add support for storing backups on Microsoft Azure Blob Storage
 * Enh #1196: Add `--group-by` to `forget` command for flexible grouping
 * Enh #1203: Print stats on all BSD systems when SIGINFO (ctrl+t) is received
 * Enh #1205: Allow specifying time/date for a backup with `--time`

Details
-------

 * Bugfix #1167: Do not create a local repo unless `init` is used

   When a restic command other than `init` is used with a local repository and the repository
   directory does not exist, restic creates the directory structure. That's an error, only the
   `init` command should create the dir.

   https://github.com/restic/restic/issues/1167
   https://github.com/restic/restic/pull/1182

 * Bugfix #1164: Make the `key remove` command behave as documented

   https://github.com/restic/restic/pull/1164

 * Bugfix #1191: Make sure to write profiling files on interrupt

   Since a few releases restic had the ability to write profiling files for memory and CPU usage
   when `debug` is enabled. It was discovered that when restic is interrupted (ctrl+c is
   pressed), the proper shutdown hook is not run. This is now corrected.

   https://github.com/restic/restic/pull/1191

 * Enhancement #1132: Make `key` command always prompt for a password

   The `key` command now prompts for a password even if the original password to access a repo has
   been specified via the `RESTIC_PASSWORD` environment variable or a password file.

   https://github.com/restic/restic/issues/1132
   https://github.com/restic/restic/pull/1133

 * Enhancement #1179: Resolve name conflicts, append a counter

   https://github.com/restic/restic/issues/1179
   https://github.com/restic/restic/pull/1209

 * Enhancement #1218: Add `--compact` to `snapshots` command

   The option `--compact` was added to the `snapshots` command to get a better overview of the
   snapshots in a repo. It limits each snapshot to a single line.

   https://github.com/restic/restic/issues/1218
   https://github.com/restic/restic/pull/1223

 * Enhancement #317: Add `--exclude-caches` and `--exclude-if-present`

   A new option `--exclude-caches` was added that allows excluding cache directories (that are
   tagged as such). This is a special case of a more generic option `--exclude-if-present` which
   excludes a directory if a file with a specific name (and contents) is present.

   https://github.com/restic/restic/issues/317
   https://github.com/restic/restic/pull/1170
   https://github.com/restic/restic/pull/1224

 * Enhancement #697: Automatically generate man pages for all restic commands

   https://github.com/restic/restic/issues/697
   https://github.com/restic/restic/pull/1147

 * Enhancement #1044: Improve `restore`, do not traverse/load excluded directories

   https://github.com/restic/restic/pull/1044

 * Enhancement #1061: Add Dockerfile and official Docker image

   https://github.com/restic/restic/pull/1061

 * Enhancement #1126: Use the standard Go git repository layout, use `dep` for vendoring

   The git repository layout was changed to resemble the layout typically used in Go projects,
   we're not using `gb` for building restic any more and vendoring the dependencies is now taken
   care of by `dep`.

   https://github.com/restic/restic/pull/1126

 * Enhancement #1134: Add support for storing backups on Google Cloud Storage

   https://github.com/restic/restic/issues/211
   https://github.com/restic/restic/pull/1134
   https://github.com/restic/restic/pull/1052

 * Enhancement #1144: Properly report errors when reading files with exclude patterns.

   https://github.com/restic/restic/pull/1144

 * Enhancement #1149: Add support for storing backups on Microsoft Azure Blob Storage

   The library we're using to access the service requires Go 1.8, so restic now needs at least Go
   1.8.

   https://github.com/restic/restic/issues/609
   https://github.com/restic/restic/pull/1149
   https://github.com/restic/restic/pull/1059

 * Enhancement #1196: Add `--group-by` to `forget` command for flexible grouping

   https://github.com/restic/restic/pull/1196

 * Enhancement #1203: Print stats on all BSD systems when SIGINFO (ctrl+t) is received

   https://github.com/restic/restic/pull/1203
   https://github.com/restic/restic/pull/1082

 * Enhancement #1205: Allow specifying time/date for a backup with `--time`

   https://github.com/restic/restic/pull/1205


Changelog for restic 0.7.1 (2017-07-22)
=======================================

The following sections list the changes in restic 0.7.1 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Fix #1115: Fix `prune`, only include existing files in indexes
 * Enh #1055: Create subdirs below `data/` for local/sftp backends
 * Enh #1067: Allow loading credentials for s3 from IAM
 * Enh #1073: Add `migrate` cmd to migrate from `s3legacy` to `default` layout
 * Enh #1081: Clarify semantic for `--tasg` for the `forget` command
 * Enh #1080: Ignore chmod() errors on filesystems which do not support it
 * Enh #1082: Print stats on SIGINFO on Darwin and FreeBSD (ctrl+t)

Details
-------

 * Bugfix #1115: Fix `prune`, only include existing files in indexes

   A bug was found (and corrected) in the index rebuilding after prune, which led to indexes which
   include blobs that were not present in the repo any more. There were already checks in place
   which detected this situation and aborted with an error message. A new run of either `prune` or
   `rebuild-index` corrected the index files. This is now fixed and a test has been added to detect
   this.

   https://github.com/restic/restic/pull/1115

 * Enhancement #1055: Create subdirs below `data/` for local/sftp backends

   The local and sftp backends now create the subdirs below `data/` on open/init. This way, restic
   makes sure that they always exist. This is connected to an issue for the sftp server:

   https://github.com/restic/restic/issues/1055
   https://github.com/restic/restic/pull/1077
   https://github.com/restic/restic/pull/1105
   https://github.com/restic/rest-server/pull/11#issuecomment-309879710

 * Enhancement #1067: Allow loading credentials for s3 from IAM

   When no S3 credentials are specified in the environment variables, restic now tries to load
   credentials from an IAM instance profile when the s3 backend is used.

   https://github.com/restic/restic/issues/1067
   https://github.com/restic/restic/pull/1086

 * Enhancement #1073: Add `migrate` cmd to migrate from `s3legacy` to `default` layout

   The `migrate` command for chaning the `s3legacy` layout to the `default` layout for s3
   backends has been improved: It can now be restarted with `restic migrate --force s3_layout`
   and automatically retries operations on error.

   https://github.com/restic/restic/issues/1073
   https://github.com/restic/restic/pull/1075

 * Enhancement #1081: Clarify semantic for `--tasg` for the `forget` command

   https://github.com/restic/restic/issues/1081
   https://github.com/restic/restic/pull/1090

 * Enhancement #1080: Ignore chmod() errors on filesystems which do not support it

   https://github.com/restic/restic/pull/1080
   https://github.com/restic/restic/pull/1112

 * Enhancement #1082: Print stats on SIGINFO on Darwin and FreeBSD (ctrl+t)

   https://github.com/restic/restic/pull/1082


Changelog for restic 0.7.0 (2017-07-01)
=======================================

The following sections list the changes in restic 0.7.0 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Fix #1013: Switch back to using the high-level minio-go API for s3
 * Fix #965: Switch to `default` repo layout for the s3 backend
 * Enh #1021: Detect invalid backend name and print error
 * Enh #1029: Remove invalid pack files when `prune` is run
 * Enh #512: Add Backblaze B2 backend
 * Enh #636: Add dirs `tags` and `hosts` to fuse mount
 * Enh #989: Improve performance of the `find` command
 * Enh #975: Add new backend for OpenStack Swift
 * Enh #998: Improve performance of the fuse mount

Details
-------

 * Bugfix #1013: Switch back to using the high-level minio-go API for s3

   For the s3 backend we're back to using the high-level API the s3 client library for uploading
   data, a few users reported dropped connections (which the library will automatically retry
   now).

   https://github.com/restic/restic/issues/1013
   https://github.com/restic/restic/issues/1023
   https://github.com/restic/restic/pull/1025

 * Bugfix #965: Switch to `default` repo layout for the s3 backend

   The default layout for the s3 backend is now `default` (instead of `s3legacy`). Also, there's a
   new `migrate` command to convert an existing repo, it can be run like this: `restic migrate
   s3_layout`

   https://github.com/restic/restic/issues/965
   https://github.com/restic/restic/pull/1004

 * Enhancement #1021: Detect invalid backend name and print error

   Restic now tries to detect when an invalid/unknown backend is used and returns an error
   message.

   https://github.com/restic/restic/issues/1021
   https://github.com/restic/restic/pull/1070

 * Enhancement #1029: Remove invalid pack files when `prune` is run

   The `prune` command has been improved and will now remove invalid pack files, for example files
   that have not been uploaded completely because a backup was interrupted.

   https://github.com/restic/restic/issues/1029
   https://github.com/restic/restic/pull/1036

 * Enhancement #512: Add Backblaze B2 backend

   https://github.com/restic/restic/issues/512
   https://github.com/restic/restic/pull/978

 * Enhancement #636: Add dirs `tags` and `hosts` to fuse mount

   The fuse mount now has two more directories: `tags` contains a subdir for each tag, which in turn
   contains only the snapshots that have this tag. The subdir `hosts` contains a subdir for each
   host that has a snapshot, and the subdir contains the snapshots for that host.

   https://github.com/restic/restic/issues/636
   https://github.com/restic/restic/pull/1050

 * Enhancement #989: Improve performance of the `find` command

   Improved performance for the `find` command: Restic recognizes paths it has already checked
   for the files in question, so the number of backend requests is reduced a lot.

   https://github.com/restic/restic/issues/989
   https://github.com/restic/restic/pull/993

 * Enhancement #975: Add new backend for OpenStack Swift

   https://github.com/restic/restic/pull/975
   https://github.com/restic/restic/pull/648

 * Enhancement #998: Improve performance of the fuse mount

   Listing directories which contain large files now is significantly faster.

   https://github.com/restic/restic/pull/998


Changelog for restic 0.6.1 (2017-06-01)
=======================================

The following sections list the changes in restic 0.6.1 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Enh #985: Allow multiple parallel idle HTTP connections
 * Enh #981: Remove temporary path from binary in `build.go`
 * Enh #974: Remove regular status reports

Details
-------

 * Enhancement #985: Allow multiple parallel idle HTTP connections

   Backends based on HTTP now allow several idle connections in parallel. This is especially
   important for the REST backend, which (when used with a local server) may create a lot
   connections and exhaust available ports quickly.

   https://github.com/restic/restic/issues/985
   https://github.com/restic/restic/pull/986

 * Enhancement #981: Remove temporary path from binary in `build.go`

   The `build.go` now strips the temporary directory used for compilation from the binary. This
   is the first step in enabling reproducible builds.

   https://github.com/restic/restic/pull/981

 * Enhancement #974: Remove regular status reports

   Regular status report: We've removed the status report that was printed every 10 seconds when
   restic is run non-interactively. You can still force reporting the current status by sending a
   `USR1` signal to the process.

   https://github.com/restic/restic/pull/974


Changelog for restic 0.6.0 (2017-05-29)
=======================================

The following sections list the changes in restic 0.6.0 relevant to
restic users. The changes are ordered by importance.

Summary
-------

 * Enh #957: Make `forget` consistent
 * Enh #966: Unify repository layout for all backends
 * Enh #962: Improve memory and runtime for the s3 backend

Details
-------

 * Enhancement #957: Make `forget` consistent

   The `forget` command was corrected to be more consistent in which snapshots are to be
   forgotten. It is possible that the new code removes more snapshots than before, so please
   review what would be deleted by using the `--dry-run` option.

   https://github.com/restic/restic/issues/953
   https://github.com/restic/restic/pull/957

 * Enhancement #966: Unify repository layout for all backends

   Up to now the s3 backend used a special repository layout. We've decided to unify the repository
   layout and implemented the default layout also for the s3 backend. For creating a new
   repository on s3 with the default layout, use `restic -o s3.layout=default init`. For further
   commands the option is not necessary any more, restic will automatically detect the correct
   layout to use. A future version will switch to the default layout for new repositories.

   https://github.com/restic/restic/issues/965
   https://github.com/restic/restic/pull/966

 * Enhancement #962: Improve memory and runtime for the s3 backend

   We've updated the library used for accessing s3, switched to using a lower level API and added
   caching for some requests. This lead to a decrease in memory usage and a great speedup. In
   addition, we added benchmark functions for all backends, so we can track improvements over
   time. The Continuous Integration test service we're using (Travis) now runs the s3 backend
   tests not only against a Minio server, but also against the Amazon s3 live service, so we should
   be notified of any regressions much sooner.

   https://github.com/restic/restic/pull/962
   https://github.com/restic/restic/pull/960
   https://github.com/restic/restic/pull/946
   https://github.com/restic/restic/pull/938
   https://github.com/restic/restic/pull/883


