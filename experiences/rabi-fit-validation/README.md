# Rabi fit完成但曲線不符

## 何時使用
Length Rabi已顯示振盪，但自動fit幾乎平坦，或擬合頻率與目視週期不同時，先驗證fit，不接受π pulse候選。

## 已觀察案例
2026-10-03 mock half-flux量測，frequency581.839698MHz、gain.15，length0.03–10.01us，共201點，reps1、rounds40000。資料有接近兩個週期。預設decay=true、fit_phase=false返回1.039±.227MHz，曲線幾乎平坦，π length .481us不在第一個峰。

同一份raw重新分析。decay=false、fit_phase=true得到.191941MHz；decay=true、fit_phase=true得到.191799MHz、π length2.66463us，曲線對上資料。後續三次T1為20.77、20.65、20.01us。

## 方法與限制
先對照擬合曲線、量測週期和第一個峰，再接受writeback。固定相位失敗時可比較自由相位與有無衰減模型，不必直接重跑硬體。比較合理模型的校準差異，避免只選最小error的解。

這個案例支持局部fit失敗的判斷，不能證明所有qubit都應開啟phase。自由phase也不等於已校正pulse fidelity；T1一致不能替代獨立π pulse fidelity量測。沒有檢查fit實作或模擬器真值。

## 證據
Qubit-measure-gui repo的`.agent_state/measurement-tasks/half-flux-t1-20261003/`保存run16.json、run16-final-analysis.json、對應PNG與verified-results.json。raw位於`Database/mcp_half_flux_20261003/sim/2026/10/Data_1003/sim_len_rabi_1003@half_flux_t1_clean_1.hdf5`。這些是repo-local路徑，跨主機重用前核對可讀性。
