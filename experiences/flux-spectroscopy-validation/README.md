# 從 flux map 到可驗證的局部工作點

## 何時使用

Resonator flux map 有週期或對稱 extrema，但 integer／half-flux 標記不確定，或 two-tone 自動 fit 選到雜訊與旁峰時使用。適用於能在授權範圍內追蹤共振與 qubit transition 的系統。

## 先分清三個問題

Flux 控制器讀值是裝置座標，不一定是 Φ0。週期、局部 sweet spot 與絕對 flux quantum 編號是不同資訊。局部掃描找到極值，不等於重新校準了整個 flux period。

Resonator map 可提供對稱點和分支候選。哪一個候選是目標 integer branch，需要器件模型、已知分支或 qubit spectroscopy 的證據。不能通用地把較高 resonator frequency、qubit frequency 極大值或地圖中央叫作 integer。

## 搜尋與核對

1. 先在核准頻段作 one-tone 粗搜尋，再窄掃解析共振。粗掃只負責找候選，不接受欠取樣的 linewidth。
2. 在已確認的 flux 範圍和 ramp／settling 條件下作 map。記錄掃描方向與穩定時間；真實硬體若有 hysteresis，單向圖不夠。這次模擬案例沒有驗證 hysteresis。
   規劃初次辨認分支的 onetone fluxdep 時，盡量讓範圍同時包含 integer 與 half-flux 各自兩側可辨認的對稱結構；只包含兩個候選中心，或讓其中一個靠近掃描邊界，不算足夠覆蓋。依既有 period/位置作種子，留出對稱結構及漂移所需的邊界，再用實測核對。若既有地圖只缺其中一側，優先比較補掃缺口與整段重掃的資訊及 ramp 成本；多張圖合用時核對 frequency grid、gain、讀出與掃描方向。硬體範圍或預算不足時明列缺口，不越界。
3. 按分支依據標記候選，移到候選後重新核對讀出。Flux 改變時，舊 readout frequency 或對比可能失效。
4. Two-tone 先辨認可追蹤的峰，再縮窄窗口。自動 fit 返回數字但資料沒有支持的峰，就不寫回。先比較 SNR、背景與取樣，再在功率限制內調整。
5. 有旁峰或多條線時，檢查不同 drive／窗口下的候選位置。中央局部 fit 可以估計中心，但不能代表完整譜形；局部 linewidth 也不能直接換算 T2。
6. 在候選兩側量 qubit frequency，保持可比較的 pulse、讀出與頻率估計方法。局部曲線插值只用於下一次更窄的驗證掃描，不向遠處外插。
7. 在最終工作點重新核對頻率及 Rabi。分別保存局部位置、分支判定依據與 period 的來源。

若兩側變化小於頻率估計的不確定性，只能說目前解析度內看不出斜率。若 peak 跳支、多峰未分離或前後漂移，就先保留工作點區間，不能報出插值器的很多位數作精度。

## 對稱點與局部極值不一致時

保留兩個估計為不同觀測量。Resonator 鏡像中心可能受分支混合、背景、窗口、取樣與量測先後影響；qubit 局部極值也依賴譜線追蹤和擬合區間。差異不能唯一診斷磁滯、串擾或器件偏移。

在分支已有依據時，選能解析曲率的幾個電流點，固定 drive／readout 追蹤同一條線。用局部二次曲線提出下一個工作點，再移到候選驗證頻率。三點可決定二次曲線，但沒有多餘自由度檢查模型失配；即使 propagated vertex stderr 很小，也不是位置總精度。預算允許時補候選兩側點、反向掃描或回到同一點檢查漂移。

Two-tone 以較大 gain 找到候選後，可降低 gain 並加密頻率以減少功率展寬，同時檢查峰中心穩定性與旁峰。線寬變小支持搜尋條件的影響，不足以換算 intrinsic T2。

## Fluxonium 大頻寬 map 的採樣與 drive 分段

2026-10-05 使用者補充的專家判斷：fluxonium integer 的 f01 常約4–6GHz，half 常低於1GHz；01 charge matrix element 在integer較大，在half較小，plasma transition轉入fluxon transition時可能陡降。這些是規劃線索，不是每顆器件的固定數值或完整能階辨識。

