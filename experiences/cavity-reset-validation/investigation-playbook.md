# Reset 後 gate 異常：可移轉的調查流程

本流程用於 reset 後出現前端偏向、累積 parity 或序列相關校準差。先讀 [domain 入口](README.md)；這裡集中最小辨別實驗與接受標準，不提供可直接套用的 gain、flux 或等待值。依據是 [Q1 四輪整合案例](cases/q1-half-investigation-20261007/README.md)，不是所有晶片皆有同一根因的聲明。

## 1. 先建立正確的比較單位

原始 pulse、occupied slot、shot period、上一點 history 是四個不同量。固定 relax_delay 不等於固定 period；固定 period 不等於固定 RF 負載。記錄 point/sweep averaging、loop nesting、scan direction、reset/analysis phase 與 frame。對照應說明固定哪些量、只改哪一個干預。

把 waveform channel clock、event-time clock、instruction core、board reference 分開。以 live SoC 和版本固定的公開來源為依據；比較 actual start/end、signed DDS words、rounded mixer 與 gen/ADC common grid。Zero control 保留同 channel、同 slot 的零 gain pulse，不能任意換成純 delay。跨 channel 掃 length 時可選共同整數格點固定末端，避免同時掃到 ringdown。

軟體編譯核對只證明預定事件，不證明真實 DAC delivery。浮點約1e-15的 near-overlap 警告須回到整數 ticks；也不能因為 QICK 能編譯，就推定符合當次最短 pulse 限制。

## 2. 最小因果分離

| 要辨別的問題 | 最小控制 | 接受／限制 |
| --- | --- | --- |
| Reset 是否為累積異常的必要條件 | Cavity on/zero × terminal π on/zero，完整timing相同；各配reset相反phase與probe正反/zero | Zero-both仍有主要正規化形狀，限制MIST專屬主因；低contrast不作精確null |
| 初態相干偏向 | 四analysis phases、reset π 0/180、同長zero probe；另用等待位置控制 | 偏向隨reset phase反號支持phase-sensitive prep；analysis gate也可能受history影響 |
| Photon wait與cycle混淆 | 固定總cycle，把等待放tone前或π後；另固定qubit block移tone | 只有after-tone獨有的可重複差才支持該位置的作用；投影不是已校正人口 |
| Compiler／接縫 | 同總時長continuous pulse、loop/unrolled或獨立native序列 | 異常跨實作重現，該實作不是必要條件；不是所有硬體邏輯皆排除 |
| Effective drive vs detune | Drive-on chevron分窗口fit，核對free curvature與residual | Ω與center的變化分开；vertex可能受RF transfer slope偏置 |

S/Z與reset phase-even/odd要與原IQ對比一起呈現。不要把 odd/even RMS、zero-reset對比比例或IRB差拆成總和100%的原因責任。

## 3. RF history 的辨別順序

1. 固定frame，比較point/sweep averaging和正反掃描；這是history干預，不只是測量顯示順序。
2. 在ADC完成後放active/zero RF tail。當前readout已完成，後續shot差可建立跨shot影響；保持數位slot，避免寄存器／排程混淆。
3. 固定能量與period只改tail recency，再用單pump後的固定probe train，解除length sweep與前一點負載的混淆。
4. 將cavity RF與qubit RF分開掃frequency/gain/length；每個frequency配same-frequency zero gain。固定digital gain不是固定chip功率，不能把曲線直接當吸收譜。
5. 等gain²T配對用來否定總劑量唯一模型。固定末端下不同duration有不同RF年齡分布，因此不重合不單獨證明功率非線性。

單pump實驗的probe/reset/readout train仍持續施加負載。其fit τ是该sequence下的描述量，不是單一元件 impulse response。Zero-probe及no-analysis controls可揭示analysis gate本身的history敏感性；若使用affine兩能級模型，明列GE中心、leakage、軸不對稱等限制。

## 4. 模型與解法分層驗收

先固定係數、來源、窗口與hash，再取新條件。模型應預測完整曲線、相反方向或另一sequence；endpoint相近而early residual有結構仍算未通過。小holdout不能區分兩模型時，保留不可辨識，不只引用較好的training RMS。

遇到大幅兩block差，追加同cfg的ABBA/F-R-R-F區分方向和時段。未重現原差也保留原點與未觀測狀態，不刪掉反例。單點gain/frequency null、Rabi變平、zigzag loss下降與IRB改善分別是不同層级的證據。

相反terminal π phase等權平均，可處理平均transverse preparation偏向而不新增pulse/wait；需每個採集點相位平衡、gate words/timing不變並檢查population/leakage。它不是每shot完美reset，亦不自動修主累積或目標benchmark。

若不同pulse history下的有效drive response仍變，元件定位需要直接RF amplitude/phase取樣，分DAC端及後級鏈路比較。沒有這項證據時保留控制鏈與chip候選，不指定哪顆放大器故障。

## 5. 收尾保留重現能力

正式raw不可改寫；錯軸以旁存actual-axis及source hash更正。每個run保留intent、完成與saved receipt、cfg、ASM、analysis方法、版本及不確定性。政策偏差與失敗模型保留在案例，不用事後選點抹除。

臨時診斷結束後，將source overlay與測試按base commit/hash封存，從正式catalog移除任務入口；保留已驗證的通用bug fix與獨立驗收的功能。完整報告需說明磁碟source與live GUI記憶體可能不同，清理不自動意味硬體已重新載入。

正式模組的回歸測試不屬於應刪的『測試垃圾』。刪除臨時功能時，其測試隨封存；保留功能的cases需有清楚映射與清理後驗證。原始資料、凍結預測與必要離線分析留作稽核。
