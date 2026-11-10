# 安裝與更新

本文件說明 WinCD 的安裝、範例包與更新。WinCD 目前以 pre-release 發行，授權為 [Apache License 2.0](https://github.com/cyljacky02/wincd/blob/main/LICENSE)；使用前請先讀要安裝版本的[發行說明](https://github.com/cyljacky02/wincd/blob/main/CHANGELOG.md)，了解驗證範圍與限制。

以下指令都在 Windows PowerShell 5.1（`powershell.exe`）執行。先把 `$version` 設成要安裝的版本：

```powershell
$version = '0.1.2'
```

## 1. 下載與核對

從同一個 [GitHub Release](https://github.com/cyljacky02/wincd/releases) 下載：

- `wincd-engine-<版本>.zip`：引擎安裝包
- `SHA256SUMS`
- 選用：`wincd-samples-<版本>.zip`：範例包，見第 4 節

計算每個 zip 的 SHA-256，與 `SHA256SUMS` 中同名檔案的值比對；不一致就停止。

```powershell
Get-FileHash ".\wincd-engine-$version.zip" -Algorithm SHA256
Get-FileHash ".\wincd-samples-$version.zip" -Algorithm SHA256
Get-Content .\SHA256SUMS
```

SHA-256 只證明檔案內容相同；與安裝包放在一起下載、沒有簽章的 checksum，不能獨立證明發行者的身分。見[發行完整性政策](https://github.com/cyljacky02/wincd/blob/main/RELEASE-POLICY.md)。

## 2. 安裝到新的專用目錄

以**已提升權限的 Windows PowerShell 5.1** 執行。以下只適用於全新安裝；不要解壓到既有的引擎或進行中的部署上。根目錄預設為 `C:\WinCD`，要放在其他位置見下方「自選根目錄」。

```powershell
$root = 'C:\WinCD'
Unblock-File ".\wincd-engine-$version.zip"
New-Item $root -ItemType Directory -ErrorAction Stop
icacls $root /inheritance:r /grant:r '*S-1-5-32-544:(OI)(CI)F' '*S-1-5-18:(OI)(CI)F' '*S-1-5-32-545:(OI)(CI)RX'
if ($LASTEXITCODE -ne 0) { throw '無法限制安裝目錄的權限。' }
Expand-Archive ".\wincd-engine-$version.zip" -DestinationPath $root
& "$root\engine\wincd.ps1" status
```

zip 內只有一個最上層的 `engine` 資料夾。安裝後的配置：

```text
C:\WinCD\engine\  執行檔案，入口是 wincd.ps1
C:\WinCD\inbox\   收件資料夾：要換版的 release zip，需要時才建立
C:\WinCD\data\    資料根目錄：狀態、紀錄、備份與 release 保留區，需要時才建立
```

不要讓不受信任的使用者能寫入 `engine`。release 內容與備份可能含有明文帳密；不要公開或廣泛分享 `inbox` 與 `data`。

`status` 是唯讀的：不建立資料目錄、不修復權限，也不設定 IIS。變更指令會自行準備需要的目錄，並檢查各自的前提。

### 自選根目錄

根目錄就是 `engine` 資料夾的上一層。沒有安裝設定檔或安裝參數：引擎一律使用自己所在的根目錄，`engine`、`inbox`、`data` 都在它底下。例如要放在 D 槽，就在上面的指令把 `$root` 設為 `'D:\WinCD'`：

```text
D:\WinCD\engine\
D:\WinCD\inbox\
D:\WinCD\data\
```

- 根目錄必須是 WinCD 專用的本機磁碟絕對路徑；不能是磁碟根目錄（例如 `D:\`），不能在 Windows 目錄內，也不能是 `C:\Program Files`、`C:\Program Files (x86)`、`C:\ProgramData`、`C:\Users` 本身；路徑上不能有 junction 或 symlink。
- 引擎資料夾必須名為 `engine`，直接放在根目錄下。
- `engine` 與 `data` 不能分開放：`data` 固定是 `<根目錄>\data`，也不能是指向其他位置的 junction。TUI 與 runner／agent 因此一定使用同一個資料根目錄。
- 範例包的導覽以 `C:\WinCD` 為例；使用其他根目錄時，請替換導覽中的路徑。

## 3. 開啟 TUI

```powershell
& C:\WinCD\engine\wincd.ps1
```

根目錄不是 `C:\WinCD` 時請替換路徑。主控台介面是繁體中文；在後面加上動詞（例如 `status`、`deploy`）則改用指令列。請在 PowerShell 中以呼叫運算子 `&` 啟動 `wincd.ps1`，如上所示；v0.1.1（含）以前的版本不能用 `powershell.exe -File` 啟動。試用時不要指向正式環境的服務、目錄或導流端點。

選用的 `install-monitor` 指令會變更 IIS 設定與權限，並需要額外的主機準備；不要把它當成例行的安裝檢查。公開的監測端點回報的是本機的導流意圖，不是應用程式的健康狀態，也不是負載平衡器已確認的成員狀態。

## 4. 選用：範例包

`wincd-samples-<版本>.zip` 是另外下載的範例包，與引擎安裝包由同一個 commit 建置。它不是引擎的一部分，不要解壓到 `engine` 資料夾。

| 範例包內的路徑 | 內容 |
| --- | --- |
| `README.md` | 測試機上手導覽：換版 v1 → 換版 v2 → 退版 → 清理，並列出對應的指令列指令。 |
| `INSTALL.md` | 本文件（v0.1.2 起）。 |
| `demo/wincd-demo-v1.zip`、`demo/wincd-demo-v2.zip` | 專案 `wincd-demo` 的 release：不需要 Tomcat、IIS 或 F5，只寫入 `C:\WinCD-demo\`。 |
| `reference/order-api/` | 逐欄註解的完整部署設定參考，並列出 IIS 站台、程序服務、檢查步驟與 F5 模式的替代寫法。它指向 `D:\Tomcat9`、`C:\nginx` 與 IIS 監測站台，不能在乾淨的測試機上直接換版；撰寫自己的部署設定時從這裡複製。 |

demo **只在測試機使用**：它以管理員身分執行，讓 `C:\WinCD-demo\site` 與每次換版的 release 一致（完整同步會刪除 release 中沒有的檔案，保留清單除外），而且 `wincd-demo` 專案的紀錄會留在 WinCD 的 `data`，WinCD 沒有刪除專案紀錄的指令。

## 更新

更新前，先結束所有 WinCD TUI 與工作，並處理完未完成的復原作業。核對新安裝包與發行說明中的相容性，然後整個換掉 `engine` 資料夾；`inbox` 與 `data` 保持不動。不要逐檔覆蓋，也不要假設舊版引擎能讀取較新的部署設定或交易狀態。

```powershell
$root = 'C:\WinCD'
Unblock-File ".\wincd-engine-$version.zip"
Remove-Item "$root\engine" -Recurse
Expand-Archive ".\wincd-engine-$version.zip" -DestinationPath $root
& "$root\engine\wincd.ps1" status
```

公開版本不承諾與更早的開發快照相容或可遷移。選用的 IIS handler 有變更時，可能需要另外規劃監測站台的安裝或更新；一般執行期操作不會自動修復或升級它。
