# FramePlayer (Mike's fork)

**Unofficial fork** of [yellkell/frameplayer](https://github.com/yellkell/frameplayer), not affiliated with or endorsed by the original author. All credit for FramePlayer goes to yellkell.

這是 Valve Steam Frame 影片播放器 FramePlayer 的非官方個人 fork，以 `v0.3.0-alpha.20` 為基礎，主要補上中文使用需要的修正。

## 和原版的差異

| 改動 | 解決的問題 | 給原作者的 PR |
|---|---|---|
| 中文介面與字幕 | 中文檔名和字幕顯示不出來，現在使用系統的 Noto Sans CJK | [#8](https://github.com/yellkell/frameplayer/pull/8) |
| 字幕編碼轉換 | Big5、GBK、UTF-16 字幕檔會被丟掉，現在自動轉成 UTF-8 | [#8](https://github.com/yellkell/frameplayer/pull/8) |
| 10-bit 影片改走軟解 | 10-bit H.265 綠屏、破圖或卡住 | [#6](https://github.com/yellkell/frameplayer/pull/6) |
| 追蹤跳動時移回選單 | 啟動後畫面全黑，選單其實被甩到背後 | [#7](https://github.com/yellkell/frameplayer/pull/7) |
| 關閉 App 內更新 | 避免原版更新蓋掉這個 build（只屬於這個 fork） | — |
| Docker 建置環境 | 可在 Apple Silicon Mac 上編譯（只屬於這個 fork） | — |
| Steam 封面 | 自製的遊戲庫封面、橫幅和圖示（只屬於這個 fork） | — |

## 使用限制

- 10-bit 影片是軟解，4K 通常跑得動，但 8K、高位元率或高幀率片源可能掉幀，也比較耗電。
- 中文字型來自系統，SteamOS 內建；系統沒有這套字型時，中文不會顯示，其他功能不受影響。
- 內嵌在 MKV 裡的非 UTF-8 字幕不會轉換，很短的非 UTF-8 字幕檔也可能判斷錯編碼。
- 這個 fork 不提供編譯好的安裝包，也不會自動更新，要自己編譯。

## 自己編譯

需要 Docker，指令在 [CLAUDE.md](CLAUDE.md) 的「建置與測試」和「部署到 Frame」。

```bash
git clone https://github.com/mikelitw-bot/frameplayer.git
cd frameplayer
docker build -t frame-player-build tools/docker
```

輸出的 zip 在 `dist/`，解壓到 Frame 的 `~/devkit-game/FramePlayer` 後，用 Steam 捷徑啟動。

## 授權

和原版相同，採 [Apache License 2.0](LICENSE-APACHE) 或 [MIT](LICENSE-MIT) 雙授權，原作者的版權聲明維持不變。

---

以下是原版 README。

# FramePlayer

A native Linux ARM64 VR video player for the Valve Steam Frame. Aims to be a complete, open-source DeoVR replacement with one-click installation.

See [docs/OUTLINE.md](docs/OUTLINE.md) for the full project outline: platform constraints, feature set, architecture, install strategy, milestones, and risks.

Also here: Chromium XR for Steam Frame, a separate WebXR browser app (ff2.web.app and other WebXR pages); [docs/webxr/README.md](docs/webxr/README.md) has its build, patches and install.

## Status

The player is feature-complete for a first release and runs end to end
against a simulated OpenXR headset (Monado) and in the headless preview
harness; it has not yet run on Steam Frame hardware.

- Plays flat, 180°, 360°, fisheye (190/200/220°) and EAC video, mono or
  stereo, with per-video format overrides and adjustments
- Library with thumbnails, search, filters, favorites, ratings, resume and
  history; ~/Videos, ~/Downloads, microSD cards and USB drives are indexed
  automatically (exFAT cards from Windows/macOS included: FramePlayer
  mounts what SteamOS won't)
- Network sources: DeoVR/HereSphere feeds (XBVR, Stash), SMB, WebDAV, DLNA,
  HTTP folders
- Haptics: funscripts with Intiface, TCode devices and The Handy
- Phone web remote and DeoVR remote API
- Signed self-updates; install via Frame Control/FrameDrop or
  `frameplayer-install`

Guides: [user guide](docs/USER-GUIDE.md), [development](docs/DEVELOPMENT.md).

`frame-probe` (Milestone 0) answers the remaining platform questions in one
run on a headset (`tools/frame-probe.sh --build`); results go in
[docs/platform-notes.md](docs/platform-notes.md).

## License

FramePlayer is licensed under either of [Apache License, Version 2.0](LICENSE-APACHE)
or [MIT license](LICENSE-MIT), at your option. Bundled third-party libraries
and fonts keep their own licences, and the controller pictures are renders of
Valve's Steam Frame controller models; see [packaging/licenses](packaging/licenses/THIRD-PARTY.txt).

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in FramePlayer by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
