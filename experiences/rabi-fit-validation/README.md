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

若同一工作點已有可信 Rabi 週期，在局部近似線性 drive 響應下，可用 `g_new ≈ g_old × t_pi_old / t_pi_target` 提出下一個 gain。這只用於設計下一輪；非線性、失諧、pulse shaping 與 leakage 都會破壞比例，必須重新量測。目標需同時容納 π、π/2 的建議時長，並符合執行時的設定限制。

分清最後使用的 pulse 長度與 Rabi 掃描窗口。前者應落在有效範圍；後者需涵蓋可解析振盪，以估計週期與 phase。過長窗口可能因 T2 使後半段只剩雜訊，過密窗口也可能量化成重複或不符請求的點。擬合用 actual axis，檢查首點、step、終點和獨立點數。

2026-10-05 使用者進一步澄清：pulse 長度會量化，實驗座標軸原則上已經使用量化後的值；所稱不能短於 0.03 µs，是不合法設定會在執行時顯式報錯的限制。因此要區分設定的合法性與已量化的實驗座標，不要再手動量化一次，也不要僅因 actual 座標略低於設定下限而刪點或宣稱硬體違規。

[真實案例](../coherence-bringup/cases/real-integer-20261005/README.md) 名義首點 0.030 µs、保存座標 0.02838 µs，Run 成功且未回報該下限錯誤。先前把它稱作不合法點、要求起點改 0.035 µs 的推論已撤回。曾做的排除首點 fit 只是一個分析敏感性對照，不是必要的有效性修復。遇到顯式 pulse 長度錯誤時，依公開契約修正設定；成功執行後以實驗提供的量化座標擬合。成功執行也不等於已驗證 pulse fidelity。

## Zigzag 與獨立 pulse 檢查

2026-10-05 使用者建議以 zigzag 實驗檢查 Rabi pulse 是否正常。這是待依實驗定義執行的專家建議，不能把一次 Rabi 擬合良好當成已通過 zigzag。先查 live adapter 的可用入口、sequence 與 phase convention；該次 measure-gui adapter.list 沒有提供 zigzag，尚未執行。若後續版本提供入口，再依當次預算與硬體授權安排。

## 恢復等待條件的對照

已有足夠振盪而 residual 仍有結構時，不要只增加 fit 自由度。用暫定 pulse 取得 T1 初估後，檢查 repetition／recovery wait 是否影響起始狀態。確認等待時間在 sequence 中的位置，以及實際 repetition interval 是否包含 pulse、讀出與額外 delay。T1 只提供一個時間尺度，不能單靠固定倍數保證 reset；熱激發、leakage 或其他慢過程仍需另外辨別。

在同工作點固定 pulse、讀出、實際軸及分析模型，比較不同等待時間。若週期、contrast、候選或 residual 變動，就以驗證後的等待條件重新校準，再進入 coherence。真實硬體還需考慮 duty cycle、加熱與總量測時間。改善不唯一證明「未充分 reset」，沒有改善也不能排除所有起始狀態問題。

[2026-10-05 simulate 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 對 relax 30.5／150 µs 的已存 raw 使用相同自由相位模型，R² 為 0.988703／0.999755，residual RMS 為 7.9899／1.4515。這支持等待時間影響 Rabi，並排除只是 GUI phase 選項不同造成改善。每個 wait 只有一個 Run，沒有測 gate fidelity。

該案例先前的 Amp Rabi 候選 π gain 約 0.3256，與使用的 0.3 不一致且有 residual，沒有接受它。Length／amplitude 交叉檢查應在相同 pulse shape、頻率、讀出與起始狀態下比較旋轉角，不強迫兩個有偏模型給出相同答案。找出不一致值得補驗證，但不等於已完成第二種校準。

## 證據
Qubit-measure-gui repo的`.agent_state/measurement-tasks/half-flux-t1-20261003/`保存run16.json、run16-final-analysis.json、對應PNG與verified-results.json。raw位於`Database/mcp_half_flux_20261003/sim/2026/10/Data_1003/sim_len_rabi_1003@half_flux_t1_clean_1.hdf5`。這些是repo-local路徑，跨主機重用前核對可讀性。