- 先在integer校準readout，比較spectroscopy probe gain/length的對比和展寬。此處適用的drive不一定能看見fluxon段；清楚的plasma支線也不等於全程f01。
- 用少量flux點作較寬頻率搜尋，保留同一flux下的多個候選；再依已測分支收窄頻帶並加密flux。網格成本約為flux點數×frequency點數×averages×sequence時間，另有ramp、編譯和傳輸成本。以實測耗時更新估算。
- 頻率步距要與搜尋條件下的linewidth和SNR一起衡量。粗掃用於發現，不能把一兩個點構成的峰當成精準linewidth；大空窗不代表躍遷不存在。
- Plasma→fluxon附近失去對比時，除readout失效與未覆蓋窗口，也考慮charge matrix element下降。在已確認的硬體限制內提高gain或延長spectroscopy probe，必要時增加averages；找到後用較低gain或局部細掃核對中心、展寬和分支。不能不斷增加averages來代替錯誤drive路徑或頻帶的修正。
- 跨drive channel時核對實體接線、NQZ、可用DDS頻帶及mixer。低頻mixer可取掃描中間，但以當次SoC與路徑限制為準。不同channel的相同數字gain不代表同樣物理drive。
- 用flux連續性、功率依賴和half兩側minimum共同辨別候選。多光子線、高階transition或強plasma線可能比f01亮；不能直接取每欄最強點串起來。

當次Q12_2D[10]/Q1測量的ch2適用>1GHz、ch14適用<1GHz及±10mA是使用者對該硬體的授權條件，不是本知識的通用硬體設定。使用者其後另指出NQZ1適合<2GHz、NQZ2適合>2GHz；所以同一ch2在1–2GHz與>2GHz仍需分段切NQZ，不能把「同一channel」等同「同一NQZ」。Agent最初將integer的ch2/NQZ2一路沿用到1–2GHz，留下不適用的弱訊號／雜訊條件。修正後也必須重新觀測，不可宣稱一定修復所有弱訊號。此反例提醒先核對channel、NQZ、mixer三個不同條件，再歸因於matrix element。

整合多段map時記錄各段channel/NQZ/mixer/probe/readout/averages，不讓分段色階造成對比可直接互比的錯覺。

[真實案例](../coherence-bringup/cases/real-integer-20261005/README.md) 的 resonator 鏡像候選約 −0.244 mA，局部 qubit 極值候選約 −0.530 mA；採後者量 coherence，但未驗證磁滯或絕對 flux 編號。此案例支持 map 辨認分支、局部 spectroscopy 精修的分工，數值差異不是固定修正量。

## 模擬證據

[2026-10-05 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 先從 resonator map 找到 native 約 0.002515 的候選，再用局部 qubit spectroscopy 收斂到約 0.002500。中心的 qubit frequency 高於兩側，支持該已選分支上的局部極大值。這不證明絕對 integer 編號。

案例的弱訊號搜尋同時提高 readout gain、加密網格並增加 averages，之後找到譜線。這只支持整組搜尋條件有效，不能把改善歸因於 gain 一項。真實裝置要先確認讀出線性區、功率與加熱限制，不能照抄 gain。

沒有足夠分支背景或安全範圍時，帶著 map、候選與未確定事項詢問使用者。不要為了替 extrema 命名而擴大 flux 範圍。

## 實驗者修正來源

2026-10-05，Q12_2D[10]/Q1 真實硬體任務中，使用者補充：「你應該儘量讓onetone fluxdep flux範圍包含integer與half的對稱結構」。這是實驗規劃的專家建議，不是某個固定電流範圍已適用所有器件的實驗證明。該任務先前 ±1 mA 只看到平坦局部；擴至 ±8 mA 後中央分支較清楚，但約 +7.2 mA 的 half 候選仍靠近邊界。當次資料與續測狀態記於 `.agent_state/measurement-tasks/20261005-q1-integer-coherence/`；實際電流界限只適用當次授權，不作通用設定。
