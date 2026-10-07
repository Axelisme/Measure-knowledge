# Cavity reset：初態、分類與 gate 條件驗證

## 何時使用

以 cavity tone 建立遠離熱平衡的 g/e steady state，準備用 reset 取代長 relax_delay，或 reset 後的 pulse 校準與 passive 條件不同時使用。操作契約仍讀 live adapter guide；本條目不提供其他器件可直接沿用的功率或時序。

## 可辨別的工作路線

1. 先核對完整 singleshot cfg，尤其 reset、init_pulse、probe 與 readout。可選 init 即使未主動設定也可能預設為 π pulse。PreparedState 是 acquisition 的 off/on 標記，不能單憑名字認定為物理純 G/E。
2. 在明確的 no-init passive reference 下核對 IQ centres 的物理標籤，再校準分類。初態有熱混合時，raw assignment、Gaussian 分離、推估初始分布及 confusion matrix 是不同量。
3. 以 MIST power 找有用的 steady-state 區域，同時看 Other 及不同 prep 的收斂。先前錯標的 centres 必須更正；兩個 prep 曲線一致可以支持收斂，但不能把分類校正後的 clipping=1 當作完美 reset。
4. 在同一 tone freq/gain 下掃 duration，選達到目標純度的短候選；以 fresh cavity linewidth 決定 photon ringdown。若使用者指定 5*2pi/kappa，且回傳 FWHM=κ/2π（MHz），相應 delay 是 5/FWHM（µs）。先核對 linewidth 定義，不直接把 angular κ 和 MHz 混用。
5. 核對 reset module 的完整公開型別與時序。TwoPulseReset / dual-tone 名稱不保證先 tone 再 π；公開 guide 可能是兩個 tone 同時。需要順序時明確設定時序，並用物理 IQ / singleshot check 驗證初態。
6. 用獨立 singleshot/reset_check 掃不同初態，核對 reset-only 分布、Other 和 reset+probe。分類人口不是完整 reset-channel fidelity；Other 也不是已校準 leakage。
7. 改用短 relax_delay 後，重新驗證目標實驗。較純初態有機會降低 averages，但分類良好不保證 gate rotation 或 RB decay 與 passive 条件相同。保留有／無 reset 的可比控制、reference decay 與獨立 IRB；native gain loss 只作候選。

## Pulse proxy 與初始化混淆

Zigzag 的 step-difference loss 可能被前幾點 transient 或初態 coherence 影響。普通 zigzag 的 parity offset 與隨 repetition 增長的 parity 要分開看。native min 落到邊界需擴掃；即使有內部 min，仍需獨立 ordinary zigzag 與目標 gate benchmark。不要直接把 reset 條件下的 gain 套回 passive 條件。

有／無 reset 的差異本身不能唯一判定 photon、qubit drive timing、初態 coherence、加熱、硬體或編譯原因。有限的間隔控制未改善時，保留未知；不要用未公開 implementation 推測替代實測。

## Ringdown 足夠但 zigzag 仍異常時

先分開 parity 的初始 offset 與隨 gate 數累積的變化，再選能否定候選原因的控制。2026-10-07 Q1 排查提供下列可重用順序，具體幅度不作通用門檻：

