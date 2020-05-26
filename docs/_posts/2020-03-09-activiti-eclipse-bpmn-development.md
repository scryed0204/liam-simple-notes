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
  1. BPMN 2.0 規範 － [OMG協會網站](https://www.omg.org/index.htm "OMG協會網站")
  2. Activiti完整功能介紹 － [Activiti User Guide](https://www.activiti.org/userguide/ "Activiti User Guide")

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
| ![image-center]({{ '/images/20200309/20190623172041.PNG'| absolute_url }}){: .align-center}   | Start Event 開始事件 | 一般的流程開始事件，無特殊觸發條件，由使用者呼叫api來啟動流程時的開始點。|
| ![image-center]({{ '/images/20200309/20190623172108.PNG'| absolute_url }}){: .align-center}   | End Event 結束事件 | 表示流程正常結束的事件。如果一個流程有分支產生且各自有自己流程上的結束事件時，則要等全部的結束事件都到達此流程才算真的結束。|
| ![image-center]({{ '/images/20200309/20190623172126.PNG'| absolute_url }}){: .align-center}   | Terminate End Event 終止結束事件 | 表示流程被強制終止的事件。如果流程有開啟分支，走入這個事件則全部分歧都被終止。|
| <img src="{{site.baseurl}}/images/20200309/20190623172236.PNG" alt="image-center" class="align-center" width="300" />  | User Task 人工任務 | 一個流程最主要的構成元素就是活動(Activity)，而User Task便是其中一種活動，此種活動需要有一個受理人或受理的群組來承辦，Activiti會依照設定產生一個或多個待辦任務。|
| ![image-center]({{ '/images/20200309/20190623172333.PNG'| absolute_url }}){: .align-center}  | Exclusive Gateway 單一(排它)閘道 | 閘道表示流程的決策點，判斷接下來流程要怎麼走，也依照性質分有不同的閘道，此範例皆使用「排它(Exclusive)」的閘道，會限制流程經過閘道後只會往一條線移動。|

## 定義流程變數與表單屬性
在把流程圖繪製好了以後，為了要讓Activiti可以正確解讀流程怎麼運作，我們還需要定義好其他必要的資訊，首先要規劃「流程變數」有哪些。以此需求為例，在起案時，我們至少必須告訴Activiti起案人的資訊、直屬主管的資訊、經理的資訊與案件的金額；而在各個User Task時，各個使用者必須告訴Activiti審核結果是什麼；如果是退回到了申請人時，由於申請人可以調整案件內容，所以還能再提供一次案件金額。因此依照上述的內容，我們可以規劃出下列流程變數：
1. initiator -   起案人資訊(起案時提供)
2. supervisor  - 直屬主管資訊(起案時提供)
3. manager - 經理資訊(起案時提供)
4. amount - 案件金額(起案時與申請人調整時提供)
5. decision  - 審核結果(各個User Task提供)

規劃好流程變數之後，開發者就可以利用Plugin圖型介面中的Properties功能來定義表單屬性(Form Properties)，如下圖：
![image-center]({{ '/images/20200309/20190623174153.PNG'| absolute_url }}){: .align-center}

由於流程圖檔(.bpmn)的內容其實為XML格式，我們可以用相關編輯器開啟。以下為新增好表單屬性後，將「開始事件(Start Event)」以XML格式呈現出來的樣貌：
```xml
<startEvent id="startevent1" name="Start">
  <extensionElements>
	<activiti:formProperty id="initiator" name="起案人" type="string" variable="initiator" required="true"></activiti:formProperty>
	<activiti:formProperty id="supervisor" name="直屬主管" type="string" variable="supervisor" required="true"></activiti:formProperty>
	<activiti:formProperty id="manager" name="經理" type="string" variable="supervisor" required="true"></activiti:formProperty>
	<activiti:formProperty id="amount" name="金額" type="long" variable="amount" required="true"></activiti:formProperty>
	<activiti:executionListener event="start" class="org.activiti.example.bpmn.listener.ExampleStartEventListener"></activiti:executionListener>
  </extensionElements>
</startEvent>
```

將流程變數定義好了之後，我們可以在一些地方馬上引用這些變數：

1.設定User Task的受理人(Assignee)
 
   自開始事件(Start Event)中的表單屬性中取得了三種角色的人員資訊後，就可以把這些變數設定到對應的User Task中，其設定位置在Properties -> Main config -> Assignee欄位，以${變數名稱}的表示填入，如下圖：
![image-center]({{ '/images/20200309/20190623175049.PNG'| absolute_url }}){: .align-center}
   以XML格式呈現的樣貌如下：

```xml
<userTask id="directSupervisorReview" name="直屬主管審核" activiti:assignee="${supervisor}">
  <extensionElements>
    <activiti:formProperty id="decision" name="決議" type="string" variable="decision" required="true"></activiti:formProperty>
  </extensionElements>
</userTask>
```

2.設定判斷條件(Condition)

   前面在繪製此流程圖就已經知道，流程在通過閘道(Gateway)時是只會選擇一條路來走的(因為用的都是排它型的閘道)，為了要讓Activiti知道流程離開閘道時到底要沿著哪條線移動，我們就必須定義判斷條件(Condition)，此內容必須設置在自閘道延伸出來的線段上，可以在Properties -> Main config -> Condition來進行設定，以${判斷式}的表示式填入，如下圖：
![image-center]({{ '/images/20200309/20190623175717.PNG'| absolute_url }}){: .align-center}

   以XML格式呈現的樣貌如下：
```xml
<sequenceFlow id="flow6" name="金額大於10萬" sourceRef="e_gateway_supervisor" targetRef="managerReview">  
  <conditionExpression xsi:type="tFormalExpression"><![CDATA[${decision == "approve" && amount > 100000}]]></conditionExpression>  
</sequenceFlow>
```

## 定義監聽器(Listener)
假設本範例多了一項需求：「需要計算案件被退回到申請人的次數」，要達到這個需求當然有很多方式，但如果要用Activiti來處理，那首先想到的，就是要在流程中多定義一項流程變數來記錄這個資訊，不過，這個變數資訊並不需要由表單屬性(Form Properties)來傳達給Activiti，而是由Activiti自行計算，那要怎麼把這樣的邏輯設定到流程圖中呢? 這就需要定義監聽器(Listener)了。   

如果要定義並撰寫Listener，就必需要新增一些java類別到我們的專案中，而這些類別必須去實作Activiti API的介面，這些介面包括了ExecutionListener與 TaskListener，至於要選哪一項來實作，則要看Listener是要掛在什麼地方。
備註：如果在實際佈署Activiti服務的時候，會需要把這些Listener建置成.jar再放到classpath中。

在這邊我們先新增一支ExampleStartEventListener，它實作了ExecutionListener這個介面，在notify(DelegateExecution execution)的方法中我們可以去寫我們要的邏輯，請見下列程式碼：
```java
package org.activiti.example.bpmn.listener;

import org.activiti.engine.delegate.DelegateExecution;
import org.activiti.engine.delegate.ExecutionListener;

@SuppressWarnings("serial")
public class ExampleStartEventListener implements ExecutionListener {
	
	@Override
	public void notify(DelegateExecution execution) throws Exception {
		execution.setVariable("rejectCnt", 0);//設定流程變數rejectCnt，值為0
	}
}
```
上面這支Listener的邏輯很簡單，我們透過方法傳入之DelegateExecution物件，利用其提供之API來設定流程變數rejectCnt到流程實體中，並且值設為0。至於Execution是什麼東西呢？可以把它想成是流程執行的線路，一個流程實體可能對應了一個或多個Execution(端看有沒有開分支)，因此對於一個流程中的各個節點來說Execution也可以視為執行的環境，透過它的API，我們可以對流程相關的資訊做操作，詳細的定義可以參閱Activiti的官方文件內容。

新增完Listener的程式碼以後，我們可以在Plugin的圖形介面中，點擊我們要掛載Listener的節點，然後Properties -> Listeners -> New 去新增這支Listener的class以進行掛載，下圖就是把ExampleStartEventListener加到Start Event上：
![image-center]({{ '/images/20200309/20190624140321.PNG'| absolute_url }}){: .align-center}

依照同樣的做法，再新增另外一支Listener，這次是要掛載到User Task「使用者調整」上，程式碼如下：
```java
package org.activiti.example.bpmn.listener;

import org.activiti.engine.delegate.DelegateTask;
import org.activiti.engine.delegate.TaskListener;

@SuppressWarnings("serial")
public class ExampleInitiatorReviewListener implements TaskListener {	

	@Override
	public void notify(DelegateTask delegateTask) {
		Integer rejectCnt = (Integer)delegateTask.getVariable("rejectCnt");
		if (null != rejectCnt) {
			rejectCnt++;
			delegateTask.setVariable("rejectCnt", rejectCnt);
		}
	}
}
```

和前一個Listener不同的是，因為它是要掛載在Task的節點上，所以選擇實作TaskListener此介面，但其他作法則幾乎完全一樣，程式碼的內容也很簡單，就是把rejectCnt的變數值先取出後+1，然後再存回去。而掛載的方式與前一個Listener差不多，請見下圖：
![image-center]({{ '/images/20200309/20190624140628.PNG'| absolute_url }}){: .align-center}

上圖中請注意，Event欄位的值要設定為Create，這樣就是在進入這個User Task時會觸發Listener，如果是要在User Task完成時才觸發，Event的值就要設為End。

## 單元測試
依據前面的步驟，我們應該能建立一支符合需求且能運作的流程圖(.bpmn)了，接下來就是要測測看它的功能正不正常，包括把它餵給Activiti引擎，以及進行起案、簽核與結案等作業，為了要方便測試這些功能，因此我們需要把單元測試的環境配置好，這樣我們就能輕鬆的撰寫我們需要的單元測試。

這邊提供一個簡易的方法，其實我們安裝的Activiti BPMN Designer已經有整合IDE來自動生成單元測試了，我們只要在專案目錄下，找到流程圖檔，對它點擊右鍵->Activiti -> Generate unit test，測試程式碼就會被產生出來了，如下圖：
![image-center]({{ '/images/20200309/20190625105430.png'| absolute_url }}){: .align-center}

而自動生成的單元測試程式碼，主要內容如下(微調過檔案路徑)：
```java
public class ProcessTestExampleProcess {

	private String filename = "src\\main\\resources\\diagrams\\ExampleProcess.bpmn";

	@Rule
	public ActivitiRule activitiRule = new ActivitiRule();

	@Test
	public void startProcess() throws Exception {
		RepositoryService repositoryService = activitiRule.getRepositoryService();
		repositoryService.createDeployment().addInputStream("ExampleProcess.bpmn20.xml",
				new FileInputStream(filename)).deploy();
		RuntimeService runtimeService = activitiRule.getRuntimeService();
		Map<String, Object> variableMap = new HashMap<String, Object>();
		variableMap.put("name", "Activiti");
		ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("ExampleProcess", variableMap);
		assertNotNull(processInstance.getId());
		System.out.println("id " + processInstance.getId() + " "
				+ processInstance.getProcessDefinitionId());
	}
}
```

如上，一支單元測試就建好了，它裡面有使用到二項Activiti的核心API：RepositoryService與RuntimeService，RepositoryService在這裡負責讀取流程圖檔來進行流程的佈署，而RuntimeService負責了起案以產生流程實體，到這邊可能有人會有ㄧ個疑問，前面不是有說過Activiti需要有資料庫，那為什麼我們都不需要定義連線資訊呢? 這是因為Activiti本身有使用H2 in-memory database，因此執行單元測試時不強制需要外部資料庫。接下來我們來逐步檢視並調整這支程式，以達到完整的功能邏輯測試。

取得需要的API服務
: 對於Activiti引擎的操作，大附份就是要透過它的核心API，因此我們可以先自ActivitiRule物件來取得，如下程式碼：
```java
RepositoryService repositoryService = activitiRule.getRepositoryService(); //佈署流程圖
RuntimeService runtimeService = activitiRule.getRuntimeService(); //起案
TaskService taskService = activitiRule.getTaskService(); //待辦關卡查詢
FormService formService = activitiRule.getFormService(); //待辦關卡送出表單簽核
HistoryService historyService = activitiRule.getHistoryService(); //流程完成後之歷史資料查詢
```

或者我們也可以自ProcessEngine取得，如下程式碼：
```java
ProcessEngine processEngine = ProcessEngineConfiguration.createProcessEngineConfigurationFromResourceDefault().buildProcessEngine();

RepositoryService repositoryService = processEngine.getRepositoryService(); //佈署流程圖
RuntimeService runtimeService = processEngine.getRuntimeService(); //起案
TaskService taskService = processEngine.getTaskService(); //待辦關卡查詢
FormService formService = processEngine.getFormService(); //待辦關卡送出表單簽核
HistoryService historyService = processEngine.getHistoryService(); //流程完成後之歷史資料查詢
```

如果我們點入ActivityRule看它的原始碼，其實它也是在做ProcessEngine的建置，然後把相關服務物件初始化，有興趣可以看看ActivityRule怎麼處理的。

佈署流程圖(.bpmn)
: 如下程式碼，利用RepositoryService讀取流程圖後，再儲存到資料庫中，已完成佈署的動作。附帶一提，佈署進去的檔案資源名稱我們這邊是可以再定義的，這邊的副檔名是改用.bpmn20.xml(符合bpmn 2.0規範之xml檔)，用.bpmn20.xml或.bpmn以外的副檔名可能會無法正常運作。

```java
//佈署流程圖(.bpmn)
String filename = "src\\main\\resources\\diagrams\\ExampleProcess.bpmn";
StringBuilder flowHistory = new StringBuilder();
repositoryService.createDeployment().addInputStream("ExampleProcess.bpmn20.xml", new FileInputStream(filename)).deploy();
```

產生流程實體(ProcessInstance)
: 流程圖佈署完成後，即可用流程圖id進行起案，產生流程實體。記得我們在流程圖的Start Event有在表單(Form Properties)定義一些流程變數嗎?
這邊可以用一個Map集合來把這些資訊包起來，當成Payload，一同透過RuntimeService來進行起案，在成功起案後，可以拿到一個流程實體的id，之後就可以透過此id對流程進行操作，詳細請見以下程式碼：

```java
//起案時需要提供的表單屬性(流程變數)
Map<String, Object> variableMap = new HashMap<String, Object>();
variableMap.put("initiator", "Tang");
variableMap.put("supervisor", "Bryan");
variableMap.put("manager", "King");
variableMap.put("amount", "150000");
//起案產生流程實體
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("ExampleProcess", variableMap);
assertNotNull(processInstance.getId());
String processInstId = processInstance.getId();
System.out.println(String.format("起案產生流程實體(id=%s)", processInstId));
```

關卡簽核
: 有了流程實體以後，我們先用TaskService取得目前的待辦關卡為何，如下，拿到Task物件，目前的待辦關卡為直屬主管審核：

```java
// 查詢目前所在關卡(直屬主管審核)
Task task = taskService.createTaskQuery().processInstanceId(processInstId).singleResult();
System.out.println(task.getName() + "/" + task.getAssignee()); //印出：直屬主管審核/Bryan
```

接下來，我們可以將關卡簽核的邏輯獨立出來，把相關資訊參數化，宣告一個關卡簽核的方法：

```java
/**
 * 關卡簽核
 * @param task 待簽核關卡
 * @param properties 表單屬性資訊
 * @return 新的待辦關卡
 */
private Task completeTask(Task task, Map<String, String> properties) {
	FormService formService = activitiRule.getFormService();
	TaskService taskService = activitiRule.getTaskService();
	formService.submitTaskFormData(task.getId(), properties);
	return taskService.createTaskQuery().processInstanceId(task.getProcessInstanceId()).singleResult();
}
```

最後假設直屬主管決定審核通過，我們就設定表單屬性decision=approve來完成簽核，如下程式碼：

```java
Map<String, String> properties = new HashMap<String, String>(); //關卡要送出的表單屬性資訊
properties.put("decision", "approve"); //審核結果：核准		
task = completeTask(task, properties); //送簽(直屬主管審核)，取得下一關(經理審核)
System.out.println(task.getName() + "/" + task.getAssignee()); //印出：經理審核/King
flowHistory.append(" -> " + task.getName());
```

現在到了經理審核關卡，假設他決定退回給申請人，因此設定表單屬性decision=reject來完成簽核，如下程式碼：

```java
// 目前關卡：經理審核
properties = new HashMap<String, String>(); //關卡要送出的表單屬性資訊
properties.put("decision", "reject"); //審核結果：退回
task = completeTask(task, properties); //送簽(經理審核)，取得下一關(申請人調整)
System.out.println(task.getName() + "/" + task.getAssignee()); //印出：申請人調整/Tang
```

退回到申請人調整的關卡後，假設申請人調整了金額重送，如下程式碼設定表單屬性：

```java
// 目前關卡：申請人調整
properties = new HashMap<String, String>(); //關卡要送出的表單屬性資訊
properties.put("decision", "approve"); //審核結果：核准
properties.put("amount", "50000"); // 調整金額<100000
System.out.println(task.getName() + "/" + task.getAssignee()); //直屬主管審核/Bryan
task = completeTask(task, properties); //送簽(申請人調整)，取得下一關(直屬主管審核)
```

現在又重新回到了直屬主管審核關卡，由於目前金額調整後少於100000元，因此這關簽核結束以後，就應該沒有下一關了，請見以下程式碼：

```java
// 直屬主管審核 (第2次)
properties = new HashMap<String, String>(); //關卡要送出的表單屬性資訊
properties.put("decision", "approve"); //審核結果：核准
task = completeTask(task, properties); //送簽(直屬主管審核)，流程結束沒有下一關
assertNull(task); //沒有下一關
```

查詢歷史資料
: 流程結束以後，該流程實體的相關資訊可用HistoryService來查詢，如下程式碼：

```java
// 查詢已完成流程實體id
List<HistoricProcessInstance> finishedProcessInstList = historyService.createHistoricProcessInstanceQuery().finished().list();
assertNotNull(finishedProcessInstList);
assertEquals(processInstId, finishedProcessInstList.get(0).getId());

// 找出流程變數rejectCnt的值
Object finalRejectCnt = historyService.createHistoricVariableInstanceQuery().processInstanceId(processInstId).variableName("rejectCnt").singleResult().getValue();
System.out.println(String.format("流程實體已結案(id=%s)，案件被退回次數為%d次", finishedProcessInstList.get(0).getId(), (Integer) finalRejectCnt));
```

## 範例實作
歡迎到此[github連結](https://github.com/scryed0204/ActivitiExample/ "Activiti Example")取得範例內容，提供參考。