# pdf_divider
將pdf檔中的中文字分段，每段字數不超過512。
A tool to divide chinese text in a pdf into blocks. Each block's length is smaller than 512 for most language model.  
**主要使用語言：python**  
**執行檔資料夾連結**：https://drive.google.com/drive/folders/1xEuwa94bgNYEE1yrdXXP6pZNwNCsLdu5?usp=share_link
## 使用方法：
1. 將連結中的.exe檔和Output資料夾同時下載
2. 點擊.exe檔後出現簡易視窗
3. 按 'Select pdf' 選擇欲分割的pdf檔案
4. 按 'Start' 後結果將輸出於Output資料夾中 
## 分段邏輯&流程
### pdf_divider (main function)

輸入：欲分割的pdf檔  
輸出: (filename)_divided.csv

1. **遍歷每個 page** 進 **process_one_page**  function，回傳初次分段的結果 type: list (list 裡存一個個 string)
2. **資料清理：**
    - 去除 \<image:...>
    - 刪除換行(\n)、縮進(\t)
    - 刪除標點、英文、數字、換行、� 等非文字的符號（留下逗號、句號以方便後續分段）
      - �: fitz 套件無法辨識pdf中的文字

3. 去除**空字串** & **長度小於2**(含)的字串
4. 處理 **block 長度大於 512** 問題 => **用標點分段**
    - 判斷字串長度決定應該分幾段
    - 用逗號、句號分段
5. 處理 **block 長度大於 512** 問題 => **直接切分**
    - 有些段落==沒有逗號、句號==所以無法以第五步驟分段 => 採取直接切分的手段
    - 例：若段落有650字，則前450(可調整)一段，剩下的一段

### process_one_page function
1. 以 `page.get_text("blocks")` 方式讀取 page 的內容
    - 如果讀不到文字 -> return
    - 讀的到 -> 繼續以下步驟
2. 將讀取結果存成 DataFrame，x 軸座標四捨五入到整數位，並按 x 軸座標大小排序
![dataframe.png](https://i.imgur.com/LFWGpGH.png)
3. 以 x 軸座標大小和正負一的range分段
    - 例：ｘ軸座標在750 +- 1的範圍內為一段
    
    
    **分段規則設立依據**：
    同一個段落的文字通常會向左右兩條垂直軸對齊
    ![](https://i.imgur.com/6hUcGSG.png)

    如圖中文字向左方**紅線**對齊
    
4. 若第三步分完的段落超過 5 行 -> 用 y 軸座標分段，方法如下
    (1). df 按y軸座標大小排序
    
    (2). 算前後兩行的距離
    ![](https://i.imgur.com/0YucFm9.png)
    
    (3). 若行距太大則歸為下一段(算離群值來判斷行距是否太大)   
5. **回傳**初次分段結果
