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

## 模擬案例能支持什麼

[2026-10-05 案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 先從 resonator map 找到 native 約 0.002515 的候選，再用局部 qubit spectroscopy 收斂到約 0.002500。中心的 qubit frequency 高於兩側，支持該已選分支上的局部極大值。這不證明絕對 integer 編號。

案例的弱訊號搜尋同時提高 readout gain、加密網格並增加 averages，之後找到譜線。這只支持整組搜尋條件有效，不能把改善歸因於 gain 一項。真實裝置要先確認讀出線性區、功率與加熱限制，不能照抄 gain。

沒有足夠分支背景或安全範圍時，帶著 map、候選與未確定事項詢問使用者。不要為了替 extrema 命名而擴大 flux 範圍。

## 實驗者修正來源

2026-10-05，Q12_2D[10]/Q1 真實硬體任務中，使用者補充：「你應該儘量讓onetone fluxdep flux範圍包含integer與half的對稱結構」。這是實驗規劃的專家建議，不是某個固定電流範圍已適用所有器件的實驗證明。該任務先前 ±1 mA 只看到平坦局部；擴至 ±8 mA 後中央分支較清楚，但約 +7.2 mA 的 half 候選仍靠近邊界。當次資料與續測狀態記於 `.agent_state/measurement-tasks/20261005-q1-integer-coherence/`；實際電流界限只適用當次授權，不作通用設定。
