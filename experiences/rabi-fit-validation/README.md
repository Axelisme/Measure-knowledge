# Rabi fit完成但曲線不符

## 何時使用
Length Rabi 已顯示振盪，但自動 fit 幾乎平坦、擬合頻率與目視週期不同，或 π pulse 候選不在第一個峰時，先驗證 fit，再決定 writeback。Amp Rabi 也要核對週期與候選峰。以下主要證據來自 Length Rabi，另有一次尚未解決的 amplitude 交叉檢查。

## 已觀察案例
2026-10-03 mock half-flux量測，frequency581.839698MHz、gain.15，length0.03–10.01us，共201點，reps1、rounds40000。資料有接近兩個週期。預設decay=true、fit_phase=false返回1.039±.227MHz，曲線幾乎平坦，π length .481us不在第一個峰。

同一份raw重新分析。decay=false、fit_phase=true得到.191941MHz；decay=true、fit_phase=true得到.191799MHz、π length2.66463us，曲線對上資料。後續三次T1為20.77、20.65、20.01us。

## 方法與限制
先對照擬合曲線、目視週期和第一個峰，再接受 writeback。這裡的第一個峰是指對應 π 旋轉的首個 excited-state population 極值；原始 IQ 的正負方向可能使它顯示為谷。沒有確認讀出對比方向時，不能只把數值最大點叫作 π pulse。

確認時間或 gain 軸、單位、offset 與起點，候選 π／π2 的定義要對應目前模型。若模型有相位或時間 offset，不直接把 π length 除以二當成已驗證的 π/2。窗口或取樣不足以辨認振盪時，先保留未知，再依授權選擇調整窗口或密度。

固定相位失敗時，在同一 Run 比較自由相位與有無衰減模型，不必直接重跑硬體。保留舊圖、分析選項與必要證據，避免後一次 canonical 圖片覆蓋對照。比較合理模型下的候選差異、stderr、residual 與資料一致性，不只選最小 error 的解。

從 MCP estimate 的 `quality.fit` 或 GUI summary 的 `fit_quality.fit` 讀 R²、normalized residual RMS、optimizer 參數的 relative errors 與 invalid reason。`null` 不是零誤差；這些 optimizer 誤差不是衍生 π pulse error 的替代值。目前版本未提供指標時，使用圖、已有 fit error 與 repeat，不捏造 R²。門檻需附設備、模型、窗口和噪音條件，不設全域 R² accept 開關。

在同一工作點與可比較 pulse／讀出條件下 repeat，核對週期、第一個峰與候選的不確定性。Repeat 不一致時先辨別漂移或模型差異；相同的模型偏差也可能每次重現，不能把一致當成 pulse fidelity 的證明。

這個案例支持局部 fit 失敗的判斷，不能證明所有 qubit 都應開啟 phase。自由 phase 也不等於已校正 pulse fidelity；T1 一致不能替代獨立 π pulse fidelity 量測。沒有檢查 fit 實作或模擬器真值。案例中約 20 us 的 T1 只屬於該 mock 工作點，不是通用門檻；判讀 T1 的窗口與尾端時讀 [T1 fit 驗證](../t1-fit-validation/README.md)。

## 依 pulse 時間尺度選 gain

2026-10-05 使用者對 Q12_2D[10]/Q1 真實硬體建議：Rabi 校準後的 pulse length 儘量控制在 0.05–0.2 µs，該次硬體不能短於 0.03 µs；過長 pulse 會受 T2 影響。這是使用者提供的當次硬體限制與專家建議，不是所有設備的通用下限。調 gain 後重新校準，並同時檢查 π 與 π/2 都在有效長度範圍；不要只讓 π 合格而使 π/2 短於限制。Rabi sweep 的窗口仍需涵蓋足夠振盪以辨識週期。

## 選 gain 與核對量化

同日使用者進一步提供常用起始gain範圍：**-0.3至1.0**，並建議優先調整固定pulse length來控制此範圍內的週期數。約1.5週期是設計取捨，不是必須裁切到的硬性值；固定gain範圍在不同flux／pulse length不會對應固定週期數。近線性條件下週期數與固定length成正比，可用已測週期數估算下一次length，再實測確認。

使用者的操作經驗是：Rabi形狀正常時，通常較高gain、較短length對gate品質較有利；曲線變形則可能打到其他驅動／躍遷。將此作為優先尋找較短有效pulse的依據，同時遵守硬體下限與適用功率範圍。曲線變形時檢查頻率、其他激發、恢復等待與drive響應，不把高gain一律視為改善，也不把形狀正常或pulse縮短本身当作gate fidelity量測。

