# Rabi fit完成但曲線不符

## 何時使用
Length Rabi 已顯示振盪，但自動 fit 幾乎平坦、擬合頻率與目視週期不同，或 π pulse 候選不在第一個峰時，先驗證 fit，再決定 writeback。Amp Rabi 也要核對週期與候選峰，但下面的觀察案例只涵蓋 Length Rabi。

## 已觀察案例
2026-10-03 mock half-flux量測，frequency581.839698MHz、gain.15，length0.03–10.01us，共201點，reps1、rounds40000。資料有接近兩個週期。預設decay=true、fit_phase=false返回1.039±.227MHz，曲線幾乎平坦，π length .481us不在第一個峰。

同一份raw重新分析。decay=false、fit_phase=true得到.191941MHz；decay=true、fit_phase=true得到.191799MHz、π length2.66463us，曲線對上資料。後續三次T1為20.77、20.65、20.01us。

## 方法與限制
先對照擬合曲線、目視週期和第一個峰，再接受 writeback。確認時間或 gain 軸、單位、offset 與起點，候選 π／π2 的定義要對應目前模型。窗口或取樣不足以辨認振盪時，先保留未知，再依授權選擇調整窗口或密度。

固定相位失敗時，在同一 Run 比較自由相位與有無衰減模型，不必直接重跑硬體。保留舊圖、分析選項與必要證據，避免後一次 canonical 圖片覆蓋對照。比較合理模型下的候選差異、stderr、residual 與資料一致性，不只選最小 error 的解。

從 MCP estimate 的 `quality.fit` 或 GUI summary 的 `fit_quality.fit` 讀 R²、normalized residual RMS、optimizer 參數的 relative errors 與 invalid reason。`null` 不是零誤差；這些 optimizer 誤差不是衍生 π pulse error 的替代值。目前版本未提供指標時，使用圖、已有 fit error 與 repeat，不捏造 R²。門檻需附設備、模型、窗口和噪音條件，不設全域 R² accept 開關。

在同一工作點與可比較 pulse／讀出條件下 repeat，核對週期、第一個峰與候選的不確定性。Repeat 不一致時先辨別漂移或模型差異；相同的模型偏差也可能每次重現，不能把一致當成 pulse fidelity 的證明。

這個案例支持局部 fit 失敗的判斷，不能證明所有 qubit 都應開啟 phase。自由 phase 也不等於已校正 pulse fidelity；T1 一致不能替代獨立 π pulse fidelity 量測。沒有檢查 fit 實作或模擬器真值。案例中約 20 us 的 T1 只屬於該 mock 工作點，不是通用門檻；判讀 T1 的窗口與尾端時讀 [T1 fit 驗證](../t1-fit-validation/README.md)。

## 證據
Qubit-measure-gui repo的`.agent_state/measurement-tasks/half-flux-t1-20261003/`保存run16.json、run16-final-analysis.json、對應PNG與verified-results.json。raw位於`Database/mcp_half_flux_20261003/sim/2026/10/Data_1003/sim_len_rabi_1003@half_flux_t1_clean_1.hdf5`。這些是repo-local路徑，跨主機重用前核對可讀性。
