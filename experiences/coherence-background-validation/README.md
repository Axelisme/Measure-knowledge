# Ramsey 與 echo 的非定常背景辨別

## 何時使用

Coherence 曲線尾端有趨勢、residual 有結構，或同一工作點改 phase 後得到不相容的 T2 時使用。前提是 raw 保留複數 IQ 或已確認的線性觀測量，且 pulse、時間軸與讀出設定可追溯。

[模擬案例](../coherence-bringup/cases/sim-integer-20261005/README.md) 支持兩種辨別方法，沒有證明所有背景都遵循 T1，也沒有識別背景的微觀來源。

## 先保留線性訊號

令原始訊號為複數 `z(t)=I(t)+iQ(t)`。固定線性 projection 可寫成 `y(t)=Re[z(t) exp(-iθ)]`。記錄 θ 的來源，對比較中的資料使用相同的 projection 規則。檢查正交分量，避免只看一個看起來最漂亮的方向。

不要先對各曲線取 magnitude 再做背景相減。Magnitude 是非線性轉換，可能抹掉正負 contrast 並引入噪音偏差。不同 Run 各自旋轉、正規化或取絕對值後，也可能不再具有共同背景。

先核對實際 delay 的物理定義、量化後軸與 pulse 間隔。Echo 的總 free evolution 和單側 delay 若差一倍，不能由 fit 自行修正。

## Ramsey 的模型比較

先用同一份 raw、同一 projection 比較常數 baseline 模型與有理由的候選模型。不要同時改 projection、窗口和模型後，把改善全部歸因於新 baseline。

案例比較的是：

```text
Constant baseline:
y(t) = C + A exp(-t/T2r) cos(2π f t + φ)

Candidate baseline:
y(t) = C + B exp(-t/T1) + A exp(-t/T2r) cos(2π f t + φ)
```

C 是常數背景，B 是隨 delay 衰減的背景幅度，A 是 fringe 幅度，f 是觀測到的 fringe frequency，φ 是相位。第二式的 T1 來自同工作點的獨立量測。T2r 描述這個模型下的 envelope；它不自動等同某一種噪音機制。

採用前要看 residual 趨勢是否減少、參數是否可辨認，以及改窗口、改合理的固定 T1、改人工 fringe 後的穩定性。若 baseline 的時間尺度和 envelope 無法區分，增加自由參數可能只掩蓋不可辨認性。應補 reference 或其他能區分兩者的量測，而不是強制固定成預期 T1。

獨立 T1 也有誤差。固定它後得到的 T2r stderr 是條件式誤差，需另報 baseline 假設與敏感性。模型比較的數值見案例，不把此式設成所有 Ramsey 的預設模型。

## Echo 的互補相位差分

若兩個 sequence 的 coherence contrast 反號，而背景不變，可寫成：

```text
z+(t) = b(t) + s(t)
z-(t) = b(t) - s(t)
common(t) = [z+(t) + z-(t)] / 2
contrast(t) = [z+(t) - z-(t)] / 2
```

b 是共同背景，s 是欲估計的 coherence 訊號。先在複數 IQ 做和、差，再用一個共同的線性 projection 看背景與 contrast。省略差分的 1/2 只改幅度，但必須記錄慣例。

案例固定兩個 π/2 pulse 的 phase 為 0°，把中間 refocusing π pulse 在 0°／90° 間切換，觀察到反號 contrast。這是該 sequence 的互補相位，不是每種 echo 都可直接套用的設定。

真實硬體使用前，需驗證 phase convention、pulse 校準與反號關係。兩條資料的工作點、實際 delay、讀出和非相位設定需相同。Pulse error、leakage、phase-dependent background 或取得兩條資料期間的漂移，都會破壞共同背景假設。軸不一致時先查原因，不為了能相減就任意插值。

可在授權內交錯取得 phase pairs，或反轉取得順序再做一組。檢查 common 曲線、差分 residual 及窗口敏感性。若 contrast 不反號、common 隨 phase 改變或 repeat 不一致，就不能把差分 fit 當作已校正的 T2。

## 人工detune與互補相位可以一起用

使用者於2026-10-05建議，echo加入人工detune後，訊號繞背景上下振盪，通常比單側衰減容易區分offset與envelope。這是實驗設計建議，不保證背景恆定或fit沒有偏差。

先用兩種可解析的detune，在相同工作點比較窗口與結果。若基線仍隨delay變化，保留人工fringe並加上已驗證的互補phase pair。先做複數IQ差分，再fit衰減fringe。確認兩次Run除了phase之外的cfg和實際軸相同。

[30點模擬案例](../coherence-bringup/cases/sim-30flux-20261005/README.md) 的p03提供一個反例。單條人工detune echo套用固定T1背景後，仍有系統性residual，兩個T2e估計為14.02與13.93µs。互補phase差分後得到14.503與14.498µs，與零detune差分相容。Ramsey適用的背景模型不能直接當成echo的預設模型。圖和條件見案例。

本次證據支持這組sequence的差分方法，沒有識別背景微觀來源，也沒有證明所有phase error都會被抵消。人工detune和phase cycling各自需要對照，不以更漂亮的對稱曲線作為證明。

## 振盪取樣

使用者建議Rabi、Ramsey及echo每個振盪週期至少10個採樣點。規劃時以最快預期fringe及最大時間間隔檢查，量完再用保存的實際軸和觀察到的frequency核對：

```text
samples_per_period_min = 1 / (fringe_frequency * max_actual_time_step)
```

Frequency和time使用互為倒數的單位。這是避免欠取樣的規劃下限，不是足夠訊號、足夠週期或可辨認envelope的保證。延長窗口卻不加點數時，需要同時降低detune或重新評估取樣。若可能有aliasing，不能只相信已alias的fit frequency算出的採樣數。

30點案例的最終Rabi、Ramsey與echo資料均至少12.495點／週期。這支持該次取樣設定，不能把案例的detune ratio當成所有sequence的通用值。

## 接受與停止

單條曲線的 fit 很好，仍可能把 baseline 當成 coherence。反過來，差分後更接近預期值也不是證明。需要模型假設、對照與敏感性共同支持。

案例增加 recovery wait 沒有解決 echo 的偏差，互補相位才分離出穩定的有效衰減。不要看到任何 residual 就延長等待。先用最小的對照區分假說。

若預算或 phase 控制不允許驗證，保留單條 fit 為模型條件下的估計，明列偏差風險，不覆蓋成「已消除背景」的校準值。T2 echo 比 Ramsey 大或小，都不能單獨決定接受或拒絕。
