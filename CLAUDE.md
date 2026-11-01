# FramePlayer fork

[yellkell/frameplayer](https://github.com/yellkell/frameplayer) 的個人 fork，給 Valve Steam Frame 用的原生 VR 影片播放器（Rust、OpenXR、Vulkan、egui、FFmpeg）。

回覆用繁體中文，code 註解和 commit message 用英文。

## 分支與版本

- `mike/main`：這個 fork 的主線，也是 GitHub 預設分支。
- `upstream/tester-feedback`：原作者實際開發的分支，版本 tag 也在這裡。`upstream/main` 比它舊，比對和拆 PR 都要以 `tester-feedback` 為基底。
- 版本號：`crates/frameplayer/Cargo.toml` 的 `0.3.0-alpha.20.mike.N`。每次裝到 Frame 前 N 加一，方便從 log 第一行分辨版本。
- remote：`origin` 是這個 fork，`upstream` 是原作者。

## 只屬於這個 fork 的改動

這些不要送回原作者：

- `crates/fp-updater/src/lib.rs` 的 `RELEASE_PUBLIC_KEY = None`，避免原作者的新版蓋掉這個 build。
- `tools/docker/`、`mike` 版本號、README 開頭的 fork 說明、這份文件。
- `.gitignore` 的機密規則區塊。

其他修正都已拆成 PR 送給原作者，狀態見 README。送 PR 時：

1. 從 `upstream/tester-feedback` 開分支，只帶該修正需要的檔案，不要有版本號改動。
2. 每個分支單獨跑完整測試。
3. 目標分支選 `tester-feedback`，不是 `main`。

## 建置與測試

開發機是 Apple Silicon Mac，全部在 Linux arm64 Docker 裡跑（OrbStack 要先啟動）。

第一次要建 image 和 FFmpeg：

```bash
docker build -t frame-player-build tools/docker
docker run --rm --platform linux/arm64 -v "$PWD":/src frame-player-build tools/build-ffmpeg.sh aarch64
```

測試：

```bash
docker run --rm --platform linux/arm64 -e CARGO_TARGET_DIR=/src/target -v "$PWD":/src -v frame-player-rel:/src/target -v frame-player-cargo:/usr/local/cargo/registry frame-player-build cargo test --locked --workspace --exclude frame-probe --no-fail-fast
```

打包（輸出到 `dist/`）：

```bash
docker run --rm --platform linux/arm64 -e CARGO_TARGET_DIR=/src/target -v "$PWD":/src -v frame-player-rel:/src/target -v frame-player-cargo:/usr/local/cargo/registry frame-player-build tools/package.sh
```

## 部署到 Frame

SSH alias 是 `frame`，App 在 `~/devkit-game/FramePlayer`，由 Steam 的 Non-Steam 捷徑啟動。

- 換版前先確認 `pgrep -x frameplayer` 沒有結果，App 開著不要覆蓋。
- 解壓 zip 後取代整個資料夾，並保留 `.frame-artwork`（Steam 封面圖）。
- 原版備份在 `~/devkit-game/FramePlayer.upstream-alpha20`，不要刪。
- log：`~/.local/share/frameplayer/frameplayer.log`，上一次的是 `.log.old`。
- `~/.config/frameplayer/sources.json` 可能有 WebDAV 帳密，不要讀取或複製。

## 不戴頭盔的除錯

`--preview` 用 Frame 的真 GPU 和真解碼器離屏繪製，能截圖驗證。一律用暫存設定，不碰真實資料：

```bash
T=$(mktemp -d); cd ~/devkit-game/FramePlayer
printf 'open /path/to/video.mp4\nwait-playing 12\nwait 3\nstatus\nshot result\n' > $T/s.fps
XDG_CONFIG_HOME=$T/cfg XDG_DATA_HOME=$T/data LD_LIBRARY_PATH=$PWD/lib timeout 40 ./frameplayer --preview $T/out --script $T/s.fps
```

- 腳本指令列表在 `crates/frameplayer/src/preview.rs` 開頭。
- 純解碼速度：`./frameplayer --decode-bench FILE --frames N`，加 `--sw` 強制軟解。
- 預覽正常但頭盔不對，就看 SteamVR log：`journalctl --since ... | grep -E "xrclient_frameplayer|XRService|vrcompositor"`。
- 直接用路徑開影片不會配對外掛字幕，要從媒體庫開。

## 已知的坑

- **`frame-probe`**：測試目標在原作者程式裡編不過（`vk.rs` E0507），和這個 fork 無關，所以測試要排除它。
- **10-bit**：Frame 沒有可用的硬解路徑。`hevc_v4l2m2m` 會直接打開 Main 10，然後綠屏或卡住，而且不報錯。只有 8-bit 4:2:0 能走硬解，判斷在 `crates/fp-media/src/decode.rs` 的 `hw_for_format`。
- **黑畫面**：啟動後追蹤重新定位可能讓空間轉半圈，選單被甩到背後，程式其實正常。看 log 有沒有 `tracking space changed: recentering`。
- **每幀 CPU 很低不代表 UI 沒跑**：面板只在收到事件時重畫。
- **字型**：Noto Sans CJK 用系統的 `/usr/share/fonts/noto-cjk/`，TC 在 face index 3。檔案約 20 MB，每個面板有自己的 egui context，所以用 `OnceLock` 共用，只讀一次。
- **字幕編碼**：內建 FFmpeg 沒有 iconv，靠 `chardetng` 判斷後自己轉。`guess` 不要傳 `tw` 提示，否則短的 GBK 會被認成 Big5。
- **Docker 檔案同步**：剛改完檔案馬上跑，可能讀到舊內容或截斷的檔案（會看到奇怪的 `unclosed delimiter`）。等幾秒重跑，測試結果不合理時先 `touch` 改過的檔案。
- **zsh**：`"$1:rate"` 的 `:r` 會被當修飾字，要寫 `"${1}:rate"`；`scp frame:/path/*.png` 的星號也會被本機展開。
- **Docker image 沒有 clippy 和 rustfmt**：需要的話先 `rustup component add clippy rustfmt`。

## Push 前必查

1. `git status`：沒有 `.env`、`token*.json`、`credentials*.json`、`*.key`、`*.pem`。
2. `cat .gitignore`：確認規則還在。
3. 掃 diff：沒有 API key、密碼、Bearer token、個人路徑或影片來源網址。
4. commit 作者用 GitHub noreply 信箱。

`dist/`、`target/`、`third_party/` 都不進 git。沒有確認過 `packaging/licenses/THIRD-PARTY.txt` 的條款之前，不要公開發布編譯好的 zip。另外 `tools/package.sh` 預設的下載網址指向原作者，發布時要用 `--url-base` 改掉。
