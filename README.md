<p align="center">
  <img src="docs/icon.png" width="96" alt="">
</p>

<h1 align="center">spoti.pw</h1>

<p align="center">Spotify, in glass.</p>

<p align="center">
  <img src="https://img.shields.io/badge/iOS-000000?style=for-the-badge&logo=ios&logoColor=white" alt="iOS">
  <img src="https://img.shields.io/badge/Spotify-9.1.78-1ED760?style=for-the-badge&logo=spotify&logoColor=white" alt="Spotify 9.1.78">
  <img src="https://img.shields.io/badge/Objective--C-3A95E3?style=for-the-badge&logo=apple&logoColor=white" alt="Objective-C">
  <img src="https://img.shields.io/badge/GitHub_Actions-2671E5?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions">
  <img src="https://img.shields.io/badge/License-PolyForm_Strict_1.0.0-blue?style=for-the-badge" alt="PolyForm Strict 1.0.0">
</p>

<p align="center">
  <a href="https://spoti.pw">spoti.pw</a> ·
  <a href="#build-it">Build it</a> ·
  <a href="docs/tweaks.md">Hack on it</a> ·
  <a href="https://ko-fi.com/darkksh">Support</a>
</p>

<p align="center">
  <img src="docs/screenshots/now-playing.webp" width="16%" alt="Full screen player with lyrics">
  <img src="docs/screenshots/album.webp" width="16%" alt="Album">
  <img src="docs/screenshots/playlist.webp" width="16%" alt="Playlist">
  <img src="docs/screenshots/queue.webp" width="16%" alt="Queue">
  <img src="docs/screenshots/live-activity.webp" width="16%" alt="Live Activity on the lock screen">
  <img src="docs/screenshots/home.webp" width="16%" alt="Home">
</p>

