# 30點integer到half的模擬coherence案例

2026-10-05，Qubit-measure-gui的mock SoC與FakeDevice。任務以4小時為上限，從14:10開始，最後補量測在15:28完成。所有硬體操作都經measure-gui MCP；沒有使用predictor或讀取模擬器實作作為量測答案。

## 結果與可用範圍

30個native flux控制值均勻分布於0.002499873573093248至0，包含兩端。步長−0.0000862025370032154。Integer與half沿用前輪已選分支及局部extremum證據，不代表重新校準絕對磁通編號。

每點重新核對頻率及Rabi，再做兩個T1窗口、兩種Ramsey fringe、兩種人工detune echo的互補phase pairs。最終選用328份raw，逐份核對flux、pulse、讀出、軸與保存狀態。沒有插值補coherence。

| 指標 | 30點平均值的最小值，µs | 最大值，µs |
| --- | ---: | ---: |
| T1 | 19.9475 | 20.0612 |
| T2r | 7.9751 | 8.0586 |
| T2e | 14.4424 | 14.5914 |

這些範圍只描述模擬資料。不能據此推論真實fluxonium的coherence與flux無關，也沒有驗證gate fidelity。

[結果圖](coherence-vs-flux.png)同時畫出條件式fit標準誤及repeat／窗口診斷範圍。後者不是信賴區間。[CSV](coherence-vs-flux.csv)保留分開的欄位，[稽核摘要](audit-summary.json)保存總數與極值。

## 人工detune echo的對照

使用者建議讓echo上下振盪以改善offset與envelope的辨識。p03位於native0.0022412659620836013，驅動約5407.1543MHz。總free-evolution請求100µs、401點，人工detune ratio為0.05與0.08，對應設定detune約0.20與0.32MHz。分析一律使用raw的實際delay軸。

單條fringe套常數背景或獨立T1背景都留下結構性residual。[單條模型比較圖](p03-echo-detune.png)的中間panel可以看到這個問題。圖中的數值是各候選背景模型下的T2e，不能只憑曲線有振盪就接受。

固定T1背景的兩個候選為14.022與13.931µs。它們不代表已校正的結果。取得refocusing π phase0°／90°兩條資料，先做複數差分再線性投影，得到14.503±0.041與14.498±0.042µs。[差分圖](p03-echo-fringe.png)顯示殘差減少，兩種detune與改窗口結果相容。零detune差分約14.46與14.59µs，也支持同一有效衰減尺度。

這個對照否定了「Ramsey可用的T1背景模型可直接套到echo」。它沒有識別背景來源，也沒有證明phase cycling能消除所有pulse error。成對cfg與實際軸必須一致，共同背景假設仍需驗證。

## 取樣與pulse限制

使用者建議所有振盪實驗每週期至少10點。最終依raw最大時間間隔及觀察到的fringe frequency稽核，最少12.495點／週期。Rabi約106.7點／週期。對疑似aliasing資料仍需額外加密對照，不能只用已alias的fit反算。

Rabi固定phase自動fit在多個工作點失敗，自由phase在相同raw上對上約四個週期。這支持本次pulse候選，不能取代獨立pulse error量測。

使用者建議zigzag作交叉檢查。當次公開adapter清單沒有zigzag，未執行，也未繞過MCP拼接sequence。此建議記為專家方法，沒有標成實驗證實。

## 弱訊號點的補測

初輪靠近half的部分點有較大的fit error與窗口差。p24、p25、p27、p28保持校準不變，coherence從300 rounds提高到1200 rounds，每round為100 reps。p29另將頻率精修到581.84849MHz，重新做Rabi，再用1200 rounds補coherence。

全部初輪raw與分析仍保留。最後選用補測後的兩組估計，不依數值是否接近其他點來挑Run。p29同時改頻率、pulse和averages，不能把改善歸因於其中一項。

每點結果是兩個fit的算術平均。條件式平均標準誤為sqrt(SE₁²+SE₂²)/2。另報repeat全距及同Run窗口變化，沒有把它們拼成宣稱總準確度的單一誤差。

## 局部最佳化與轉移限制

前置fluxdep任務先在integer比較讀出和probe。固定probe gain0.3、length1.2µs、100 reps及50 rounds，換讀出後的中心contrast／off-resonant MAD比值約51.75到102.21。兩次頻率軸相同。[比較資料](integer-probe-comparison.json)保存各Run來源。

該比值使用沿中心contrast方向的線性IQ投影。中心取距5423.391831MHz小於0.15MHz的平均contrast，噪音取距中心超過4MHz的off-resonant MAD。JSON中的snr欄指這個定義，不是Gaussian分類SNR或通用門檻。相同probe下的讀出比較支持局部改善，但未量repeat，不能當成跨flux保證。

Integer有界readout搜尋的最佳已量integration length接近5µs上界，不能稱全域最佳。Half另外搜尋讀出。跨flux網格則用每點光譜和Rabi驗證可用對比，沒有將integer最佳化參數當成所有點的最優值。

大範圍地圖有少數低對比slice，不能把每片argmax都當f01。30點任務使用實測ridge作搜尋中心，再逐點窄掃；這支持這30個點，不反過來宣稱前輪每個map像素都已驗證。

## 來源與保留

原始repo為Qubit-measure-gui，任務入口：

- `.agent_state/measurement-tasks/sim-coherence-30flux-20261005/RESULTS.md`
- `.agent_state/measurement-tasks/sim-fluxdep-half-20261005/INDEX.md`

正式raw在`Database/simulate_20261005/integer_flux_coherence/2026/10/Data_1005/`。完整30點結果、328份raw的hash及pulse/readout稽核在30點任務的analysis目錄。此知識條目只保留精選圖與JSON，不複製整個量測資料庫。

[p03單條fit](p03-echo-detune.json)與[phase-pair fit](p03-echo-fringe.json)保留原始檔路徑、projection及模型設定。[複製清單](evidence-copies.json)記錄本案例檔案的來源與hash。跨主機使用前確認raw路徑仍可讀。

本案例的gain、channel、phase convention、native flux與等待時間不是實際硬體設定範本。移轉前回到[coherence流程](../../README.md)確認當次接線、單位、安全限制與授權。