- 固定 tone→π→gate，改 shot 尾端等待；再把等量等待移到 tone 之前。若結果相同，改善不需要增加本 shot 的 photon ringdown，應查 shot 歷史／duty 依賴。
- 在完整 timing 保留的條件把 cavity gain、reset π gain 分別設零。主要異常若仍存在，MIST 專屬 excitation 不是必要條件；不能因此宣布所有 photon 或加熱機制都不存在。
- 用同總時長連續 const drive 取代 seed+nπ。兩者相同時，pulse 接縫或 Repeat 重啟不能單獨解釋異常。Length 是內層掃描時，duration 和近期 duty/history 同時改變，分窗口 Rabi 不能冒稱直接 RF 包絡量測。
- 四個相位的 π/2 probe 可作相位敏感初態 witness。比較相反 phase 的 **probe-on** arms，加入 passive、reset π phase180、reset π後等待的控制。若偏向隨 reset phase反轉並隨等待消失，支持初始化相干分量；phase-dependent gate response仍是限制，不能直接稱完整state tomography。GE adapter 的 probe-off可能省略pulse，故 off/on不是duration-matched對照。
- 對 Rabi 速率變化，drive-on chevron 比長窗口 off-drive Ramsey 更能區分有效 Ω 與 detune。各duration window分別fit sqrt(Ω²+(drive−center)²)，核對自由二次曲率與殘差。有效vertex可被 RF transfer slope偏置，不自動writeback為qubit本徵頻率。
- 相同RF、改digital mixer而效果保留，僅限制IF/mixer設定特有解釋。檢查probe及conditioning pulse各自的IF；只翻轉probe IF不能代表conditioning IF亦已翻轉。離共振pre-drive若改變後續Rabi，需保留頻率、功率、等待位置及prep差異；沒有直接RF waveform／線路證據，不指認特定放大器故障。
- Resetπ後增加等待，同時改變coherence、drive recovery和總shot週期。以相同總cycle、等待放在reset前的控制比較目標benchmark，才能檢查等待位置有沒有額外收益。Phase witness改善而IRB的post−pre差未解析出來時，保留兩種觀察，不把IRB收益全歸因於coherence消失。
- 時間量化要分開 channel waveform cycles 與 reference-clock scheduling ticks。用相同 waveform、occupied time差1tick的配對，再測loop/unrolled及整段lead對不同fabric clock的完整餘數週期。名義時間不同不等於實際波形不同；只掃少數lead也不等於覆盖完整相對對齊。在Q1，這些控制均保留主要異常，但不把CI跨零當嚴格等價。
- Frequency quantization要核對compiled RF與DDS word。附近同word點作負控制、相鄰word作真變化；把名義MHz小數改很多位卻仍同word不能算頻率敏感性實驗。各pulse的gen/ADC matching grid可能不同，不用單一全域LSB代替。
- Zero-reset初態可能接近熱混合，raw zigzag振幅較小不等於gate error較小。加入零seed的初始Z尺度及四相位循環，展示正規化前後結果；這些比值仍不是各物理原因的因果占比。
- 降低gain並拉長gate可能消除可見的Rabi duration dependence，卻因耗散或其他gate誤差使IRB變差。只把它當可否定機制的控制，候選必須用新的共同seed及交錯重驗驗收。Q1的450/650ns gate比250ns更差，是不能只追求平坦zigzag的反例。
- Conditioning若與cavity重疊，須補同timing零振幅及移到cavity後的控制，並檢查resetπ是否也要重新校準。在Q1，不重疊時仍改變Rabi，因此不需要兩channel同時出力；較平響應仍未證明比簡單gain修正有IRB收益，亦非特定RF放大器故障的證據。

相位偏向消失而主要parity仍在，表示初始化coherence不是唯一原因。這也解釋為何單一gain可改善某種zigzag loss，卻未必改善隨機gate序列。最終仍以相同seed／pulse／readout的目標benchmark驗證；共同seed bootstrap只涵蓋統計變異，不涵蓋時段drift與IRB模型偏差。

## 把 reset coherence 與共同累積項分開

因果比較先固定全部waveform佔用時間、shot週期、gate/readout與採集順序，做cavity on/zero × terminal resetπ on/zero的2×2控制。每組量相反resetπ相位與probe +π/2、−π/2、同長度zero；先平均相反reset相位，再比較probe difference與zero-probe contrast。這可以分離相位奇分量與主要累積形狀，但short-cycle zero-reset穩態仍可能依前一shot而變，不能稱完整process tomography。Q1的四組late角尺度皆約.119rad/π；relax500時zero-both與MIST都約.067，說明「reset20 vs passive500」不是reset專屬因果比較。

Photon wait若放在terminal resetπ之前，不能消除之後由不完美π造成的coherence。若相反π相位使初態偏向反號，可在保持RF/gain/length時交替terminalπ的0/180相位，使每個採集點有等量兩相位；它處理平均transverse preparation error，不保證每shot純G、population正確或後續gate無誤差。Q1普通zigzag新增reset_phase_cycle，以outer reps sweep交替，偶數reps才平衡；不加RFpulse或programmed wait。需核對只有reset phase word變化，gate words與事件時間保持，並用不同sequence長度、反序blocks及DDS phase reference控制驗證。此feature不自動推廣到其他adapter。

Resetπ的單點frequency/gain null必須在實際sequence history重新驗證。Q1單點C .0275→.00337的候選，在long-history且DDS reference鎖定時只.0582→.0448，未轉移；不同RF配合free-running DDS還可能把相位平均冒充coherence消失。不能依單點null永久writeback。四phase witness採用同一GE投影，數值不是corrected Pe或已校準Bloch長度；norm有正噪聲偏置，兩blocks只給repeatability，不提供可靠母體CI。

等待位置控制須按compiled ticks配對，不能只比較名義微秒。Q1名義3.504µs的gap曾讓π→probe多1tick；離線發現後換成實際匹配的條件才量測。對比長ringdown與等cycle前置wait時，coherence沒有after-tone專屬改善，population projection反而更差；不能把加長wait當成無代價修復。已找到rounding差與它是否主導異常是兩個問題。

