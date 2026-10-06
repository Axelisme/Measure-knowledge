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

## 分別驗收 reset 的收益

初態分布、採樣時間與目標估計的 precision 要分開驗證。更純的 g/e 初態及更短 relax_delay 可以改善對比和吞吐；若要主張 averages 可減少，還需比較固定目標不確定性所需的 shots／時間，或固定 shots 下的不確定性。不能只由 ground model estimate 推出所需平均數。

保持 drive 與 readout 相同，比較 passive 長等待與 reset 短等待，再用新 seed 重驗。兩項同時改變時，驗證的是整組初始化流程，不能把 gate 差異唯一歸因於 reset tone。Q1 half 案例的 reset 提高初態純度並縮短採樣，但 matched X180 的均值在獨立重驗仍較低；統計 CI 有小範圍重疊，時段、模型偏差及具體機制未排除。這是重新驗證 gate 的反例，不是 reset 普遍降低 gate 品質的定律。

對 photon／AC Stark 假說，延長 ringdown 後 zigzag 未改善，僅限制所測條件下的解釋。帶自由 phase 的長窗口 Ramsey 未見顯著頻率差，仍可能沒有解析 gate 早期的快速 transient；不可據此宣布已排除 photon effect。Readout gain 改變時，也要重新核對 IQ centres／classifier，避免把讀出尺度變動混成初始化或 rotation 的變動。

## 依據與限制

2026-10-06 使用者在 Q12_2D[10]/Q1 half-flux 任務提出 singleshot→MIST→single-tone length→reset module→reset_check 路線，指出 reset 可提升初始 g/e 純度並降低等待與平均需求，且 photon decay 採 5*2pi/kappa、5µs 太長。這些是當次專家指示，不是跨裝置通用參數。

[Q1 真實案例](cases/q1-half-20261006/README.md) 保留 no-init 標籤校正、順序 reset、驗證圖及收尾 matched IRB 反例。具體功率、freq、duration 與估計純度只適用該次條件；完整 gate 比較以 task 最終報告與保存 IRB 為準。
