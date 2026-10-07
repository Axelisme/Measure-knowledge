# 量測知識導航

## Bring-up 與工作點

接手多flux校準、需要完整工作路線與各實驗核對入口時，從 [經確認的十項流程](experiences/coherence-bringup/README.md) 進入，再依症狀展開下列條目。

從空白或未驗證設定開始，決定量測順序、何時回頭校準、何時停止時，讀 [coherence 決策流程](experiences/coherence-bringup/README.md)。包含真實硬體前置檢查及模擬案例的適用邊界。

Flux map 有 extrema 但分支不確定，或 two-tone 自動 fit 選到雜訊、旁峰時，讀 [flux 與 spectroscopy 驗證](experiences/flux-spectroscopy-validation/README.md)。

大幅flux移動後頻率偏移、連續掃描的分段銜接，或逐點診斷拖慢主光譜進度時，讀同條目的[批次fluxdep與單點診斷](experiences/flux-spectroscopy-validation/README.md#批次-fluxdep-與單點診斷的分工)。

有限預算下安排 map、pulse 與 coherence，或需要一份帶實測反例的完整路線時，讀 [Q1 真實 bring-up 案例](experiences/coherence-bringup/cases/real-integer-20261005/README.md)。案例含來源清單和精選圖；具體設定不作通用模板。

## Pulse calibration

需要完整reset→gate異常調查、候選排除邊界、可重現流程與波序列時，讀 [reset調查流程](experiences/cavity-reset-validation/investigation-playbook.md)與[Q1四輪整合](experiences/cavity-reset-validation/cases/q1-half-investigation-20261007/README.md)。

以 cavity tone 準備非熱平衡初態、計算 photon ringdown、核對 singleshot 預設 init 或 reset 後 gain 改變時，讀 [cavity reset 驗證](experiences/cavity-reset-validation/README.md)。包含錯誤物理標籤、雙 tone 時序與獨立 gate 檢查的真實反例。

Ringdown 拉長仍有 zigzag transient／累積 parity、需要區分初態 coherence、drive-on detune及shot歷史時，讀同條目的 [辨別控制](experiences/cavity-reset-validation/README.md#ringdown-足夠但-zigzag-仍異常時)，以及帶原始來源的 [Q1 後續診斷案例](experiences/cavity-reset-validation/cases/q1-half-20261007/README.md)。

需要核對ZCU216/QICK phase frame、固定shot frame仍有scan方向差、RF tail跨shot效應或Rabi模型未通過echo holdout時，讀[獨立機制排查案例](experiences/cavity-reset-validation/cases/q1-half-mechanism-20261007/README.md)。

需要分析pump頻率/gain/長度、辨別initialization contrast與gate response、等gain²T反例或ABBA方向排查時，讀[三參數與actual-gate驗證案例](experiences/cavity-reset-validation/cases/q1-half-pump-dependence-20261007/README.md)。

Readout gain 最佳點碰到掃描邊界、length 要在短窗口與 SNR 間取捨，或準備 two-tone／flux map 的讀出條件時，讀 [readout 優化](experiences/readout-optimization/README.md)。

判讀 Rabi 的週期、第一個峰、參數誤差或 repeat，以及振盪存在但 fit 不符時，讀 [Rabi fit 驗證](experiences/rabi-fit-validation/README.md)。條目說明同一 Run 的模型比較，並保留 Length Rabi mock 案例與限制。Zigzag／AllXY 交叉檢查、gain 候選不一致與 AllXY 誤差指標的限制也由此進入。

選 gain 以符合 π／π2 時長、區分執行時設定下限與已量化座標，或規劃 zigzag 獨立檢查，也從同一 [Rabi 條目](experiences/rabi-fit-validation/README.md) 進入。

## Coherence

RB 淺depth即崩落、最大depth改變共同prefix結果、IRB候選篩選／新seed重驗，或要確認fidelity的單位與序列統計時，讀 [RB執行與擬合驗證](experiences/rb-validation/README.md)。含真實壓縮查表對照、IRB reference比較與統計CI的適用限制。

判讀 T1 的時間窗口、尾端基線、模型適用性、參數誤差或 repeat 時，讀 [T1 fit 驗證](experiences/t1-fit-validation/README.md)。條目區分重新分析與補量測，說明共用品質指標的限制。

Ramsey／echo 的 baseline 隨 delay 變化、phase 改變後 T2 不一致，或要判斷人工detune、IQ差分及每週期採樣是否足夠時，讀 [coherence 背景辨別](experiences/coherence-background-validation/README.md)。Rabi以外的Zigzag／AllXY交叉檢查與mock案例，見 [Rabi fit驗證](experiences/rabi-fit-validation/README.md#交叉檢查pulse而不只檢查rabi-fit)。

T1 慢尾端與 zero-drive reference 的設計，讀 [T1 決策流程](experiences/t1-fit-validation/README.md)。Echo 人工 detune、fringe 取樣及 GUI／離線模型差異，讀 [coherence 擬合辨別](experiences/coherence-background-validation/README.md)。

比較多個flux點的單／雙指數T1、檢查雙分量可辨識性，或區分正常點數與模型分類時，讀 [跨flux T1模型與品質分類](experiences/t1-fit-validation/README.md#跨-flux-的單雙指數比較與品質分類)。

## 設定與資料來源

多段twotone fluxdep要合成一張圖、選重疊來源、保留缺口或判斷色階是否可比較時，讀 [多段光譜合圖](experiences/spectrum-mosaic/README.md)。

小步連扫仍有峰位跳變、half最低位置在不同pass不一致，或要規劃從integer到half的整體路線時，讀 [flux驗證與主流程決策樹](experiences/flux-spectroscopy-validation/README.md)。

更改 library 後擔心 local override、請求軸與實際軸不符、操作 timeout，或準備保存與 writeback 時，讀 [校準來源核對](experiences/calibration-provenance/README.md)。具體工具操作仍以 live guide 為準。

## 維護導航

新增入口時，說明它處理什麼判斷問題、何時值得讀，以及連到哪個經驗資料夾的 README。正常表現與異常辨別都可以成為入口，不必先有診斷名稱。

同一條經驗可以從多個 domain 連入，正文只維護一份。入口多到難以瀏覽時再拆分 domain 文件，不預先建立完整分類樹。
