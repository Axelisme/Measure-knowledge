# T1 時間窗口與 fit 驗證

## 何時使用

T1 的衰減時間與掃描窗口接近、尾端尚未穩定、參數誤差大，或 repeat 不一致時，先核對窗口與模型，再決定重新分析、補量測或接受結果。正常曲線也要做同樣的基本核對，fit 完成不等於校準可信。

## 判讀與下一步

1. 核對時間座標、單位、pulse 與讀出條件。窗口需能區分初始衰減與尾端基線；尾端仍有趨勢、基線受其他效應影響，或窗口只看到很少衰減時，T1 與 offset 可能難以區分。先看原始資料與 residual，再依當次授權決定延長窗口或補基線，不用固定倍數作通用門檻。
2. 對照 fit 與資料。單指數模型假定一個主要衰減尺度；系統性的彎曲、尾端偏差或振盪要保留為模型不適用的線索。改模型或 skip/mask 時記錄理由，先在同一 Run 重新分析；不是看到較低 error 就證明新模型正確。
3. 同時看 T1 的 stderr、窗口與模型敏感性。小 stderr 是給定模型與資料下的估計，不包含模型偏差。改窗口或合理模型後 T1 明顯變動時，先辨別來源，再做 writeback。
4. 在相同工作點與可比較條件下 repeat，記錄每次的窗口、分析選項與不確定性。Repeat 一致能支持重現性，不能排除共同偏差；不一致時先核對漂移、pulse、讀出及基線，不能只挑最接近預期的一次。

## 慢尾端的決策流程

```text
單 exp 的 residual 或 fit-window 敏感性有結構？
  否 → 相同設定 repeat；保存有效 T1 與模型條件
  是 → 同 raw、同一線性 IQ projection 比較窗口與候選模型
         ├─ 雙 exp 參數不可辨認 → 不把兩個時間常數當成物理通道
         └─ 結構穩定 → 選一個控制：等待時間／pulse 校準／零 drive 背景
                          ↓
         背景也有相同 delay 趨勢？
           是 → 核對共模假設、匹配軸及漂移，再分析複數 IQ 差分
           否 → 在本對照精度內，零 drive 背景不足以解釋尾端
                          ↓
         仍有多時間尺度 → 報 effective T1 與模型敏感性，保留機制未知
```

零 drive 對照保留原 pulse 時長與 sequence 時序，只把 drive gain 設為零；其餘 delay 軸、flux、readout、frequency 和 recovery 條件需匹配。這不保證樣品處於純 ground state，不能稱作已驗證的基態 reference。平均數不同會改變差分噪音；先比較複數 IQ，再用由 signal 確定的一個固定線性 projection 處理 signal 和 reference，不要各自 PCA／取 magnitude 後相減。

參考曲線平坦只能限制該對照可見的背景，不能排除激發才出現的效應、非線性讀出、熱人口、多能級動力學或漂移。雙 exp 擬合改善也不能識別機制。記錄相對幅度、時間常數、誤差與參數相關性；窗口不足時尤其避免採用未收斂的慢分量。

[真實案例](../coherence-bringup/cases/real-integer-20261005/README.md) 在改長 recovery／窗口、改短 pulse 及 zero-drive 差分後仍有慢尾端。部分設定同時改變，證據支持「在測過的條件下仍存在」，不構成排除各機制的單因素實驗。最後保留單 exp 有效值，沒有把 GUI 雙 exp 第一個分量自動寫回唯一 `t1`。

## 最初幾點陡降時，先解析時間結構

長窗口不等於早期解析度足夠。若第一、第二點差異很大，且後續有慢尾，先補同一 flux／pulse／readout／recovery 條件的密集短時間掃描；它回答的是早期結構，不取代原長窗口的基線證據。若解析出振盪，不應只換成雙指數或刪掉前幾點來強迫得到單一 T1。排查模型時保留兩份 raw、實際量化時間軸及相同 IQ 投影。

2026-10-05 Q12_2D[10]/Q1 真實 p14（2.2407 mA、4432.835194 MHz）在 0–150 µs／181 點的 T1 前段陡降。補 0–10 µs／201 點後看見約 0.6–0.7 µs 週期的衰減振盪。長、短窗單 exp 條件式估計分別約15.37與2.75 µs，不能當成兩個已辨認的壽命；本次 CSV 的單一 T1/error 留空，status 標記 measured_nonexponential_t1，raw 和條件式 fit 仍保存。振盪與鄰近光譜弱峰同時存在，不足以唯一識別耦合、其他躍遷或驅動誤差等機制。

證據：repo-local `.agent_state/measurement-tasks/20261005-q1-40flux-coherence/p14_t1{,_dense}.json`、對應 `_provenance.json`／`_analysis_fit.png` 及 `p14_reviewed.json`。這是單一工作點的診斷案例，不是全域量測模板。對結果模型已明顯失效的點，空白加原因比填入未限定的 fit 數字更能保留資料意義；空白不表示未量測或零壽命。

## 擬合品質指標

從目前 MCP estimate 的 `quality.fit` 或 GUI analysis summary 的 `fit_quality.fit` 讀 `r2`、`normalized_residual_rms`、`relative_parameter_errors` 及 `invalid`。這些數值只描述真正送入 fit 的樣本，需連同 skip/mask 與分析條件解讀。

`null` 要讀對應的 invalid reason，不能當作零誤差。Optimizer 參數的相對誤差不包含所有衍生值的不確定性。目前版本未提供品質時，用圖、已有 fit error 與 repeat 判讀，不自行捏造 R²。

引用門檻時附設備、模型、資料範圍及噪音條件。單一 R² 不證明模型或物理參數正確，低品質也不由工具自動禁止 accept。Agent 依結果用途與證據決定接受、保留未知或補驗證。

## 窗口與 repeat 的模擬案例

[2026-10-05 simulate 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 在同一最終工作點、可比較 pulse 與讀出條件下，取得兩個窗口的 T1：

| 實際最大 delay，µs | T1，µs | Fit stderr，µs |
| ---: | ---: | ---: |
| 99.7489 | 20.1276 | 0.0414 |
| 149.7396 | 20.0225 | 0.0327 |

兩次都能看到尾端，結果支持約 20 µs 的有效 T1。差值約 0.105 µs，約為合併 stderr 的兩倍。這不支持把單次 0.03 µs 的 stderr 當成總準確度。

這是兩個獨立 Run 且窗口不同，不能把差異唯一歸因於窗口。要隔離窗口效應，先對同一份長窗口 raw 截取不同範圍重新 fit；要估計 repeat 差異，則固定條件補量測。該案例的時間、窗口與 R² 都不是通用閾值。

T1 初估也能幫助選擇 Rabi 的 recovery-time 對照，見 [Rabi 等待時間](../rabi-fit-validation/README.md)。T1 數值本身不證明已充分 reset。

## 來源與限制

本條包含模擬與真實硬體案例，均不建立通用閾值。[Rabi mock 案例](../rabi-fit-validation/README.md) 保存三次約 20 us 的 T1 數值與來源；它們只屬於該 mock 工作點，不是真實裝置門檻。真實案例另有模型敏感性與未識別的慢尾端；repeat 也不能排除共同模型偏差。

當次目標、硬體限制與預算優先於歷史經驗。若補窗口或 repeat 超出授權，保留缺口並求助，不為完成 fit 擴張量測範圍。
