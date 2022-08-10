# Computer-Organization-Final-Project

利用gpio來輸入按鈕的input跟輸出LED燈的控制訊號 再把遊戲畫面輸出到LCD上顯示 我們有設計Normal Mode 跟 Timer Mode 並限制猜測次數
在Timer Mode中是利用Timer1 每一秒發出中斷來實作倒數計時



Demo
我們總共使用了11個按鈕 (Control跟0~9)
Control按鈕有三個功能 分別是Start Enter跟Restart 在後面用到的時候會分別說明

一開始按鈕9跟8是代表讓user選擇想要玩哪個模式(9是Normal Mode  8是Timer Mode) 
Normal Mode是讓user去猜答案是多少並限制猜的次數為10次
Timer Mode則是會再加上180秒的時間限制
先從Normal Mode開始demo


接著再按Control按鈕開始遊戲(此處的功能是Start)

此時LCD會顯示初始的範圍與可以猜的次數
在user可以輸入答案的時候Control的燈會亮起 此時按鈕0~9分別代表輸入2的0~9次方 對應的LED則代表當前這個數字是否有被輸入 
若輸入後想取消再按一次按鈕即可 LED也會暗掉

在決定好要猜多少後按下Control按鈕把guess回傳(此處Control的功能是Enter)
接著會與答案比對 

若與答案相同LCD底色會轉為綠色並顯示Congratulations!並結束遊戲
接著等待Control按鈕被按下(此處的功能是Restart)

若與答案不同則會依猜測的數值來決定LCD所顯示的文字並繼續遊戲
若小於答案且大於等於下界的話 會將下界的值更新為guess+1並改變顯示的範圍


若大於答案且小於等於上界的話 會將上界的值更新為guess-1並改變顯示的範圍
若超出上界或下界的話會顯示Out of Range!!

若guess跟答案差不到50的話會將LCD底色轉紅並閃爍

若在猜錯的同時猜的次數剩餘三次的時候則會將LCD底色轉紅並閃爍
猜錯的同時猜的次數用完的話則會關閉閃爍並顯示You Lose!!並結束遊戲
接著等待Control按鈕被按下(此處的功能是Restart)


再來是Timer Mode
一開始除了顯示範圍跟次數以外還會在右下角多顯示秒數

操作與Normal Mode相同

但在秒數剩餘50秒的時候會將LCD底色轉紅並閃爍60

若秒數等於0的時候則會關閉閃爍並顯示You Lose!!並結束遊戲10

接著等待Control按鈕被按下(此處的功能是Restart)


Code analysis
我們的程式主要可以分成四個部分 
Global variable
Main function
	初始化 無限迴圈
Input function
Timer function
其中Timer function是由checkint跟isr所組成。

Global variable:
在這邊放的是會在不只一個地方用到的變數。
lcd_obj: LCD的物件(Main, Input, checkint)
ifint: 表示有沒有進過Interrupt(isr, checkint)
count_down: 表示剩餘的秒數(Main, Input, checkint)
chance: 表示剩餘可以猜的次數(Main, Input)
mode_state[2]: 表示現在是什麼模式(Main, checkint)

Main function:
初始化:
分成在無限迴圈內跟外。
外:get GPIOs object跟設定pin腳的狀態還有初始化IIC 0
內:一些在restart之後需重設的變數

無限迴圈:
最外層的是為了實作restart。
在其中有一些接在吃按鈕input後面的是拿來卡住的，如果按鈕持續被按著的話就會無法跳出迴圈。
若當前為Timer Mode會初始化Timer1跟印出開始的秒數，接著會顯示初始範圍跟總共可以猜幾次。
再來就開始猜並將結果回傳給guess。
接下來的無限迴圈是根據回傳的值印出不同的文字到LCD上並繼續遊戲，或是印出You Lose!!或Congratulations!並跳出迴圈等待restart。在迴圈的各個if中會穿插checkint()來看是否剩餘秒數為0。

Input function:
進去先把燈都初始化為暗，然後進while迴圈開始讓user去輸入想猜多少並印出來到LCD上。按一下按鈕guess會加那個按鈕代表的值，再按一下則會再扣回來。
決定好要猜什麼後，按下Control按鈕跳出迴圈並把chance – 1，如果chance為3的話就把LCD設為紅色閃爍最後再把guess回傳。
在function中的各個位置也會放上checkint()來查看是否剩餘秒數為0。

Timer function:
在isr中只會將IP clear跟ifint設為true，代表有進中斷。
在checkint中如果是Timer Mode且ifint == true的話，會把秒數減一並印在LCD的右下角。數到剩50秒的話會將LCD設為紅色閃爍，數到剩0秒的話會將ifint設回false並回傳0，其餘狀況的話一樣會將ifint設回false但回傳1。

心得
在這次的project中，我遇到最大的困難是在於如何處理中斷時的行為跟isr裡面該做什麼。因為最早的時候是將秒數print到LCD上放在isr裡面，結果發現會卡在裡面出不來。最後是只在isr中放一個flag紀錄有沒有進中斷，若有則設為true，然後在程式的各個時間點放入check來檢查並print秒數到LCD上。
我也學到了很多，例如如何善用printf的印出指定個數的功能跟設定數字要靠左還是靠右等等。