2026-10-05使用者補充amplitude Rabi的掃描設計：gain range可從負值開始，幫助擬合涵蓋完整週期；整體窗口約1.5個Rabi週期，以兼顧校準品質與gate length。固定pulse length、近共振且gain響應近線性時，gain週期約為2×π gain，故1.5週期的span約3×π gain。例如π gain約.30時，可試-.15至+.75（span .90）。這是專家提供的起始設計，不是每點必須固定同一範圍；核對signed gain支援與硬體幅度限制，依實測週期調整。負gain仍是signed drive，不能把軸取絕對值折疊，也不因前段為負值就使用skip移除。核對零點兩側、候選極值及actual gain軸，未驗證的範圍設計不等於擬合或gate已合格。

2026-10-05使用者補充：在本平台，length Rabi的時長掃描有約10ns量級的量化，而gain解析度更細，故X180／X90等gate優先使用amplitude Rabi；length Rabi先用於選擇合適固定時長與gain搜尋範圍。這是使用者的硬體經驗與操作偏好；實際量化仍以各channel保存的軸及公開硬體資訊核對，不把10ns當成所有channel的精確常數。固定時長後分別驗證π／π2 gain，必要時以Zigzag檢查累積誤差；gain數位解析度較細本身不等於已證明gate fidelity較高。

若同一工作點已有可信 Rabi 週期，在局部近似線性 drive 響應下，可用 `g_new ≈ g_old × t_pi_old / t_pi_target` 提出下一個 gain。這只用於設計下一輪；非線性、失諧、pulse shaping 與 leakage 都會破壞比例，必須重新量測。目標需同時容納 π、π/2 的建議時長，並符合執行時的設定限制。

分清最後使用的 pulse 長度與 Rabi 掃描窗口。前者應落在有效範圍；後者需涵蓋可解析振盪，以估計週期與 phase。過長窗口可能因 T2 使後半段只剩雜訊，過密窗口也可能量化成重複或不符請求的點。擬合用 actual axis，檢查首點、step、終點和獨立點數。

2026-10-05 使用者進一步澄清：pulse 長度會量化，實驗座標軸原則上已經使用量化後的值；所稱不能短於 0.03 µs，是不合法設定會在執行時顯式報錯的限制。因此要區分設定的合法性與已量化的實驗座標，不要再手動量化一次，也不要僅因 actual 座標略低於設定下限而刪點或宣稱硬體違規。

[真實案例](../coherence-bringup/cases/real-integer-20261005/README.md) 名義首點 0.030 µs、保存座標 0.02838 µs，Run 成功且未回報該下限錯誤。先前把它稱作不合法點、要求起點改 0.035 µs 的推論已撤回。曾做的排除首點 fit 只是一個分析敏感性對照，不是必要的有效性修復。遇到顯式 pulse 長度錯誤時，依公開契約修正設定；成功執行後以實驗提供的量化座標擬合。成功執行也不等於已驗證 pulse fidelity。

## Zigzag 與獨立 pulse 檢查

2026-10-05 使用者建議以 zigzag 實驗檢查 Rabi pulse 是否正常。這是待依實驗定義執行的專家建議，不能把一次 Rabi 擬合良好當成已通過 zigzag。先查 live adapter 的可用入口、sequence 與 phase convention；該次 measure-gui adapter.list 沒有提供 zigzag，尚未執行。若後續版本提供入口，再依當次預算與硬體授權安排。

## 恢復等待條件的對照

2026-10-05使用者補充真實硬體經驗：length Rabi若呈現多個U字形拼接，而非sin/cos形狀，常見原因是relax delay不夠長。將此形狀作為優先檢查恢復等待的線索；保持frequency、gain、pulse掃描、readout與分析相同，只延長relax delay比較形狀、週期及pulse候選。這是使用者專家建議，不是看到U形就已證明機制。

同日使用者提醒：amplitude Rabi在低gain的振幅小於高gain時，其中一個原因是drive frequency存在detune。優先重核共振頻率，再在相同pulse時長及讀出條件下比较；不要先把gain依賴振幅全部歸因於讀出或功率非線性。此現象也不唯一識別detuning，須用頻率對照驗證。

已有足夠振盪而 residual 仍有結構時，不要只增加 fit 自由度。用暫定 pulse 取得 T1 初估後，檢查 repetition／recovery wait 是否影響起始狀態。確認等待時間在 sequence 中的位置，以及實際 repetition interval 是否包含 pulse、讀出與額外 delay。T1 只提供一個時間尺度，不能單靠固定倍數保證 reset；熱激發、leakage 或其他慢過程仍需另外辨別。

在同工作點固定 pulse、讀出、實際軸及分析模型，比較不同等待時間。若週期、contrast、候選或 residual 變動，就以驗證後的等待條件重新校準，再進入 coherence。真實硬體還需考慮 duty cycle、加熱與總量測時間。改善不唯一證明「未充分 reset」，沒有改善也不能排除所有起始狀態問題。

