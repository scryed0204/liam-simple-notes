---
title: "流程引擎 - Activiti簡介與測試"
category: Activiti
tags:
  - table of contents
---

簡介Activiti以及其開發環境的建置。
<!--more-->

{% include toc %}

##  淺談BPMN
BPMN是Business Process Model and Notation的縮寫，可以翻譯成「業務流程模型與標記法」，是用於表現「業務流程」的圖型化標準，如下圖:
![image-center]({{ '/images/20200309/20190619175852.PNG' | absolute_url }}){: .align-center}

上圖就是利用BPMN 2.0規範去把一個業務流程給描繪出來，與UML的觀念類似，BPMN提供一種圖像化的語言，讓使用者、系統分析師或是工程師等不同角色的人都能以此作為溝通的媒介。不過，以開發者的角度來說，與客戶溝通時畫了一張流程圖，在談妥後又要把系統實作出來，可能又是一個艱鉅的任務，如果能有個工具，可以把用來「分析討論」與「系統開發」的媒介整合在一起，這樣一來既方便，同時也能控制實作出來的結果不會與談好的需求有所落差。透過Alfresco推出的流程引擎Activiti，上述的事情便可以辦到。

Activiti是一套用java開發的開源流程引擎，它根據BPMN 2.0的規範實作出商業流程管理之解決方案，讓使用者可以針對企業流程做集中化的管理。在實際應用面上，開發者把Activiti的相關jar與對應的資料庫資訊定義好了以後，便可以把寫好的流程定義檔(.bpmn)透過它的API佈署上去，未來使用者就可以透過它的API加上流程定義的key值與Activiti溝通，來讓Activiti幫忙控管流程動向與歷史記錄。

本文著重在介紹以Java開發者的角度如何建置一個開發環境，用一個簡單的需求來當作範例，介紹用Eclipse來編輯它的流程圖檔(.bpmn)，並寫一支單元測試來執行它，同時認識怎麼使用Activiti的核心API。至於BPMN 2.0完整的規範以及Activiti 引擎的完整的API功能，在此並不會做全盤的介紹，有興趣的請參考以下網站
  1. BPMN 2.0 規範 － [OMG協會網站](﻿https://www.omg.org/index.htm "OMG協會網站")
  2. Activiti完整功能介紹 － [Activiti User Guide](﻿https://www.activiti.org/userguide/ "Activiti User Guide")

##  Eclipse安裝Plugin
Eclipse是許多java開發者會選擇使用的IDE，而Alfresco針對Eclipse提供了Activiti BPMN Designer這個外掛，讓開發者們可以用圖型介面來設計流程定義檔(副檔名為.bpmn)。如果用文字編輯器打開.bpmn的話，會發現它是由XML語言格式的檔案，透過Activiti引擎能將它解讀成可實際運作的流程定義，在完持佈署後即可透過API起案，產生流程實體與相關任務實體…等等。 

本篇使用Eclipse版本: Eclipse Neon 3 (測試後，確定此版本可以用下述方式安裝Activiti BPMN Designer)，Java環境為1.8，另外使用了Maven來作為建置工具。

安裝方法：
  1. Eclipse上方Menu，點擊Help → Install New Software，開啟安裝Plugin的介面。
  2. 點擊Add開啟小視窗，於Location輸入下列網址資訊http://activiti.org/designer/update/ ，如下圖：![image-center]({{ '/images/20200309/20190620140010.PNG' | absolute_url }}){: .align-center}
  3. 找到Activiti BPMN Designer後，循序完成安裝。
  4. 安裝完成後重啟Eclipse，可以看到Activiti的Perspective出現了，切換過去後就可以繪製流程圖。![image-center]({{ '/images/20200309/20190620140228.PNG' | absolute_url }}){: .align-center}
 
##  開啟專案與新增流程圖
安裝好plugin之後就可以新增Activiti的專案了，請依照以下步驟：
1. 新增Activiti專案，如下圖：![image-center]({{ '/images/20200309/20190620140955.PNG' | absolute_url }}){: .align-center} 備註：Activiti專案本身有帶POM檔，請記得把專案轉成Maven專案，如下圖：![image-center]({{ '/images/20200309/20190624161105.PNG' | absolute_url }}){: .align-center}
2. 至 /src/main/resources/diagrams/ 底下新增Activiti Diagram (即.bpmn)，如下圖：![image-center]({{ '/images/20200309/20190620141034.PNG' | absolute_url }}){: .align-center}
3. Activiti的Perspective開啟流程圖檔(.bpmn)，即可編輯。再開始繪圖之前可以先為流程圖設定一個ID，ID本身是會依據檔名設定一個預設 值，如果要編輯ID，請點擊流程圖任意一個空白位置，看到底下Properties -> Process -> Id的欄位後，即可自行定義，而Name這個欄位定義的是流程圖的名稱，有需要也可以調整。如果找不到設定位置的話請參考下圖：![image-center]({{ '/images/20200309/20190624181752.PNG' | absolute_url }}){: .align-center}
4. 開始繪圖，可在畫面右方的Palette中尋找各個BPMN 2.0所定義的圖示，再用拖拉的方 式拖到畫面中即可，請參考下圖：![image-center]({{ '/images/20200309/20190620141411.PNG' | absolute_url }}){: .align-center}
5. 每個圖型之間需要用箭頭連接起來，表示動向。
![image-right]({{ '/images/20200309/20190620141444.PNG' | absolute_url }}){: .align-center}
6. 利用介面下方的Properties可以編輯每個項目的內容，如顯示的文字，以及後面會提到的表單屬性(Form Properties)、判斷條件(Condition)與監聽器(Listener) ...等等。![image-center]({{ '/images/20200309/20190620141909.PNG' | absolute_url }}){: .align-center} 

##  依照需求繪製流程圖
繪製流程圖的準備手續與基本方法已經介紹完成，現在我們來把需求定義好，才知道到底要畫什麼內容。假設我們與客戶談到一個企業流程，流程的邏輯如下：
1. 起案人送出案件表單，會到自己的直屬主管進行簽核。
2. 案件內容中包含了總金額的資訊，將會影響流程結束的關卡。
3. 如果金額未大於10萬元，則簽到直屬主管即可，簽核後流程結束。
4. 如果金額大於10萬元，則直屬主管簽核後還必須給經理過目，經理簽核後流程才結束。
5. 不論是直屬主管審核或者是經理審核，都可以退回給申請人調整案件內容，重送後一律再送給直屬主管。
6. 退回給申請人後，申請人也可以選擇終止案件，使流程以終止方式結束。

根據上述的需求，我們便可以開始進行流程圖(.bpmn)的繪製。在此先提供畫好的圖型，後續再做解釋：
![image-center]({{ '/images/20200309/20190620143432.png' | absolute_url }}){: .align-center}
上圖中各個圖型所代表的意義，整理如下表:

| 圖型   | 名稱   | 意義                                                         |
|------- |--------|--------------------------------------------------------------|
| ![image-center]({{ '/images/20200309/20190623172041.PNG'| absolute_url }}){: .align-center}   | Start Event 開始事件   | 一般的流程開始事件，無特殊觸發條件，由使用者呼叫api來啟動流程時的開始點。|

