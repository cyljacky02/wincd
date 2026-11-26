# 安裝與更新

本文件說明 WinCD 的安裝、範例包與更新。WinCD 目前以 pre-release 發行，授權為 [Apache License 2.0](https://github.com/cyljacky02/wincd/blob/main/LICENSE)；使用前請先讀要安裝版本的[發行說明](https://github.com/cyljacky02/wincd/blob/main/CHANGELOG.md)，了解驗證範圍與限制。

以下指令都在 Windows PowerShell 5.1（`powershell.exe`）執行。先把 `$version` 設成要安裝的版本：

```powershell
$version = '0.4.2'
```

## 1. 下載與核對

從同一個 [GitHub Release](https://github.com/cyljacky02/wincd/releases) 下載：

- `wincd-engine-<版本>.zip`：引擎安裝包
- `SHA256SUMS`
- `install.bat`：雙擊安裝或更新的安裝程式，見第 2 節
- 選用：`install-fast.bat`：固定安裝或更新 `D:\WinCD` 的快速版，見第 2 節
- 選用：`wincd-samples-<版本>.zip`：範例包，見第 4 節
- 選用：`WinCD-Packager-<版本>.zip` 與 `WinCD-Packager-<版本>.zip.sha256`：平台維護者的交接打包工具，見第 5 節

計算每個 zip 與 `install.bat`（或 `install-fast.bat`）的 SHA-256，與 `SHA256SUMS` 中同名檔案的值比對；不一致就停止。打包工具 ZIP 裡有 `SHA256SUMS`，所以它自己的 SHA-256 在 `WinCD-Packager-<版本>.zip.sha256`，不在 `SHA256SUMS`。

```powershell
Get-FileHash ".\wincd-engine-$version.zip" -Algorithm SHA256
Get-FileHash ".\wincd-samples-$version.zip" -Algorithm SHA256
Get-FileHash .\install.bat -Algorithm SHA256
Get-FileHash .\install-fast.bat -Algorithm SHA256
Get-Content .\SHA256SUMS
```

SHA-256 只證明檔案內容相同；與安裝包放在一起下載、沒有簽章的 checksum，不能獨立證明發行者的身分。見[發行完整性政策](https://github.com/cyljacky02/wincd/blob/main/RELEASE-POLICY.md)。`install.bat` 執行時會再核對一次引擎安裝包，但不能核對它自己。

## 2. 安裝

### 用 install.bat 安裝或更新（建議）

把 `install.bat`、`wincd-engine-<版本>.zip` 與 `SHA256SUMS` 放在同一個資料夾，雙擊 `install.bat`。不是系統管理員時會跳出 UAC，在新的系統管理員視窗中繼續。從網路下載的 `install.bat` 第一次執行時 Windows 可能先詢問是否執行。

1. 資料夾中有多個 `wincd-engine-<版本>.zip` 時選擇版本，預設最新。安裝包的 SHA-256 必須與 `SHA256SUMS` 相符，zip 內的引擎版本必須與檔名相同，否則停止。確認後只解壓複製到根目錄、再核對一次的副本；安裝包在等待確認時被改動就停止。
2. 搜尋每個本機固定磁碟根目錄下一層的既有安裝（例如 `D:\WinCD`），優先提供**更新**。有磁碟或資料夾無法檢查（例如鎖住的 BitLocker 磁碟，或磁碟根目錄下的 junction／symlink：安裝程式不經過它們檢查）時會列出；那裡可能已有安裝，所以新安裝要再確認一次，帶 `-Root` 的新安裝則停止：
   - 安裝包較新：更新；版本相同：重新安裝（修復引擎檔案）；安裝包較舊：不提供，不降版。
   - 只有管理員能改動的安裝才會被執行或更新。一般使用者也能在磁碟根目錄建立看起來像安裝的資料夾，所以根目錄或 `engine` 中任何檔案的擁有者不是管理員、非管理員帳號可以寫入，或上層資料夾可以被非管理員改名時，會列為「不可信」並說明原因：不會執行它，也不會更新它（和無法檢查的地方一樣，新安裝要再確認一次）。確定是自己的安裝時，先確認內容沒有被改動過，再以已提升權限的 PowerShell 收緊權限後重試（或依下方「更新」一節手動更新）：

     ```powershell
     icacls D:\WinCD /setowner '*S-1-5-32-544' /T /C
     icacls D:\WinCD /inheritance:r /grant:r '*S-1-5-32-544:(OI)(CI)F' '*S-1-5-18:(OI)(CI)F' '*S-1-5-32-545:(OI)(CI)RX'
     icacls D:\WinCD\engine /reset /T /C
     ```

     手動裝在更深位置的安裝（例如 `E:\Deploy\WinCD`），上層的 `E:\Deploy` 也要只有管理員能改名或刪除，否則整個安裝能被換掉：對 `E:\Deploy` 執行上面第 1、2 行（把 `/T` 拿掉，只改它本身）。

   - 一台主機一次只能執行一個安裝程式；另一個正在執行（例如停在確認）時停止。確認後、替換前會再讀一次已安裝的版本，和摘要不同就停止。
   - 更新前以已安裝的引擎執行 `status`，有專案待復原或交易未完成就停止。請先完成復原作業並結束所有 TUI 與 job。
   - 跨過資料格式的單向門（0.2.0、0.3.0）時，摘要會提醒更新後每個專案要先重新換版一次。
   - 只替換 `engine` 資料夾，`inbox`、`data` 與監測站台不動。有進行中的換版、退版或復原作業，有程序的指令列以路徑執行這個引擎（例如 `powershell -File D:\WinCD\engine\wincd.ps1`，或以相對路徑執行任何 `wincd.ps1`），或有程式開著引擎中的檔案時停止，引擎保持不變。以下無法偵測，更新前請自行結束：在 PowerShell 中以 `&` 開啟的 TUI（否則它之後會混用新舊引擎的檔案），以及 CI 以暫存腳本執行、沒有交易的指令（例如 `status`）。
   - 主機有 F5 監測站台時，完成後會顯示以相同 port 重跑 `install-monitor` 的指令。
   - 防毒軟體或搜尋索引可能正短暫開著剛解壓的檔案，這時 Windows 不讓安裝程式放置 `engine` 資料夾。安裝程式會顯示「等待其他程式（例如防毒掃描）放開 … 裡的檔案」，最多等約 30 秒後重試（v0.4.2 起）；仍失敗時照訊息處理。
3. 沒有安裝時選擇根目錄：每個本機固定磁碟的 `X:\WinCD`（顯示剩餘空間），或自訂路徑。根目錄的規則見下方「自選根目錄」；另外必須直接在磁碟根目錄下（例如 `E:\Deploy`），`install.bat` 下次才找得到它。已經手動裝在更深位置的引擎不會被偵測到，請用 `install.bat -Root <根目錄>` 更新。既有的空資料夾也可以，它原本的權限會被改成只有管理員能寫入；但它必須是管理員建立的（其他帳號建立的會停止，請刪除它讓安裝程式自己建立）。已有安裝時也可以從選單選「安裝到另一個路徑」（只能從選單），但需要再次確認：一台主機只應有一個 WinCD，每個根目錄有自己的 `data`。
4. 顯示摘要，輸入大寫 `Y` 才執行。新安裝的結果與下方手動步驟相同：限制根目錄權限、放入 `engine`，再執行唯讀的 `status`。
5. 完成後可以直接開啟 TUI。

**快速版 `install-fast.bat`**：根目錄固定為 `D:\WinCD`，版本固定為資料夾中最新的安裝包，不問版本與路徑。同樣放在安裝包與 `SHA256SUMS` 旁邊雙擊，只在摘要後輸入一次大寫 `Y`。`D:\WinCD` 已安裝時更新它，沒有時全新安裝。安裝包核對、不降版、有待復原時拒絕等檢查都與 `install.bat` 相同。

- 主機沒有 D 槽（或 D 槽不是本機固定磁碟）時停止，沒有改動主機；請改用 `install.bat`。
- 主機已經裝在其他位置（例如 `C:\WinCD`）時停止，不會另裝一份；要更新既有安裝請改用 `install.bat`。
- 後面也可以接參數，輸入的優先，例如在已提升權限的 cmd 執行 `install-fast.bat -Yes` 完全不詢問。

自動化（例如大量建置主機）可以在已提升權限的 cmd 或 PowerShell 中帶參數執行，不會詢問：

```bat
install.bat -Root D:\WinCD -Yes
install.bat -Version 0.4.2 -Root D:\WinCD -Yes
```

- `-Root` 是既有安裝時更新它，否則全新安裝到那裡；這台主機已有其他安裝時拒絕，不會另裝一份（有沒有 `-Yes` 都一樣）。
- 不帶 `-Root` 的 `-Yes` 只在恰好偵測到一個安裝時更新它。
- 沒有 `-Version` 時使用資料夾中最新的版本。
- 路徑含空白或 `&` 等特殊字元時加雙引號，例如 `install.bat -Root "D:\A&B"`。`install.bat` 把參數原樣交給安裝程式，不會讓 cmd 當成指令執行；路徑只能用主控台字碼頁能表示的字元（繁體中文 Windows 的中文路徑可以）。
- exit code：`0` 完成、`1` 拒絕（沒有改動主機）、`2` 失敗（訊息說明現況）。新引擎已就位、但舊引擎或暫存資料夾刪不掉時也是 `2`，訊息列出要手動刪除的資料夾。

### 手動安裝到新的專用目錄

不使用 `install.bat` 時，以**已提升權限的 Windows PowerShell 5.1** 執行。以下只適用於全新安裝；不要解壓到既有的引擎或進行中的部署上。根目錄預設為 `C:\WinCD`，要放在其他位置見下方「自選根目錄」。

```powershell
$root = 'C:\WinCD'
Unblock-File ".\wincd-engine-$version.zip"
New-Item $root -ItemType Directory -ErrorAction Stop
icacls $root /inheritance:r /grant:r '*S-1-5-32-544:(OI)(CI)F' '*S-1-5-18:(OI)(CI)F' '*S-1-5-32-545:(OI)(CI)RX'
if ($LASTEXITCODE -ne 0) { throw '無法限制安裝目錄的權限。' }
Expand-Archive ".\wincd-engine-$version.zip" -DestinationPath $root
icacls $root /setowner '*S-1-5-32-544' /T /C /Q
if ($LASTEXITCODE -ne 0) { throw '無法把安裝目錄的擁有者設為 Administrators。' }
& "$root\engine\wincd.ps1" status
```

zip 內只有一個最上層的 `engine` 資料夾。已提升權限時建立的檔案，擁有者可能是你的帳號而不是 Administrators；`/setowner` 那一行讓它們歸 Administrators 所有（和 `install.bat` 相同），BAT 操作包才會執行這個引擎。安裝後的配置：

```text
C:\WinCD\engine\  執行檔案，入口是 wincd.ps1；ManualKit\ 是 BAT 操作包的建置程式（見第 5 節）
C:\WinCD\inbox\   收件資料夾：要換版的 release zip，需要時才建立
C:\WinCD\data\    資料根目錄：狀態、紀錄、備份與 release 保留區，需要時才建立
```

不要讓不受信任的使用者能寫入 `engine`。release 內容與備份可能含有明文帳密；不要公開或廣泛分享 `inbox` 與 `data`。

`status` 是唯讀的：不建立資料目錄、不修復權限，也不設定 IIS。變更指令會自行準備需要的目錄，並檢查各自的前提。

### 自選根目錄

根目錄就是 `engine` 資料夾的上一層。引擎沒有安裝設定檔或安裝參數：它一律使用自己所在的根目錄，`engine`、`inbox`、`data` 都在它底下。例如要放在 D 槽，就在上面的指令把 `$root` 設為 `'D:\WinCD'`：

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
| `demo/wincd-kit-wincd-demo-v1/`、`demo/wincd-kit-wincd-demo-v2/` | 為 `wincd-demo` 與裝在 `C:\WinCD` 的引擎預先建好的 BAT 操作包（第 5 節），每個 demo release 一個，`release.zip` 已在裡面，在測試機試用。 |
| `reference/order-api/` | 逐欄註解的完整部署設定參考，並列出 IIS 站台、程序服務、檢查步驟與 F5 模式的替代寫法。它指向 `D:\Tomcat9`、`C:\nginx` 與 IIS 監測站台，不能在乾淨的測試機上直接換版；撰寫自己的部署設定時從這裡複製。 |

demo **只在測試機使用**：它以管理員身分執行，讓 `C:\WinCD-demo\site` 與每次換版的 release 一致（完整同步會刪除 release 中沒有的檔案，保留清單除外），而且 `wincd-demo` 專案的紀錄會留在 WinCD 的 `data`，WinCD 沒有刪除專案紀錄的指令。

## 5. 選用：BAT 操作包

不想使用 TUI 或輸入指令的部署人員，可以使用為一個部署環境（一台主機上的一個專案）預先建好的 BAT 操作包：雙擊查看狀態、換版、Go-Live（接 F5 時）、查看最後狀態，以及例外的退版與復原。BAT 只呼叫這台主機已安裝的 `wincd`，不帶 `-Yes`、`-Json` 或 `-Force`：摘要、大寫 `Y`、交易與 exit code 都是 WinCD 自己的。CI/CD 不使用操作包，照常直接呼叫 `wincd`。

操作包為一個 Release 建置：`release.zip` 在操作包裡，它的 SHA-256 在建置時固定，`2-Deploy.bat` 自己核對、不詢問任何值。**交給部署人員（建議）：打包工具。** 平台維護者解壓 `WinCD-Packager-<版本>.zip`，把 Release 放在 `Pack.bat` 旁邊命名為 `release.zip`，雙擊 `Pack.bat`（不需要系統管理員權限、離線）。它以 WinCD 自己的唯讀 Plan 檢查 Release，寫出一個交接 ZIP：一頁的 `00-先讀我.txt`、`1-Install\`（與發行的檔案逐位元相同的 `install-fast.bat`、引擎安裝包與 `SHA256SUMS`）、把操作包連同 Release 放進 `D:\WinCD-Kits\<專案>` 的 `2-Prepare-Kit.bat`，以及操作包。部署人員解壓後依說明依序雙擊。用法見打包工具 ZIP 裡的 `README.txt`。

**直接建置。** 建置程式隨引擎安裝在 `engine\ManualKit\`（v0.4.1 起固定 SHA-256；v0.4.0 附的是要求貼上 SHA-256 的舊流程）。以已提升權限的管理員在主機上執行：

```powershell
& D:\WinCD\engine\ManualKit\New-WinCDManualKit.ps1 -Project order-api -F5Mode None -ReleasePath .\release.zip -OutputPath "$env:TEMP\wincd-kit-order-api"
```

- `-Project`：這個部署環境的專案，必須是 Release 的 `Project`。建置程式以 WinCD 的唯讀 Plan 檢查 Release：不通過，或 `Project`、`F5.Mode` 與參數不同時不建置。
- `-ReleasePath`：必填，這個操作包要換版的 Release ZIP：複製成操作包裡的 `release.zip`，SHA-256 固定在 `wincd-kit.psd1`。
- `-F5Mode`：這個部署環境 Release 的 `F5.Mode`（必須相同）：`None`、`IisEndpoint` 或 `LegacyFile`。接 F5 時多一個 `3-Go-Live-Up-F5.bat`；`-IncludeDownF5` 另加只改導流、不停服務的 `Manual-Down-F5.bat`。
- `-Reason`：選填，每次操作帶給 WinCD 的固定 Reason（例如變更單號）。
- `-EngineEntry`：選填，預設是建置程式所在的引擎（上例是 `D:\WinCD\engine\wincd.ps1`）。
- `-OutputPath`：必填，已存在時拒絕；不能在 `engine` 裡（更新引擎時整個 `engine` 會被換掉）。建在只有你能寫入的位置，例如上面的 `$env:TEMP`。

結果列出每個檔案的 SHA-256。交付依操作包內 `README.txt` 的「受保護的交付位置」：一般由交接 ZIP 的 `2-Prepare-Kit.bat` 完成；手動交付時先建立只有系統管理員能修改的資料夾，再複製整個操作包、把擁有者設為 Administrators，並依建置結果核對每個檔案的 SHA-256。操作包只從這樣的資料夾執行，否則在要求 UAC 之前就拒絕。每個新的 Release 都重新建置並整個取代操作包；BAT 與啟動程式逐位元相同，只有固定的 SHA-256 與 `release.zip` 不同。不要只換 `release.zip`：與固定的 SHA-256 不符時 `2-Deploy.bat` 會拒絕。

更新引擎不影響已交付的操作包：它帶著自己的啟動程式，只呼叫已安裝的 `wincd`。v0.4.0 建置、要求貼上 SHA-256 的操作包在 v0.4.1 照常使用；要改用固定 SHA-256 的流程，重新建置並整個取代它。

範例包的 `demo/wincd-kit-wincd-demo-v1/` 與 `demo/wincd-kit-wincd-demo-v2/` 是為 `wincd-demo` 建好的操作包，可以在測試機試用（見範例包的 `README.md`）。

**限制：** BAT 操作包與交接 ZIP 的 `2-Prepare-Kit.bat` 尚未在真實的 UAC 視窗、檔案總管雙擊與 RDP 主控台完成人工驗收；自動測試以替身模擬 UAC。正式使用前，先在測試機走一次。

## 更新

更新前，先結束所有 WinCD TUI 與工作，並處理完未完成的復原作業。核對新安裝包與發行說明中的相容性，然後整個換掉 `engine` 資料夾；`inbox` 與 `data` 保持不動。不要逐檔覆蓋，也不要假設舊版引擎能讀取較新的部署設定或交易狀態。用 `install.bat` 更新見第 2 節；手動更新：

```powershell
$root = 'C:\WinCD'
Unblock-File ".\wincd-engine-$version.zip"
Remove-Item "$root\engine" -Recurse -ErrorAction Stop
Expand-Archive ".\wincd-engine-$version.zip" -DestinationPath $root
icacls "$root\engine" /setowner '*S-1-5-32-544' /T /C /Q
if ($LASTEXITCODE -ne 0) { throw '無法把引擎的擁有者設為 Administrators。' }
& "$root\engine\wincd.ps1" status
```

公開版本不承諾與更早的開發快照相容或可遷移。選用的 IIS handler 有變更時，可能需要另外規劃監測站台的安裝或更新；一般執行期操作不會自動修復或升級它。
