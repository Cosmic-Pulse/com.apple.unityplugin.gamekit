# com.apple.unityplugin.gamekit — Cosmic Pulse mirror

Apple's GameKit plug-in for Unity, built from [`apple/unityplugins`](https://github.com/apple/unityplugins)
and republished here so it can be installed by UPM git URL. Apache 2.0 (`LICENSE.txt`);
Apple is the author, Cosmic Pulse only builds and hosts.

Apple publishes no GitHub releases and no registry listing for these plug-ins. The only
supported way to consume them is to clone the repo and run `build.py`, which compiles the
native libraries with Xcode. Partner games can't be asked to do that, so the build happens
once here and partners pin a tag.

Consumed by [cosmic-sdk](https://github.com/Cosmic-Pulse/cosmic-sdk) — see its README for
the manifest block. **Pin a tag; never track `main`.**

## Publishing a version

Run the **Mirror** workflow (Actions → Mirror → Run workflow). It runs on the self-hosted
macOS runners (Astra / Anubis), builds from the upstream tag you give it, commits the
package to `main`, and tags `v<version>`. Tags are written once and never moved — partners
pin them.

Inputs worth knowing:

| Input | Default | Notes |
|---|---|---|
| `upstream_tag` | `GameKit-4.0.1` | Apple tags per plug-in |
| `platforms` | `iOS iPhoneSimulator tvOS AppleTVSimulator` | macOS omitted — see below |
| `xcode_app` | `/Applications/Xcode.app` | Must be a released Xcode; the workflow refuses a beta |
| `raise_deployment_targets` | off | Turn on only if the runner's Xcode refuses upstream's floors |
| `publish` | on | Off builds and verifies without pushing |

## Deviations from upstream

- **No macOS native library.** Upstream's `GameKitWrapperMac` target is pinned to Apple's
  own signing team, so it can't be built outside Apple without re-pointing it at ours.
  No partner game ships a macOS player. A macOS Unity build would fail to link.
- **No visionOS slice.** Not built, and no partner targets it. The C# assembly still lists
  VisionOS as a supported platform, so build the slice before shipping a visionOS title.
- **`Tests/` removed.** Upstream's test assembly references `com.unity.test-framework`,
  which a partner project need not have installed; without it the assembly fails to
  compile and takes the console down with it.
- **Deployment targets** may be raised to iOS/tvOS 15.0 and macOS 12.0 when the build
  Xcode refuses upstream's 13.0/10.15 floors. Unity 6000.5 defaults to 15.0, so this costs
  partner games nothing. Whether it was applied is recorded in each publish commit.

Every published commit records the upstream tag, the exact Xcode, the machine, and the
platform list, so any tag here can be traced back to how it was built.

## Why not a beta Xcode

Two reasons, both load-bearing. Apple can reject App Store submissions containing binaries
built with beta tooling — and partners ship these binaries. And the beta SDK drifts ahead
of Apple's own source: the iOS 27 beta SDK added a `GKGameActivityListener` requirement
that GameKit 4.0.1 doesn't implement, so the build fails outright. The workflow refuses a
beta Xcode rather than producing something that looks fine until review.
