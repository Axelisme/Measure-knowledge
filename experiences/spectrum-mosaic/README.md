# 多段 spectroscopy 的合图與追溯

## 何時使用

要將不同頻帶、電流窗口或掃描方向的twotone fluxdep整合成一張圖時使用。主圖呈現量到的範圍與分支，不把不同條件的資料合成單次實驗。

## 整理流程

1. 從已保存raw讀實際frequency/current軸、單位、IQ與當次cfg；不從目前tab設定回推舊Run。特別核對Hz/MHz、A/mA、升降方向、NaN與中止範圍。
2. 建來源manifest，至少保留raw、方向、完整點數、channel/NQZ/mixer、probe gain/length、readout、reps/rounds與取捨理由。較粗survey、控制實驗、錯誤頻段與部分中止資料仍保存，但不默認全部疊進主圖。
3. 若扣背景，記錄方法與頻率窗口。滑動中位數可能削弱寬線，IQ magnitude可能失去正負資訊；它們適合呈現可見度，不直接當population或精確fit輸入。判讀寬弱區仍回看原始I/Q。
4. 不同gain、readout、channel下的強度不可直接互比。可以每段獨立正規化，但必須在圖上寫明，色條亦標為每段normalization。同一數字gain不保證不同channel有相同物理功率。
5. 明訂重疊優先順序，例如較細flux網格優先、同解析度採較新批次。這只是展示規則，不能當成較新必然較準；有歷史差異時另留分pass比較圖，不平均或平移掉差異。
6. 未顯示資料的區域留白，不在不同掃描矩形間補插值。1D補測畫窄欄或獨立點並標記，不能把一條1D spectrum鋪成大面積已量區域。標準像素面積代表取樣顯示範圍，不是額外量測點。
7. 加整體圖及必要局部放大。工作點註記附來源；自動追蹤的可疑候選不進入已確認f01標記。輸出PNG/PDF後檢查標籤、色條、圖例是否裁切。

## Fit與合圖分開驗收

有先驗搜尋窗也不保證自動peak finder選對線。若fit偏離相鄰分支，先核對該欄raw、I/Q與殘差，必要時比較候選峰和窗口；不能只因formal error小就接受，也不能為符合預期而強制移峰。先驗只提供搜尋範圍，不是測量證據。

Q1 rawflux17在7.1mA的自動局部fit候選為303.489MHz，而相鄰pass的分支線索約325MHz；該候選尚未完成原始單欄判讀，因此僅標為可疑、不接受為f01。合成圖顯示raw，沒有加入這個fit標記。這是待解的分析警示，不是已證實的假峰或新躍遷。

## 案例與可移轉限制

2026-10-05 Q12_2D[10]/Q1合圖，檢查17份2D raw、選10段作主圖並疊15個已核對local點。主分支integer約4.631GHz、half約309MHz；5.6mA寬弱區未用插值补成精確f01。各段獨立背景與色階，未作flux漂移校正。

來源為量測repo `.agent_state/measurement-tasks/20261005-q1-twotone-fluxdep/`：`build_overview.py`、`segment_manifest.json`、`sequential_points.json`、`spectrum_overview.png/pdf`、`descending_half_fits.json`。Raw在 `Database/Q12_2D[10]/Q1/2026/10/Data_1005/`。這些是repo-local來源，跨主機先核對可讀性。腳本只讀raw並輸出衍生圖與manifest，沒有硬體操作；其固定檔名、suffix排除與排序規則屬該案例，不能直接作其他任務的通用筛選器。
