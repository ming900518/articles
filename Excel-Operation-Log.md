# 大三統計套裝軟體（Excel）上課操作記錄

記錄一下老師上課到底教了啥操作......快速鍵前面放Windows後面放Mac的

## 快速導引
- [2021/09/25](#2021/09/25)
- [2021/10/02](#2021/10/02)
- [2021/10/09](#2021/10/09)

## 內文

### 2021/09/25

- [空白處填滿](https://user-images.githubusercontent.com/15919723/134762368-ef3c400d-45cf-47a3-97b5-b284bb8f0a70.mov)  
  快速鍵：Ctrl + Enter / Command + Return
- [貼上原值](https://user-images.githubusercontent.com/15919723/134762787-76d321ba-1544-406b-9b0c-8a8c94159524.mov)
- [建立樞紐分析表 + 調整欄位](https://user-images.githubusercontent.com/15919723/134763542-3e66931c-6904-4ce6-a510-64380b3b8e46.mov)

### 2021/10/02

- 自訂排序
   <img width="1440" alt="Screen Shot 2021-10-02 at 1 43 05 PM" src="https://user-images.githubusercontent.com/15919723/135705132-45139e0a-9b2b-4264-83c9-b69942f6e9ea.png">
   
- 排序順序自訂清單
  <img width="1440" alt="Screen Shot 2021-10-02 at 1 46 53 PM" src="https://user-images.githubusercontent.com/15919723/135705211-46137f45-e70b-4d83-a4dc-5e20ae6fc7ba.png">
  
- 多重小計：先做一次「資料」-「小計」，之後再做一次小計時取消選擇「取代目前小計」即可
  <img width="1440" alt="Screen Shot 2021-10-02 at 2 20 05 PM" src="https://user-images.githubusercontent.com/15919723/135706004-907e9af5-b806-4103-9cba-09b1de570b51.png">
  
- 表格
    1. 「插入」-「表格」-Excel會自動帶入範圍，按下確定（有標題的表格如果不選，Excel會自己加標題）。點完之後，標題會出現下拉式選單可以做排序跟調整，也會出現「表格工具」功能。  
      <img width="1440" alt="Screen Shot 2021-10-02 at 2 30 17 PM" src="https://user-images.githubusercontent.com/15919723/135706262-a5fa9621-3e4e-4baf-8874-4ae95fd6d793.png">
        
    2. 點出「合計列」會在最後一列加入最後一行的加總，可以用下拉式選單調整內容  
      <img width="99" alt="Screen Shot 2021-10-02 at 2 38 14 PM" src="https://user-images.githubusercontent.com/15919723/135706405-2f4b88f5-bc21-4135-953d-8a664062a53d.png">
      
    3. 合計格子右下角黑點滑鼠移過去變黑色加號後按住拉動可以增加其他合計  
      <img width="1440" alt="Screen Shot 2021-10-02 at 2 43 11 PM" src="https://user-images.githubusercontent.com/15919723/135706531-bda2acbd-2981-45e9-99c8-aeef1ede5c1f.png">
      
    4. 在最後一列的儲存格按Tab可以新增新列

    5. 在最後一欄的儲存格拉儲存格右下角的黑點可以新增新欄
      
    6. 在表格中的欄位使用函數，會自動套用到整行
      <img width="1440" alt="Screen Shot 2021-10-02 at 2 56 25 PM" src="https://user-images.githubusercontent.com/15919723/135706889-89cd6336-d30d-4914-b7dd-fdb4c04b1307.png">
      
    7. 「常用」-「設定格式化」功能可以在選取的表格中新增不同的效果，並可透過「醒目提示儲存格規則」和「頂端/底端項目規則」進行調整  
      1. 資料橫條（進度條）
        <img width="1440" alt="Screen Shot 2021-10-02 at 3 18 50 PM" src="https://user-images.githubusercontent.com/15919723/135707478-1cdb111e-ae06-462b-a2d3-453e3aac12ef.png">
      2. 色階
        <img width="1440" alt="Screen Shot 2021-10-02 at 3 19 07 PM" src="https://user-images.githubusercontent.com/15919723/135707488-28307b59-ddb1-4a0d-a8c2-4e19e1796e7d.png"> 
      3. 圖示集
        <img width="1440" alt="Screen Shot 2021-10-02 at 3 20 38 PM" src="https://user-images.githubusercontent.com/15919723/135707529-fce5097a-c72a-43bf-94e7-89899e22b5da.png">

### 2021/10/09

- 篩選
  (1)
  1. 數字篩選(3)(2)
  2. 依色彩篩選(4)
  3. 前幾位篩選(5)(6)
  4. 介於篩選(7)(8)

  Q: 金額小於600大於10000要怎麼按
  A: (9)

  ``
  SELECT * FROM table WHERE 金額 > 10000 AND 金額 < 600; # 且
  SELECT * FROM table WHERE 金額 > 10000 OR 金額 < 600; # 或
  ``

  5. 日期篩選(10)
  6. 局部篩選（後1/3）：先選要篩的那排，點那一列的資料，再按篩選(11)
  7. 進階篩選：先做好一般篩選，按進階(12)
  8. 交叉篩選：插入->表格->交叉分析篩選器

