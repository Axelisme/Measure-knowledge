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

### 人工 detune 與擬合

2026-10-05 使用者建議 echo 加入人工 detune，讓訊號在背景上下振盪，更容易辨識 envelope。這是專家方法建議；人工相位造成的振盪不等同物理 drive detuning。使用前依 live adapter guide 確認 detune_ratio 與實際 delay step 的關係，搭配 fringe fit，記錄總 free-evolution delay、人工 fringe frequency 及 phase 設定。上下對稱有助於辨認背景，但不能單獨证明背景恆定或消除模型偏差。

### 互補相位對照

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

## 接受與停止

單條曲線的 fit 很好，仍可能把 baseline 當成 coherence。反過來，差分後更接近預期值也不是證明。需要模型假設、對照與敏感性共同支持。

案例增加 recovery wait 沒有解決 echo 的偏差，互補相位才分離出穩定的有效衰減。不要看到任何 residual 就延長等待。先用最小的對照區分假說。

若預算或 phase 控制不允許驗證，保留單條 fit 為模型條件下的估計，明列偏差風險，不覆蓋成「已消除背景」的校準值。T2 echo 比 Ramsey 大或小，都不能單獨決定接受或拒絕。
