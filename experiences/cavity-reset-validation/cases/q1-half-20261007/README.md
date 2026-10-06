# Q1 half flux：初始化 coherence 與 drive 歷史分開辨別

2026-10-07，Q12_2D[10]/Q1，flux +7.1804mA；qubit309.15320594MHz、ch14/NQZ1/mixer317.5，readout5349.92185792MHz、gain.02。MIST5345.80198085MHz/gain.11/20.01µs，tone後約.350µs才做250ns resetπ。這些是案例條件，不能直接移植。

## 新證據改變了什麼

前一日的「殘留 photon／快速 Stark」仍是合理待驗證假說。本次確認 ringdown 已進入編譯，且主要累積異常在 cavity tone及resetπ都設gain0時仍存在；MIST專屬殘留photon不是觀察到duration dependence的必要條件。沒有量到qubit線的RF waveform，仍不唯一識別控制鏈或qubit微觀機制。

相同MIST與gate，只改relax20→500µs，短Rabi f由1.97423±.00137→1.99803±.00129MHz，回到20重驗1.97296±.00123。額外480µs移到tone之前亦1.99759±.00150；tone→π→gate未改。GE初始G約95.59%與95.63%，不支持把差異只歸因於初態population。

將X90+nX180換成同總時長continuous const pulse，主要trace重合，限制Repeat接縫解釋。高gain .94686長Rabi在0–1.9／6–8µs局部頻率约1.9774／1.9149MHz；gain .5約1.0824／1.0767。無MIST但保留timing亦有下降。這是有效旋轉響應，不是RF振幅直接測量；內層length scan同時改變近期duty/history。

最後降低重複率至relax20ms（400reps×1、201點），早/晚仍1.99556±.00529／1.94539±.00761MHz，差−50.17±9.26kHz；單靠延長shot等待未消除duration dependence。此為一筆低shots控制，誤差是局部fit formalσ，不含模型／drift，也不能據此指定RF時常數。長等待條件間的微小差異不必單調。

![Shot等待至20ms](longwait-comparison.png)

## 初態相位 witness

30k shots、no-init、π2 probe四phase；以固定GE向量投影IQ。MIST20的0−180／90−270差為−.02447±.00539／+.05270±.00540。Passive500接近零；resetπphase180時變成+.04212±.00537／−.04136±.00583；π後等待20µs變成+.00151±.00526／−.00804±.00570。誤差為100shot block SEM，未計跨run漂移。

![初態相位與zigzag累積分量](coherence-controls.png)

左圖為扣除各組均值的probe-on IQ，支持reset相關相干分量；包含gate imperfections，不是完整tomography。右圖parity-demodulation顯示phase反轉主要改offset，n4..12 slope約−.01966／−.01915仍相近；post20 slope仍−.01422。相干分量不能單獨解釋累積異常。不同adapter的採集次序／history不同，不用GE witness數值直接預測zigzag n=0。

## Drive-on 與 conditioning 控制

±1MHz共7個RF的長Rabi，早／晚window fit最低Ω=1.97530±.00108／1.91555±.00105MHz；effective center309.10113±.00348／309.10640±.00358MHz。中心差5.27±4.99kHz，主要變化在Ω。曲率與sqrt模型相符，但絕對vertex可能受RF transfer slope偏置。將zigzag seed/repeat改到309.10613MHz並未改善主要parity，未writeback頻率。

![Drive-on局部chevron](drive-on-chevron.png)

图源drive_on_chevron.png，windows來自重建actual pulse axes及無phase邊界fit；formal errors不含模型／時段系統誤差。限制簡單detune主導解釋，不排除所有快速或drive-induced物理。

Cavity後、Rabi前加2µs/+50MHz qubit drive，可將Rabi由約1.980→1.919MHz改為約1.891→1.898MHz，變慢但較平坦；gain0的同timing控制回復。−50MHz較弱，sameRF但mixer300結果保留。同總cycle的pre/post2或20µs等待顯示postwait恢復前段速率較多。**這些conditioning組均無最後ground resetπ，以cavity準備E-biased初態**；不和ground組直接當成同prep比較。