[2026-10-05 simulate 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 對 relax 30.5／150 µs 的已存 raw 使用相同自由相位模型，R² 為 0.988703／0.999755，residual RMS 為 7.9899／1.4515。這支持等待時間影響 Rabi，並排除只是 GUI phase 選項不同造成改善。每個 wait 只有一個 Run，沒有測 gate fidelity。

該案例先前的 Amp Rabi 候選 π gain 約 0.3256，與使用的 0.3 不一致且有 residual，沒有接受它。Length／amplitude 交叉檢查應在相同 pulse shape、頻率、讀出與起始狀態下比較旋轉角，不強迫兩個有偏模型給出相同答案。找出不一致值得補驗證，但不等於已完成第二種校準。

## 交叉檢查pulse，而不只檢查Rabi fit

使用者於2026-10-05建議用zigzag檢查Rabi產生的pulse。後續已完成 [Zigzag／AllXY mock 對照](cases/sim-zigzag-allxy-20261005/README.md)，觀察到 Rabi 候選仍有 sequence-dependent 偏離。先讀當前公開experiment guide，確認sequence要驗證π、π/2或哪一種誤差，再選代表工作點對照。沒有可用入口時保留缺口，不把普通Rabi或coherence repeat重新命名為zigzag。

[30點案例](../coherence-bringup/cases/sim-30flux-20261005/README.md) 的live adapter清單沒有zigzag，因而沒有執行。每點Rabi都有可辨認的約四個週期，且pulse候選與曲線相符；這仍不構成gate fidelity量測。

Zigzag 的平坦度與 AllXY 的模型偏差應共同檢查。改 gain 時記錄是只改 repeated pulse，還是連 X90 preparation 一起改。兩種實驗選到不同候選時，先保留不一致，不挑一個較漂亮的指標當作校準完成。AllXY 的 power_err／detune_err 在本次 guide 是模型的 mean state deviation，不是 gate infidelity，也不是直接可套用的 gain／frequency 修正量。Fit g/e levels 的選項會影響數值，須保留分析敏感性。

振盪取樣也要核對。使用者建議連續時間 fringe 每週期至少10點，算法及aliasing限制見[背景辨別的取樣說明](../coherence-background-validation/README.md#振盪取樣)。不要只增加averages來補救時間網格過疏。Zigzag 的整數 repetition 與 AllXY 的 gate-pair index 是離散 sequence，不能套用相同的時間取樣判準。

## Sequence 時序與分析模型要一致

Gate 名稱不完整描述實驗。核對 pulse duration、identity 的等待時間、gate slot、padding、pre/post delay 及 pulse 間隔。I 表示不旋轉，不保證零時間。Fixed-slot 與 back-to-back 在有耗散或 detuning 時不同，不能預設其中一種一定正確。先確認要執行的 sequence，再決定分析模型；不能把非預期等待一律交給更多 fit 參數吸收。

時間資訊應來自該筆 Run 的保存條件，不由目前 library 反推。Length-calibrated π／π2 改成等長、不同 gain，需要新的 amplitude calibration，不能當作無影響的實作調整。

## 用 residual 判斷是否缺少物理

固定 pulse／讀出條件做 repeat，將 trace 差異與 fit residual 比較。Residual 有可重現結構、且大於 repeat-noise 時，先檢查模型與時序，不只增加 averages。兩次獨立、相近雜訊的 repeat 可用差值 RMS 除以 sqrt(2) 估單次 noise；漂移、相關噪音與樣本不足會影響這個估計，不設跨器件的固定倍數門檻。

T1／T2 在 pulse 和 idle 中都會作用。只看理想階梯，或只用振幅／detuning 參數擬合，可能把耗散歸因為 gate error。使用獨立 coherence 校準作固定條件，再檢查其不確定性；不預設單條 21 點 AllXY 能同時辨識 coherence、兩個 pulse error、detuning 和讀出尺度。Echo 的有效 T2 也不在所有噪音環境下等於 homogeneous T2。

資料 min/max 可作讀出尺度初值，不能當作獨立 g/e 校準。重新正規化後 error 變小，需同時核對 residual 與參數穩定性。參數名稱也不足以決定單位；確認回報的是 gain 百分比、角度、population、Bloch z，還是 fidelity。不能將模型中的 mean deviation 直接作 gain correction。

[本次 mock 案例](cases/sim-zigzag-allxy-20261005/README.md) 保留後續 DEVELOPMENT 排查與量測的區別。實作／模擬真值可以用於已授權的工具診斷，不能反過來補成實驗已驗證的校準，也不擴張 MEASUREMENT 的權限。

## 證據
Qubit-measure-gui repo的`.agent_state/measurement-tasks/half-flux-t1-20261003/`保存run16.json、run16-final-analysis.json、對應PNG與verified-results.json。raw位於`Database/mcp_half_flux_20261003/sim/2026/10/Data_1003/sim_len_rabi_1003@half_flux_t1_clean_1.hdf5`。這些是repo-local路徑，跨主機重用前核對可讀性。