當使用者問「原因占比」，分別報coherence witness抑制幅度、contrast尺度與累積角尺度，不把它們轉成相加100%的責任比例。改善某個初態witness也不代表整條zigzag已修復；保留未消除的主累積圖與未唯一定位的RF/qubit機制。

詳見 [Q1 2026-10-07 診斷案例](cases/q1-half-20261007/README.md)。

## 用獨立QICK實驗分開phase frame與RF歷史

先核對live SoC類型、firmware日期、PC/board QICK版本及generator/RFDC配置，再讀對應版本的primary source。ZCU216型號本身不能決定每個channel的時鐘、mixer或類比接線。int4的const可選DDS直接路徑，繞過PL envelope FIR；不能把flat_top的FIR workaround套到const，也不能因此排除後級RFDC interpolation。Per-pulse DDS phrst與RFDC mixer NCO reset是不同層；用actual signed IF和pulse start separation預測未補償fringe，再以硬體正控制驗證。

固定shot frame只固定週期，沒有固定RF能量、上一點pulse history或初態。Point averaging與full-sweep averaging、forward/reverse應當視為不同干預。Post-ADC active/zero RF tail若改下一shot，可建立跨shot影響；固定能量與period而只改tail recency能辨別單純平均功率。再用單次pump後的固定gate探針，避免把length sweep curvature直接當RF包絡。

Rabi窗口fit建立的模型必須用rotary echo、不同frame或不同scan order作完整曲線holdout。尾端符合而早段有結構殘差，不能宣布模型通過或將effective τ命名為放大器時間常數。RFconditioning使Rabi較平也不能代替目標zigzag驗收；Q1的pre/post/complement conditioning均未修復主要parity。

四相位analysis得到的Bloch witness magnitude下降，不一定是初態混合。Analysis π/2可能受同樣RF history影響。加同時間槽zero probe與no-analysis arm，在affine兩能級readout假設下可比較直接Z與四axis平均的`Z cos(alpha)`；這有助於發現analysis gate旋轉角改變，但GE中心drift、leakage、軸不對稱與不同arm的history仍限制推導，不當作無假設tomography。

短窗口四相位Ramsey應保留共同phase reference、reset-phase even/odd、visibility及相反block順序。沒有可重現short−long phase差只限制所測窗口的Stark解釋；不要由free-phase長Ramsey或單次中心fit宣稱photon完全排除。浮點schedule warning需回到integer ticks核對，軟體timestamp與真正DAC輸出仍是不同證據。

版本來源、獨立實驗與反例見[Q1機制排查案例](cases/q1-half-mechanism-20261007/README.md)。

## 分別驗收 reset 的收益

初態分布、採樣時間與目標估計的 precision 要分開驗證。更純的 g/e 初態及更短 relax_delay 可以改善對比和吞吐；若要主張 averages 可減少，還需比較固定目標不確定性所需的 shots／時間，或固定 shots 下的不確定性。不能只由 ground model estimate 推出所需平均數。

保持 drive 與 readout 相同，比較 passive 長等待與 reset 短等待，再用新 seed 重驗。兩項同時改變時，驗證的是整組初始化流程，不能把 gate 差異唯一歸因於 reset tone。Q1 half 案例的 reset 提高初態純度並縮短採樣，但 matched X180 的均值在獨立重驗仍較低；統計 CI 有小範圍重疊，時段、模型偏差及具體機制未排除。這是重新驗證 gate 的反例，不是 reset 普遍降低 gate 品質的定律。

對 photon／AC Stark 假說，延長 ringdown 後 zigzag 未改善，僅限制所測條件下的解釋。帶自由 phase 的長窗口 Ramsey 未見顯著頻率差，仍可能沒有解析 gate 早期的快速 transient；不可據此宣布已排除 photon effect。Readout gain 改變時，也要重新核對 IQ centres／classifier，避免把讀出尺度變動混成初始化或 rotation 的變動。

## 依據與限制

2026-10-06 使用者在 Q12_2D[10]/Q1 half-flux 任務提出 singleshot→MIST→single-tone length→reset module→reset_check 路線，指出 reset 可提升初始 g/e 純度並降低等待與平均需求，且 photon decay 採 5*2pi/kappa、5µs 太長。這些是當次專家指示，不是跨裝置通用參數。

[Q1 真實案例](cases/q1-half-20261006/README.md) 保留 no-init 標籤校正、順序 reset、驗證圖及收尾 matched IRB 反例。具體功率、freq、duration 與估計純度只適用該次條件；完整 gate 比較以 task 最終報告與保存 IRB 為準。