> This is a fork of [skopevoj/spoti.pw](https://github.com/skopevoj/spoti.pw).
> The product website, releases, support links, and star history below refer to
> the upstream project. Build this fork from its checked-out source.

A no-jailbreak Theos tweak that rebuilds Spotify for iOS in Liquid Glass, injected into your own
decrypted IPA and signed with your own certificate.

Built and tested on **Spotify 9.1.78** — use that version's IPA. The mod hooks Spotify's own classes,
which change between releases, so another version may build and then break.

| | |
|---|---|
| The redesign | **iOS 26+** |
| Legacy look | iOS 16.1+ |
| Live Activity | iOS 17+ |

The redesign is `UIGlassEffect`, which only exists from iOS 26. Below that the Redesigned UI switch
is greyed out and the mod runs Spotify's own screens with everything else it adds on top. Both live
in Settings → Mod Settings.

## What it changes

Choose **Native** or **Redesigned UI** in Settings → Mod Settings → Appearance.
Native keeps Spotify's screens with configurable visual tweaks; Redesigned UI
uses the mod's Liquid Glass player, navigation, and browsing screens on iOS 26+.
The two looks are selected at launch, so restart Spotify after changing the mode.

Shared settings include lyrics sources, gestures, blocked artists, audio effects,
playback speed and pitch, haptics, privacy options, and Live Activity controls.
See [the tweak guide](docs/tweaks.md) for the feature layout and implementation notes.

## Build it

No IPA is distributed. Bring a decrypted **Spotify 9.1.78** IPA; you get an unsigned
`spoti.pw-<mod version>.ipa` to sign with SideStore, Feather or any certificate signer. Each
[release](https://github.com/skopevoj/spoti.pw/releases) also carries the tweak's `.deb`.

### Build with GitHub Actions

Fork the repo, enable Actions, run **Build IPA from your own Spotify IPA**. It takes a direct link to
your decrypted `.ipa` and, with the default **artifacts** upload method, returns the
built IPA as a workflow artifact. The job runs on a macOS GitHub runner, so a local
Mac is not required. The workflow includes input masking; keep IPA URLs out of
issues and pull requests. Choosing **filebin** or **both** uploads the result to
an external file-sharing service.

### Build on a Mac

Install Theos in `~/theos` (or set `THEOS` to its directory), and select a full
Xcode installation with an iPhoneOS 26+ SDK using `xcode-select`. The current
pipeline also invokes `xcrun --sdk iphoneos clang` to build the App Group shim;
a standalone SDK in Theos does not satisfy that step.

Clone this fork and install the build tools. The `uv` command is required for the
Python tool installation below:

```bash
git clone https://github.com/t91a60/spoti.pw.git
cd spoti.pw
```

    brew install make ldid dpkg zsign ideviceinstaller libimobiledevice
    uv tool install --with pillow "cyan @ git+https://github.com/asdfzxcvbn/pyzule-rw"

Put the decrypted `.ipa` in `ipa/`, then:

    make release    # out/spoti.pw-<version>.ipa, ready to sign
    make install    # out/spoti.dev.ipa, signed and pushed over USB

`make install` reads `SIGN_P12`, `SIGN_PROFILE` and `SIGN_P12_PASSWORD` from `.signing.env`; copy
`.signing.env.example` and fill it in:

```bash
cp .signing.env.example .signing.env
```

Keep the certificate, provisioning profile, password, and `.signing.env` private.
`make install` uses a development name and icon; its cyan installation needs Pillow
for the icon conversion. Use `make release IPA=/path/to/spotify.ipa` to select an
IPA explicitly instead of the first file in `ipa/`.

The first build spends a minute reading Spotify's flags out of your IPA. `make flags` regenerates it.

### Signing

Sign with a bundle id matching your certificate's App ID. If it doesn't match, the app still works
but tapping the player on the lock screen won't open it — and it tells you on first launch which id
to use. In Feather, copy the App ID into **Identifier** and leave **PPQ protection** off; AltStore,
SideStore and Sideloadly get this right on their own.

By default, the release build keeps Spotify's bundle id. The signer or local
installation script may change it to match your provisioning profile.

## Source layout and checks

| Path | Purpose |
| --- | --- |
| `tweak/Sources/Native/` | Changes to Spotify's existing screens |
| `tweak/Sources/Redesigned/` | Liquid Glass interface |
| `tweak/Sources/Shared/` | Behavior and data used by either look |
| `tweak/Sources/Core/`, `Settings/`, `App/` | Infrastructure, settings UI, and composition |
| `extension/` | Live Activity extension and App Group support |
| `scripts/` | Build, injection, signing, and device inspection |
| `vendor/` | Third-party audio libraries and FLEX inspector |
| `docs/tweaks.md` | Architecture and feature-development guide |

The source uses Objective-C, Logos, Swift, and C with Theos. There is no standalone
automated test suite in this snapshot. The build runs an import-layer check;
you can run it independently:

```bash
sh scripts/check-layers.sh
```

On a configured Mac, `make release` checks the full build pipeline. Runtime
validation requires the matching Spotify IPA and an iOS device. `make log` streams
device logs; `make install FLEX=1` includes the inspector used by `make session`
and `make trees`. Read [the tweak guide](docs/tweaks.md) before changing hooks.

### Troubleshooting

- **No IPA:** put a decrypted IPA in `ipa/` or pass `IPA=/path/to/spotify.ipa`.
- **Missing tool or SDK:** check the pipeline's named prerequisite and selected
  Xcode installation. Building the shim and extension requires the iPhoneOS tools.
- **Redesigned UI is unavailable:** it requires iOS 26; use Native on older versions.
- **A setting appears unchanged:** restart Spotify; UI mode and many hooks are
  selected when the app launches.
- **Another Spotify version fails:** internal class hooks can change. This
  snapshot's documented target is 9.1.78, not an assurance for other versions.

## Support

If it made your phone nicer to use, a coffee is a good way to say so.

<a href="https://ko-fi.com/darkksh">
  <img src="https://img.shields.io/badge/Ko--fi-Buy_me_a_coffee-FF5E5B?style=for-the-badge&logo=kofi&logoColor=white" alt="Support on Ko-fi">
</a>

## Contributing

Pull requests are welcome. The pull request's description has a box for agreeing to the
[Contributor License Agreement](CLA.md), which gives the project's owner the rights to the
contribution; it is ticked once, before the first pull request is merged.

## Star history

<a href="https://star-history.com/#skopevoj/spoti.pw&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=skopevoj/spoti.pw&type=Date&theme=dark">
    <img src="https://api.star-history.com/svg?repos=skopevoj/spoti.pw&type=Date" alt="Star history chart">
  </picture>
</a>

## Credits

[cyan](https://github.com/asdfzxcvbn/pyzule-rw) injects, [Theos](https://theos.dev) builds, and
[FLEX](https://github.com/FLEXTool/FLEX), as hopeless's AutoFLEX build in `vendor/`, is the inspector
the view trees are read through.

## License

Source available under the [PolyForm Strict License 1.0.0](LICENSE): you can read the code and use
the mod yourself, but not change it, reuse it in other projects or redistribute it. Releases up to
v0.21.1 were published under GPL-3.0 and stay under it. Files in `vendor/` and `.agents/` keep their
own licences.

Not affiliated with Spotify.
