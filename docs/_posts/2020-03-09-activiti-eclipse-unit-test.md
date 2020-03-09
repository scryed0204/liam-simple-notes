---
title: "流程引擎 - Activiti簡介與測試"
category: Activiti
tags:
  - table of contents
---

簡介Activiti以及其開發環境的建置。


{% include toc %}

##  淺談BPMN
BPMN是Business Process Model and Notation的縮寫，可以翻譯成「業務流程模型與標記法」，是用於表現「業務流程」的圖型化標準，如下圖:
![image-center]({{ '/images/20190619175852.PNG' | absolute_url }}){: .align-center}

上圖就是利用BPMN 2.0規範去把一個業務流程給描繪出來，與UML的觀念類似，BPMN提供一種圖像化的語言，讓使用者、系統分析師或是工程師等不同角色的人都能以此作為溝通的媒介。不過，以開發者的角度來說，與客戶溝通時畫了一張流程圖，在談妥後又要把系統實作出來，可能又是一個艱鉅的任務，如果能有個工具，可以把用來「分析討論」與「系統開發」的媒介整合在一起，這樣一來既方便，同時也能控制實作出來的結果不會與談好的需求有所落差。透過Alfresco推出的流程引擎Activiti，上述的事情便可以辦到。



