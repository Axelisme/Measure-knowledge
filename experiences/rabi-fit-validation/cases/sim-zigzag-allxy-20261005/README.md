# Zigzag／AllXY mock 交叉檢查

## 條件與來源

2026-10-05，Qubit-measure-gui 的 mock SoC，half flux native=0，context coh30_p29。沿用前輪 Rabi 的 const pulse，frequency 581.84849 MHz，π length 1.256784 µs，π/2 length 0.631288 µs，gain 0.3。Readout 沿用該點的 readout_dpm。1D 實驗為 100 reps、1200 rounds、relax 150 µs。

原 task 為 `.agent_state/measurement-tasks/sim-zigzag-allxy-20261005/`。RESULTS.md 保留完整方法與 GUI 收尾，summary.json 記錄 14 份已保存 raw 的路徑、SHA-256、條件及比較指標。本目錄的 summary.json 與 comparison.png 是該 task 的副本。量測階段沒有讀 implementation 或模擬器真值。後續使用者另行授權以 DEVELOPMENT 檢查實作；其結果獨立記錄於下節，不回填為量測校準。

## 直接觀察

Zigzag 的 π 與 π/2 都有交替偏離。AllXY 基準 gain=0.3 的 GUI power_err 為 15.082%，回到基準重測為 15.090%。兩個 pulse 同改 gain=0.285、0.315、0.33，power_err 分別為 23.619%、6.298%、1.727%。這說明新實驗能對 gain 變化產生可辨認的反應，不能由此斷言 Rabi 模型或真實 pulse 的誤差來源。

π Zigzag gain scan 得到候選 0.3209119。以可保存的一維 Zigzag 重測，並把同一 gain 用於 π/2，兩者偏離均縮小但仍不平坦。AllXY 在此 gain 的 power_err 為 3.439%，高於 gain=0.33 的值；gain=0.33 的 Zigzag 卻已反向偏離。兩個實驗沒有給出一致的最佳候選，未寫回校準。

![共用線性 IQ 參考的比較](comparison.png)

圖的共同尺度由基準 AllXY 的 II 與最後四組 excited pair 建立，所有 Run 使用同一 signed IQ projection。Y 軸是 normalized signal，沒有獨立人口校正，不當成激發機率。Zigzag 候選 Run 同時更改 X90 preparation，並非只更改 repeated π 的單因子比較。完整 gain 條件見 summary.json。

## 可遷移判斷與限制

Rabi fit 與週期吻合，不能替代 sequence-level 交叉檢查。調整 gain 後應重測 Zigzag 和 AllXY，保留方向反轉及兩種指標不一致的結果。量測階段保留 preparation、relaxation、讀出正規化和模型差異等解釋，未由當時資料單獨辨別。後續離線排查支持時序與耗散模型缺項，見下節；這仍不代表所有真實器件的不一致都由同一原因造成。

AllXY guide 把 power_err／detune_err 定義為模型的 mean state deviation，不是 gate infidelity。Gain=0.33 的同一 raw，fit_ge=false／true 得到 power_err 1.727%／3.746%，顯示分析假設也會改變數值。不同定義的指標不可直接換算或混稱誤差率。

這是 mock 案例。除了基準 AllXY 重測，各候選只有一個 Run，沒有可信的 gate-fidelity 結論，也沒有完成 π／π2 獨立校準。

## 後續 DEVELOPMENT 排查

使用者另行授權「我允許你檢查實作，排查原因」。檢查版本 7035402e97cd51cf6b3beb56509f2df43c83bdd8。完整交接報告位於原 task 的 diagnosis/REPORT.md。這一節是程式與離線模型證據，不是新硬體量測。

現行 Run 用 ComputedPulse，短 pulse 後面補 idle 至最長候選。π／π2 長度不同，故 π/2 和 identity 後面各有約 0.625496 µs padding。每個 gate slot 約 1.256784 µs。總 slot 時間相同，但 slot 中的 drive、idle 與 qubit state 不同。原 AllXY fit 只使用 gate 名稱的低階多項式，未讀取這些時序，也未包含 T1／T2 與模擬器的 detune ensemble。

兩份 baseline raw 的原 fit residual RMS 為 3.5604／3.9081，repeat 差異估出的單次 noise 約 0.7241。使用固定的模擬物理參數、原 pulse cfg 與正確 padding，僅擬合 affine readout offset／scale，residual 降至 0.6981／0.5654。另外兩個 gain 的既有 raw 也降至相近 noise 尺度。不含 padding 的耗散模型仍有約 1.74 至 2.04 的 residual，說明不能只補一個 T1 參數而忽略 sequence。

![物理模型與 residual 對照](physical-model-comparison.png)

圖由原 task 的離線 model_probes.py 產生，數值副本見 [model-probes.json](model-probes.json)。full_padded 使用固定 T1=20 µs、T2=15 µs、T2*=8 µs、41 點 Lorentzian detuning quadrature 及原 pulse／padding。每個 pair 從 thermal equilibrium 起始，省略逐 rep 的 readout backaction、relax carry 和 hardware timestamp rounding。此近似足以解釋本案例，不是可無條件移轉的 production fitter。

九組不同 optimizer 初值返回相同 minimum。放寬 contrast bound 幾乎不改變結果。這些對照排除了本案例的初值與 contrast-bound 假說，不表示所有 fit failure 都可排除它們。

另以精確無耗散旋轉注入 +1% gain error，現行 power_param 返回 0.015694，接近 π/2×0.01，而非 0.01。它的小誤差尺度是 π/2 pulse 的角度誤差，與文件所稱相對振幅不符。power_err 則是平均 Bloch z 偏差，同模型下的平均 population deviation 須除以二；均不是 gate infidelity。這是當次實作的語意問題，修版後重新核對，不將換算常數硬套到其他工具。

Fixed-slot 或 back-to-back 應由實驗目標決定。此排查沒有證明 I 應一律零時間，也沒有核准刪除 padding。修正方向仍需先確認 Run 的時序定義，再使分析一致。沒有修改核心程式或寫回校準。

## 工具限制

新增 experiment 後，MCP reconnect 只刷新 catalog，未讓舊 GUI 載入新 experiment。當次沒有公開 reload RPC，取得使用者授權後重啟才出現新 adapter。重啟初始化把 fake_flux 設成 0.5，量測前已恢復並核對 0。恢復 project／context 不等於恢復裝置值。

Zigzag gain scan 的 31 gain 點 × 7 repetition 配置可 Run、可分析，但 raw Save 失敗，錯誤為 `axis 'Times' length 7 != z dim 31`。圖像已保存，raw 留在 GUI tab；沒有 reset、重用或關閉該 tab。這是當次版本的操作證據，不推定為所有版本的限制。Scan 候選僅作定位，後續一維 Run 有保存 raw，才納入本案例比較。
