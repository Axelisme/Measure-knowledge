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

## 同名欄位也要核對單位

2026-10-05的flux與30點模擬任務中，twotone/freq保存的Frequency欄以MHz表示，twotone/flux_dep的同名欄則以Hz表示，兩者單位欄都空白。先比對Run cfg、realized axis及數值範圍，再轉換單位。不能把某一adapter的schema假定套到所有檔案；也不能因為這次觀察就認定未來版本永遠如此。

分析輸出要保留raw路徑、方法與轉換。批次處理遇到schema或解析失敗，停止依賴該結果的步驟。缺少搜尋中心時不能默默退回GUI預設值，否則可能在錯的頻段取得看似完成的資料。

## 定期清理已保存的工作頁

相同adapter連續量測時重用tab。只在需要並排對照或保留獨立狀態時增加工作頁。使用者於30點任務要求定時清理，agent在批次間核對未使用tab的完整artifact狀態，補存後以不丟棄資料的方式關閉13個舊頁。

曾有last_saved_path不代表目前內容已保存。看到unsaved_changes時，先依當前保存契約處理並核對終態。關閉前確認raw、必要圖片與來源紀錄齊全；canonical圖可能已被重新分析覆寫。具體guard、save和close步驟仍以當前工具契約為準，不照抄案例的tab ID。

## 這次案例的反例

[2026-10-05 simulate 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 中，echo 的 π phase local override 保留了舊 frequency。Run 前讀取完整 cfg 才發現，隨後把 frequency 和 length 一起更新，最終 raw 確認使用新值。

同一次任務的 Rabi 請求掃到 8 µs，實際終點約 7.3203 µs。最終 echo 請求 100 µs，raw 終點約 99.0513 µs。這些差異提醒我們核對實際軸，不是本工具或其他設備的固定換算比例。

最後 echo tab 的單條預設 fit 約 10.8 µs，而經相位差分支持的有效 T2 echo 約 14.35 µs。直接接受目前 tab 的候選會把兩種分析混在一起。模擬案例的數值不能作為真實硬體的設定或結果門檻。
