# 從未驗證的設定到可交付的 coherence

## 何時使用

從空白 context 開始，或接手一組缺少來源的設定，要決定下一個量測及何時回頭校準時使用。本流程以有 flux 調控、色散讀出與微波控制的 qubit 為背景。其他架構需重新核對觀測量與 pulse sequence。

[2026-10-05 simulate 案例](cases/sim-integer-20261005/README.md) 示範一次完整走法。案例只驗證模擬環境中的決策與分析，沒有驗證真實接線、功率、安全範圍或 gate fidelity。下列硬體前置檢查是移轉時的要求，不是這次完成過的硬體測試。

[30點integer-to-half模擬案例](cases/sim-30flux-20261005/README.md) 補充多工作點校準、人工detune echo、取樣稽核與低訊號補測。它保留失敗模型及未完成zigzag的限制，沒有把插值當量測。

## 真實硬體開始前

先取得目標、結果用途、可用時間與停止條件。模擬模式的授權不包含切換真實硬體。

- 確認實際連線、裝置模式、channel、LO／IF 與 sideband。確認線路衰減、放大器和 ADC 的允許範圍。數位 gain 不能直接當成樣品端功率。
- 取得 flux 的物理單位、允許範圍、ramp、settling 與 output 切換程序。缺少其中會影響安全的條件就停下詢問。FakeDevice 的 native 座標及 output off 行為不能移植。
- 確認 qubit／resonator 可搜尋的頻段、已知禁區、讀出與 drive 的功率限制。弱訊號不構成任意加大 gain 或擴大掃描的理由。
- 在允許的設定下核對讀出線性區、ADC clipping、IQ 對比、trigger 與 integration window。記錄漂移、加熱或 leakage 的觀測與停止條件，不以模擬中沒有異常作保證。
- 核對現有校準的工作點、時間、來源和沿用情況。空白 context 不等於設備未知；有數值也不等於已驗證。

任何階段若需要越過授權範圍、未知 output 狀態或無法解釋的操作狀態，停止相依操作並求助。

## 決策樹

```text
安全條件、授權與目前狀態可確認？
  否 → 只整理已有資料，取得缺少的條件
  是 → 讀出訊號及 integration window 可用？
         否 → lookback／時序與讀出鏈檢查
         是 → one-tone 粗搜尋，再窄掃解析共振
                ↓
       flux 分支與目標工作點有依據？
         否 → 受限 flux map → 分支辨別 → 局部 qubit spectroscopy
         是 → 在目前 flux 核對 readout 與 qubit frequency
                ↓
       Rabi 可辨認週期、第一個峰與 pulse 候選？
         否 → 先查實際軸與模型，再決定補窗口、取樣或訊號
         是 → 暫定 pulse → T1 初估 → 檢查 recovery wait
                ↓                       ↓
       若等待時間影響 Rabi，回到 pulse 校準後再測 coherence
                ↓
       T1／Ramsey／echo 的窗口、baseline 與 residual 可解釋？
         否 → 同 raw 模型比較，必要時 reference／phase cycling
         是 → repeat、窗口敏感性、設定來源核對
                ↓
       保存 raw、分析方法、未解限制 → 只寫回有依據的校準
```

這是有回路的流程。更動 flux、頻率、pulse 或讀出後，回到受影響的檢查，不把舊的成功狀態一路沿用。

## 每一步需要什麼證據

| 階段 | 能繼續的依據 | 不足時先做什麼 |
| --- | --- | --- |
| Lookback | 可辨認回波到達時間，integration window 避開不想積分的瞬態，且訊號未 clipping | 檢查 trigger、鏈路與讀出；arrival 和 trigger offset 分開記錄 |
| One-tone | 窄掃有足夠樣點描述共振與背景，候選不只是粗掃單點 | 在核准頻段加密，不從未解析的 linewidth 推論物理參數 |
| Flux 與 two-tone | 候選分支有依據，局部譜線可追蹤，工作點兩側的頻率可比較 | 讀 [flux 與 spectroscopy](../flux-spectroscopy-validation/README.md)，不要只採 auto-fit |
| Rabi | 週期、第一個峰、模型與 π／π2 定義一致 | 讀 [Rabi fit 驗證](../rabi-fit-validation/README.md)，先辨別模型與窗口 |
| T1 與 recovery | 衰減與尾端基線可區分，改等待時間的影響有檢查 | 讀 [T1 fit 驗證](../t1-fit-validation/README.md)，回查 pulse；不要固定套用案例的等待時間 |
| Ramsey | 實際取樣能解析 fringe 和 envelope，沒有未處理的 baseline 趨勢 | 核對 delay 定義、人工 fringe 與真實 detuning，讀 [背景辨別](../coherence-background-validation/README.md) |
| Echo | delay 定義明確，pulse phase 與背景處理有依據 | 先確認時間是總 free evolution 或半段 delay，再選模型或互補相位 |
| 交付 | raw、實際設定、模型、repeat 和限制可追溯 | 讀 [設定與來源核對](../calibration-provenance/README.md)，不把目前 GUI fit 自動當成最後答案 |

若 coherence 太短而無法辨認多個 fringe，不能只提高 fringe frequency。先核對實際取樣間隔與 pulse 頻寬，再選可辨認的 detuning 和窗口。平均只能改善隨機噪音，不能修正 aliasing、錯誤設定或系統性 baseline。

## 如何花下一段預算

已有 raw 足以比較模型時，先離線分析。這能區分 fit 假設，不增加設備占用；但不能補回缺少的時間窗口或參考訊號。

需要補量測時，優先選能區分候選原因的一個變更。例如固定分析模型比較 recovery wait，或固定 pulse 比較互補相位。若為了找訊號同時改 gain、取樣與 averages，要記錄這是搜尋，不是單因素因果驗證。

預算不足時，交付已驗證的部分及缺口。不能用單次小 stderr 取代 repeat，也不能為了讓 echo 大於 Ramsey 而挑結果。兩者的大小關係不是通用驗收門檻。

## 最小交付

保留工作點及其單位、校準來源、實際 pulse／readout／delay、raw 路徑、模型與資料轉換。報告 fit stderr、repeat 差異和窗口／模型敏感性，不把三者混成同一種誤差。若只支援有效衰減時間，就用這個名稱，不宣稱已識別噪音機制或 gate fidelity。
