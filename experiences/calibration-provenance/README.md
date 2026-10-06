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

保存軸若與明確的編譯/實際時序證據矛盾，也可能是軟體缺陷；不能將成功保存當成它必然正確。2026-10-07的[Q1 Length Rabi反向sweep案例](../rabi-fit-validation/README.md#已量化軸也可能有保存實作缺陷)保留原raw、另存有來源的compiler重建軸，並區分顯示負值與實際pulse長度。這是經授權開發排查確認的例外，不是鼓勵常態二次量化或由目前library猜旧Run。

## 逐 flux 點使用獨立 context

2026-10-05 使用者的專家建議：逐點校準與 coherence 量測，每個 flux 點建立一個獨立 context，可 clone 前點繼承所需內容。這能避免後點 writeback 覆蓋前點的校準狀態；它是流程建議，不表示 clone 的頻率與 pulse 在新點已驗證。

相鄰移動電流並核對 actual current 後，建立帶點號／工作點識別的 context，記錄 clone 來源，再重校 RO／qubit frequency 與 amplitude pulse。開始 Run 前仍核對展開的 cfg、local overrides 與輸出目的地。各點 raw、分析模型及單位連回 CSV；context 名稱不能取代 raw 中實際電流。若流程中途才開始分點 context，保留已保存檔案原路徑，明列哪些校準來自建立前的來源，不改名冒充新量測。

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

### 固定點與 flux map 峰位不一致

先用保存的實際軸、同一局部窗口與同一複數模型比較，避免把全頻自動選峰、背景模型或功率展寬差異誤認成頻率漂移。核對 raw 中完整 pulse、readout、等待、averages 及時間；只比 tab 當前設定不足以還原舊 Run。

可以依成本逐步控制：回到相同工作點重測；分別比較頻率步距、掃描窗口與 reps；用 flux adapter 的單一電流點；再作包含該點的短正反向 map。這些控制能縮小候選解釋，未重現差異卻不能證明原資料錯誤，也不能由正反向一致就排除所有磁滯或 settling 問題。需要更可靠的主圖時，以新條件重掃並保留來源，不對舊 map 套用未證實的常數修正。

2026-10-05 Q1 的5.8mA案例：早期1–2GHz map局部複數fit約1575.53MHz，同pulse／RO的固定點、窗口／步距／reps控制及單點adapter、短正反向map約1568.5–1569.3MHz。約7MHz差異沒有被控制重現，原因保持未知。任務來源為 `.agent_state/measurement-tasks/20261005-q1-twotone-fluxdep/compare_nqz1_repeat.py` 與 journal 17:20–17:49；原始flux檔 `_6`、`_9`–`_11`及freq檔 `_11`、`_22`–`_25`。這是比較流程的反例，不是平台固有偏移量。

## 同名欄位也要核對單位

2026-10-05的flux與30點模擬任務中，twotone/freq保存的Frequency欄以MHz表示，twotone/flux_dep的同名欄則以Hz表示，兩者單位欄都空白。先比對Run cfg、realized axis及數值範圍，再轉換單位。不能把某一adapter的schema假定套到所有檔案；也不能因為這次觀察就認定未來版本永遠如此。

分析輸出要保留raw路徑、方法與轉換。批次處理遇到schema或解析失敗，停止依賴該結果的步驟。缺少搜尋中心時不能默默退回GUI預設值，否則可能在錯的頻段取得看似完成的資料。

## 定期清理已保存的工作頁

相同adapter連續量測時重用tab。只在需要並排對照或保留獨立狀態時增加工作頁。使用者於30點任務要求定時清理，agent在批次間核對未使用tab的完整artifact狀態，補存後以不丟棄資料的方式關閉13個舊頁。

曾有last_saved_path不代表目前內容已保存。看到unsaved_changes時，先依當前保存契約處理並核對終態。關閉前確認raw、必要圖片與來源紀錄齊全；canonical圖可能已被重新分析覆寫。具體guard、save和close步驟仍以當前工具契約為準，不照抄案例的tab ID。

## 模擬反例

[2026-10-05 simulate 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 中，echo 的 π phase local override 保留了舊 frequency。Run 前讀取完整 cfg 才發現，隨後把 frequency 和 length 一起更新，最終 raw 確認使用新值。

同一次任務的 Rabi 請求掃到 8 µs，實際終點約 7.3203 µs。最終 echo 請求 100 µs，raw 終點約 99.0513 µs。這些差異提醒我們核對實際軸，不是本工具或其他設備的固定換算比例。

最後 echo tab 的單條預設 fit 約 10.8 µs，而經相位差分支持的有效 T2 echo 約 14.35 µs。直接接受目前 tab 的候選會把兩種分析混在一起。模擬案例的數值不能作為真實硬體的設定或結果門檻。
