# Readout 優化：掃描收斂與短窗口取捨

## 何時使用

準備 two-tone spectrum／flux map，或 readout gain 的最佳值落在邊界、readout length 的 SNR 已接近平台時使用。前提是目前工作點的 g/e 初始化 pulse 已驗證，且 readout 接線、功率和取樣窗口限制已確認。

## 決策流程

1. 在目標工作點掃 readout frequency，核對 g/e 對比；只由 resonator dip 得到的頻率不一定有最佳分辨力。
2. 掃 gain。若最佳值落在邊界且仍上升，只能叫「目前範圍內最佳」。在授權功率範圍內擴掃，直到解析峰值／平台與更高 gain 的行為；若已到限制，保留未收斂標記。
3. 有多個 gain 峰時，比較 SNR、穩定性與功率，不必追逐最高單點。高功率下降或多峰本身不能唯一診斷非線性、加熱或躍遷。
4. 在候選 gain 下掃 length。優先選接近平台起點、SNR 已足夠的短窗口。可預先指定保留平滑峰值 SNR 的比例作任務內取捨，例如約 95%；這是操作選擇，不是通用門檻。
5. 沒有平台而有峰後下降時，保留實測形狀，核對 pulse 持續時間、積分窗口、trigger offset 和初始化。鬆弛與雜訊是可能因素，未做辨別不可直接歸因。
6. Frequency、gain、length 相互影響；大幅改變其中之一後，至少用最終完整組合驗證目標實驗的對比。g/e SNR 良好不等於任何 flux 下的 spectroscopy 都可信。

原始 argmax、人工選擇的工作值與選擇理由分開保存。若 GUI scalar 名稱仍叫 best_ro_length，另記它是短窗口取捨值；同步核對 ModuleLibrary 的 pulse length、RO length、frequency、gain 和 trigger，而不是只改 MetaDict。

## 沒有當地 g/e pulse 校準時的 spectroscopy 診斷

Flux 改變後若 qubit line 消失，清楚的 one-tone dip 只驗證讀出共振可找到，不能證明 g/e 對比仍足夠；integer 的最佳RO也不保證適用整段flux。尚無可信當地f01／π pulse時，不把預設g/e優化程式的SNR當作已校準的判據。可先用可重現的spectral feature及相鄰flux連續性比較讀出條件，稱為spectroscopy可見度診斷。

功率、積分窗口和頻率都值得納入有限的辨別實驗，而非只增加probe或averages。2026-10-05真實Q1在6.8mA的較早ROgain.08／integration1.1us條件下缺乏可信峰，ROgain.02／integration3us的組合恢復明顯IQ峰；drive.15／10us時原生中心約530.555MHz，後續五個相鄰flux點也有連續線。這支持新組合可用，未單獨證實是gain或length造成改善，亦未驗證其他flux可直接沿用。

反例是同任務5.6mA：較低RO及較強drive後仍只有寬弱Q結構，把integration3us縮到.4us（pulse3.2→.6us）增加雜訊而未改善可辨識度。不能一律認為長window較好，也不能由短window控制就確診快速T1或讀出混合；幾組有辨別力的控制後應回到相鄰區域及批次map，避免無限單點調參。

來源：`.agent_state/measurement-tasks/20261005-q1-twotone-fluxdep/journal.md`；rawfreq27/28、rawflux13為恢復案例，rawfreq42–44為5.6mA反例。更多流程見[局部失線的決策](../flux-spectroscopy-validation/README.md#譜線在局部-flux-區域消失的決策流程)。

## 依據與限制

2026-10-05 使用者於 Q12_2D[10]/Q1 任務修正：「ro_gain的掃描範圍似乎不夠大，還沒收斂。而ro_length通常snr會趨緩，建議取兼顧長度足夠短同時snr足夠的點」。這是專家建議；SNR 必然單調或必然形成平台不在此主張內。

當次 integer −0.530 mA、5351.299879 MHz，gain .01–.15 初掃的最佳點靠上界；擴到 .5 後解析內部峰值及高功率下降，採 .144296。Length 重測 .2–4.1 µs 得寬峰約1.96 µs、SNR約2.13，選1.1 µs窗口／1.2 µs pulse以保留約95–97%峰值SNR。這些是該工作點的數值，不是其他器件的模板。

正式來源位於量測 repo `Database/Q12_2D[10]/Q1/2026/10/Data_1005/`：`Q1_ro_opt_gain_1005@100515_twotone_fluxdep_1.hdf5`與`_2.hdf5`，以及`Q1_ro_opt_length_1005@100515_twotone_fluxdep_3.hdf5`。分析圖位於 `result/Q12_2D[10]/Q1/exps/100515_twotone_fluxdep/image/`。任務 `.agent_state/measurement-tasks/20261005-q1-twotone-fluxdep/` 保存初始圖與決策。後續 spectroscopy 的驗證仍以該任務紀錄為準。