![Conditioning與等待位置](conditioning-controls.png)

頻率不對稱不支持未加限制的「broadband amplifier compression」斷言；需要直接RF／線路證據才能指認元件。Readout gain .01/.02/.04的長Rabi速率相近，16倍readout功率差未移除duration dependence。

後續dummy offsets −75/−50/−25/+25/+50/+75MHz的全窗口effective Rabi f約1.93708/1.93652/1.93334/1.92066/1.90092/1.89480MHz；高頻側影響較強，並非只有單一+50MHz點。Same physical RF、mixer300/317.5/400MHz的active−zero差為−38.64±.83/−39.53±.84/−39.66±.83kHz。400MHz連dummy IF符號也翻轉，返回317.5仍再現，限制單純IF符號／mixer配置錯誤解釋。Formal誤差不含模型／跨時段系統誤差。

![Conditioning頻率與IF符號控制](conditioning-frequency-map.png)

## 目標benchmark與限制

80共同seeds×2rounds、200reps、depth0:4:160、root2026100701，原gate/readout、同MIST：relax20 F98.4747%，500 F98.8764%。共同seed bootstrap3000次得ΔF=+.4018pp、95%CI[+.2351,+.5756]pp；逐round及depth窗口保留方向。改善不只是zigzag代理，但CI不包含序列模型偏差與時段drift。

同日passive500 F98.9462%，相對MIST500差+.0698pp、CI[−.0754,+.2135]，未解析到長等待下MIST的额外penalty。原MIST20重驗98.4855%，相對初測+.0109pp、CI[−.1141,+.1420]，不支持以簡單整體漂移解釋長等待改善。

Reset前／resetπ後各加20µs且relax仍20，F分別98.6679／98.6804%。等總cycle比較post−pre為+.0125pp、CI[−.1109,+.1419]，沒有解析到等待位置的額外IRB收益。Post20雖使phase witness接近零，但不能把IRB改善全歸因於消除coherence；shot cycle/history也是重要混淆。不同實驗／sequence的coherence敏感性不可互相等同。

![最終配對IRB](final-irb-summary.png)

## 原始來源與分析修正

量測repo：`C:/Users/QEL/Desktop/MeasureScriptX/QuantumMeasurementProcedures/Members/Codex-agent/Qubit-measure`。
任務入口：`.agent_state/measurement-tasks/20261007-q1-mist-debug/REPORT.md`及`INDEX.md`；每個tag JSON連到正式HDF5、cfg及操作完成證據。

- `compile_timing.py`與`*_asm.txt`：離線編譯時序，非實測RF波形。
- `continuous_comparison.json`、`rabi_windows.json`、`actual_rabi_axes.json`：連續pulse／duration比較。
- `phase_cycle.json`、`tomography_*`、`zigzag_resetphase180`、`zigzag_post20`：相位及等待控制。
- `drive_on_chevron.json`與`rabi_long_chevron_*`：drive-on中心／Ω。
- `rabi_long_dummy*`與`conditioning_controls.png`：離共振conditioning；各cfg保存了prep及等待差異。
- `irb_final_conditions.json`、`irb_condition_analysis.py`、六份`irb_*80`：目標benchmark與同seed統計，native分析另存。

Raw位於`Database/Q12_2D[10]/Q1/2026/10/Data_1006/`及重啟GUI後的`Data_1007/`；1006是當時GUI保存位置，不搬動改日期。Cos phase optimizer邊界、下降座標auto-init及time-axis量化缺陷已修正並整合回原workspace，249項相關測試通過；native forward/reverse Rabi保存軸與compiler差<1e−15µs且fit成功。Raw保持不變，重建軸有provenance。這些缺陷影響分析／保存座標，不能解釋整數n raw zigzag形狀。詳見task `development_validation.md`及知識庫Rabi／calibration-provenance條目。
