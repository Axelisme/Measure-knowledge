# 量測知識導航

## Bring-up 與工作點

從空白或未驗證設定開始，決定量測順序、何時回頭校準、何時停止時，讀 [coherence 決策流程](experiences/coherence-bringup/README.md)。包含真實硬體前置檢查及模擬案例的適用邊界。

Flux map 有 extrema 但分支不確定，或 two-tone 自動 fit 選到雜訊、旁峰時，讀 [flux 與 spectroscopy 驗證](experiences/flux-spectroscopy-validation/README.md)。

## Pulse calibration

判讀 Rabi 的週期、第一個峰、參數誤差或 repeat，以及振盪存在但 fit 不符時，讀 [Rabi fit 驗證](experiences/rabi-fit-validation/README.md)。條目說明同一 Run 的模型比較，並保留 Length Rabi mock 案例與限制。

## Coherence

判讀 T1 的時間窗口、尾端基線、模型適用性、參數誤差或 repeat 時，讀 [T1 fit 驗證](experiences/t1-fit-validation/README.md)。條目區分重新分析與補量測，說明共用品質指標的限制。

Ramsey／echo 的 baseline 隨 delay 變化、phase 改變後 T2 不一致，或要判斷 IQ 差分是否適用時，讀 [coherence 背景辨別](experiences/coherence-background-validation/README.md)。

## 設定與資料來源

更改 library 後擔心 local override、請求軸與實際軸不符、操作 timeout，或準備保存與 writeback 時，讀 [校準來源核對](experiences/calibration-provenance/README.md)。具體工具操作仍以 live guide 為準。

## 維護導航

新增入口時，說明它處理什麼判斷問題、何時值得讀，以及連到哪個經驗資料夾的 README。正常表現與異常辨別都可以成為入口，不必先有診斷名稱。

同一條經驗可以從多個 domain 連入，正文只維護一份。入口多到難以瀏覽時再拆分 domain 文件，不預先建立完整分類樹。
