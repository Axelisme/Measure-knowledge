# 確認校準真正用在這一次量測

## 何時使用

更改 pulse library、重用 tab、重新分析、等待逾時或準備 writeback 時使用。此條處理設定與證據的來源，不取代目前 GUI／MCP 的 live guide。工具名稱、欄位與接受行為應依當次公開契約確認。

## 比較四份資訊

| 資訊 | 要回答的問題 |
| --- | --- |
| 校準來源 | 值在哪個工作點、何時、用哪份 raw 與模型得到？是沿用還是重新驗證？ |
| Run 前完整設定 | 此 tab 最後展開的 frequency、gain、length、phase、readout、wait 和 sweep 是什麼？有沒有 local override？ |
| Run 的保存資料 | 實際軸、點數、單位、pulse 與裝置值是否符合意圖？ |
| Writeback 目的地 | 要寫哪個 context／library 項目？是原始 fit、修正分析還是彙整值？ |

名稱相同不表示內容相同。引用 library 的 module 在局部編輯後可能保留舊設定；只看到 ref 名稱不足以證明最新校準已生效。更改 flux 或頻率後，核對所有相依 pulse，不只更新眼前的 π pulse。

請求軸也不等於實際軸。硬體時序量化可能改變 step、終點與可解析的週期數。擬合使用保存的實際軸，並核對它代表的是 delay、pulse length 或總 evolution time。若端點差異會使窗口不足，回到量測設計，而不是改圖的座標標籤。

## 等待、保存與接受是不同事件

等待逾時只表示 client 沒有拿到完成回覆。先核對 operation／execution 及 GUI 狀態，不能以 timeout 作為再次 Run 的理由。不知道是否已執行時，記錄未知並停止相依操作。

Run 完成不保證全部分析和檔案保存成功。交付前確認 raw 與必要圖片的實際持久路徑。Preview、reserved 路徑和 session handle 不能替代已保存產物。

重新分析可能覆蓋 canonical 圖。模型比較前保存舊圖與選項，保留 raw 不動。每個衍生結果記錄輸入、資料轉換、fit 模型、窗口、固定參數與單位。

Writeback 前讀目的地和完整候選集合，不推測未勾選候選一定不會寫入。離線修正估計與 GUI 預設 fit 要分開標記；目前 tab 顯示的數字不一定是應採用的值。若 context 只能存一個 scalar，另存分析來源與限制，不能只留下無來源的平均數。

## 真實量測的操作反例

[Q1 真實案例](../coherence-bringup/cases/real-integer-20261005/README.md) 留下幾個公開工具層面的教訓。以下是當次契約的觀察，重用前仍讀 live schema：

- Recipe 可能重新建立 cfg。需要保留自訂 recovery wait 等欄位時，確認 recipe 是否支持；若不支持，用完整 cfg observation → 編輯 → Run → 保存 → 分析的公開 RPC 流程，不能先改 cfg 再假定 recipe 保留它。
- Context、tab、SoC、device 的完整觀察與 cfg revision 是不同資源的 guard。Run 完成後結果也可能改版，保存前要重新核對結果；busy／stale 回覆後先讀現況，不盲目重送。
- MCP operation handle 與 GUI result 的 source operation token 不應憑數字相近或同名就互換。帶 result token 保存時，必須用對應公開契約取得的 token；不能把等待用的 handle 猜成結果 ID。
- 保存回覆可能只是預留路徑與 handle。等保存成功，再開始依賴保存完成的分析或下一輪；用 artifact 的 saved 狀態確認。Raw RPC 與 recipe 的自動保存／分析範圍不同。
- 裝置 cached snapshot 可能沒有反映 scan 結束後的實際值。Flux scan 的終點處置需事先確認；後續透過公開裝置流程設定工作點、等待及核對，不能因快取仍顯示舊值便假定已回程。
- 原生分析圖檔名可能被下一輪覆蓋。每次模型／條件比較先保存來源對應的圖與結果，最後 tab 的 Run 也可能是 background control，而非報告採用的 signal Run。

以上不是允許跳過 guard、繞過 MCP 或忽略錯誤的理由。當次保存資料中應同時記錄 requested 與 actual 軸；執行時設定下限與量化座標的區別，見 [Rabi](../rabi-fit-validation/README.md)。實驗提供的軸已量化時不再二次量化，也不僅憑座標與名義下限的小差異判定 Run 無效。

## Pulse 長度上限的層次

長 spectroscopy probe 也可能有單指令 pulse duration 上限，不能只看波形記憶體或 coherent pulse 的下限。2026-10-05 Q12_2D[10]/Q1、QICK0.2.394、ch2 const pulse 的 runtime 明確拒絕1000us（599040cycles，錯誤指出超過2**16），而100us可完成；當時SoC公開資訊的fabric clock為599.04MHz，故單pulse約109us是該配置的上限尺度。這是當次硬體／韌體觀察，不是所有channel或waveform通用常數。

設定遭runtime拒絕後，核對operation失敗與result來源，不能把tab殘留上一輪result另存成新量測。回到已驗證範圍並比較gain、averages或其他公開支持的序列；不假定GUI可輸入的數字代表硬體一定可執行，也不把max envelope size誤當const pulse duration限制。失敗來源記於當次twotone任務journal的16:03條目（op146），沒有新raw。

## Repeat 與條件比較的標記

### Reps 與 rounds 的成本／觀察取捨

2026-10-05 使用者說明本平台的 reps 為硬體掃描平均、rounds 為軟體掃描，liveplot 以 round 更新；主要平均數交給 reps 通常較快，需要觀看過程時再分配一些 rounds，例如10。使用者的實務經驗是長時間量測 reps 超過30000可能報錯，控制10000內通常可行。這是本平台的操作經驗，不保證其他韌體或任何資料量皆安全，也不能把兩個門檻當成已量測出的通用硬體常數。

規劃時分開記錄總平均數與其分配。保持reps×rounds不變仍可能改變漂移平均、更新頻率和軟體開銷；若要歸因於某個設定，核對實際條件和時間戳。只有一個round時，沒有即時曲線更新不代表Run停住；按公開operation進度判斷。Flux外圈每點的進度仍可幫助觀察，但不能在未完成round時假定該點已有完整平均。

嚴格 repeat 要保持工作點、pulse、readout、等待、actual sweep、人工 fringe 及分析方法可比較。為了改善結果而一起改 pulse length、gain、drive frequency、窗口或平均數，應標成「條件比較」；它可以支持新條件下仍有某特徵，不能把差異唯一歸因於其中一項。

每個 scalar 校準連到產生它的 raw、模型與生效的 module。若只更新 MetaDict 而 library 存的是固定 frequency，下一次 Run 可能仍使用舊 frequency；更新後重新展開相關 π／π2 等模組核對，不只看 parameter 表。

## 模擬反例

[2026-10-05 simulate 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 中，echo 的 π phase local override 保留了舊 frequency。Run 前讀取完整 cfg 才發現，隨後把 frequency 和 length 一起更新，最終 raw 確認使用新值。

同一次任務的 Rabi 請求掃到 8 µs，實際終點約 7.3203 µs。最終 echo 請求 100 µs，raw 終點約 99.0513 µs。這些差異提醒我們核對實際軸，不是本工具或其他設備的固定換算比例。

最後 echo tab 的單條預設 fit 約 10.8 µs，而經相位差分支持的有效 T2 echo 約 14.35 µs。直接接受目前 tab 的候選會把兩種分析混在一起。模擬案例的數值不能作為真實硬體的設定或結果門檻。
