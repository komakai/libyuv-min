# RECIPE: libyuv-min

How the `main-min` branch of this repository was made from upstream, and
everything that differs from it. It is a submodule of webrtc-third_party-min (at `libyuv`) for [webrtc-min](https://github.com/komakai/webrtc-min).

## Upstream

- Repository: https://chromium.googlesource.com/libyuv/libyuv.git
- Revision: 41a6e684a68950f07912b7e6f57ba3fa2e10fb65 (2026-09-23T22:42:30-07:00)

The first commit on `main-min` is an unmodified snapshot of that revision, so
`git diff <first commit> main-min` shows every change made here.

## Stripped

Removed because nothing in the build needs them (run from the repository
root on the snapshot):

```sh
# Tests, tools, docs, CI and the other build systems (Android.bp/mk, Bazel,
# gyp, make, upstream CMake, gn).
git rm -rq build_overrides docs infra riscv_script tools_libyuv unit_test util
git rm -q Android.bp Android.mk BUILD.bazel BUILD.gn CMakeLists.txt \
  CM_linux_packages.cmake codereview.settings DEPS DIR_METADATA \
  download_vs_toolchain.py GEMINI.md .gn libyuv.bzl libyuv.gni libyuv.gyp \
  libyuv.gypi linux.mk OWNERS PRESUBMIT.py public.mk pylintrc vpython.toml \
  vpython.toml.uv.lock winarm.mk WORKSPACE.bazel
```

## Deviations from upstream

- **CMake** (commit "Add CMake build"): `CMakeLists.txt` replaces upstream's
  with Chromium's BUILD.gn targets: `libyuv` (`libyuv_internal`) plus the
  `libyuv_neon` (`-march=armv8-a+dotprod+i8mm` on arm64), `libyuv_sve` and
  `libyuv_sme` objects, `libyuv_config`'s defines, and MJPEG support through a
  `libjpeg` target (libjpeg_turbo-min) unless `LIBYUV_DISABLE_JPEG` is set.
  The LoongArch (lsx/lasx) targets are left out.
- **Apple** (commit "Disable SME on Apple targets"): `libyuv_sme` is built
  only for Android/Linux arm64, so for Apple targets `libyuv_config` also
  defines `LIBYUV_DISABLE_SME` (as gn does for iOS) so the SME paths aren't
  called.

## Updating to a new upstream revision

`main-min` isn't a git fork of upstream (no upstream history), so updates are
re-applied rather than merged:

1. Replace the tree with upstream at the new revision and commit it as
   "Snapshot of upstream at <rev>" (`git rm -rq . && git archive` of the new
   revision, or a copy of a fresh checkout).
2. Re-run the strip commands above and commit.
3. Re-apply the deviation commits (`git cherry-pick` them from the previous
   `main-min` history) and fix up any conflicts.
4. Update the source lists in the CMake files for files upstream added,
   removed or renamed (compare with the new BUILD.gn in the snapshot commit),
   then build webrtc-min and compare with a gn build.
