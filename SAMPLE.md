# sitecore整合

## 批次運作機制
TODO  描述&圖
## 快取運作機制
TODO  描述&圖

## PC版系統設計
- cust1:velocity元件異動內容
新增category、code、area屬性，這三個屬性為SiteCore文案的鍵值，頁面中的各文案區塊所定義的文案鍵值各自對應一組文案內容。

- cust1:velocity元件使用方式
原先定義的i18n、escape、params、template等屬性都可以繼續使用，如果template、category、code、area等屬性同時有值時，優先以SiteCore鍵值取得文案內容，如果取不到文案資料或文案內容為空時，改以template取得文案檔案內容來顯示。

### 運作流程
TODO  圖
### 程式範例
- JSF的XHTML檔案
``` html
<cust1:velocity
	i18n="true"
	template="/remind/fcp/FCP_Fr_OverDeclareAmount.html"
	params="#{currBean.vaWebSetupUrl}"
	category="FCP"
	code="FCP03005"
	area="edit_1"/>
```

## M版系統設計
TODO 描述
### 運作流程
``` puml
browser->loginPage: welcome
loginPage->browser: page and json

browser->loginHome: login submit
loginHome->browser: page and json

browser->ib.js: click menu
ib.js->txn.js: load homeCtrl.jsp
ib.js->txn.js: load homeCtrl.js
Note left of ib.js: sitecore處理 \n jsp檔案中宣告data-bind-sitecore \n表示文案要崁入的位置

ib.js->homeCtrl.js: init()
activate homeCtrl.js

homeCtrl.js->funcController.java: send ajax
activate funcController.java
funcController.java->SiteCoreCacheMgr: getTemplaties(rsData, keyMap)
SiteCoreCacheMgr->funcController.java: sitecore templates
funcController.java->homeCtrl.js: rsData && sitecore templates
Note left of funcController.java: sitecore處理 \n取得快取的文案內容

homeCtrl.js-->txn.js: addAllSiteCoreTemplates(map)
Note left of homeCtrl.js: sitecore處理 \n 暫存文案在交易中，切換交易會清除
deactivate funcController.java

homeCtrl.js->ib.js: txn.bindonce()
activate ib.js
ib.js->ib.js: search data-bind-sitecore on homeCtrl.jsp
ib.js->txn.js: fetchRequiredTemplate(key)
txn.js-->ib.js: {templaties}
Note left of ib.js: sitecore處理 \nreplace data-bind-sitecore tag \nby $.replaceWith()
deactivate ib.js


homeCtrl.js --> ib.js: txn.showPage()
deactivate homeCtrl.js
```
## M版程式範例
- java端 - Controller中撰寫專屬文案取得的method取得所需文案
```java
/**
 * 定義要抓取文案的清單
 **/
function List<WcmDataBan> getWCMList() throws DatabaseException {
	//文案清單
	List<WcmDataBan> wcmDataBanList = new ArrayList<WcmDataBan>();
	WcmDataBan bean1 = new WcmDataBan("FFM", "FFM03001", "edit_reminder")
	WcmDataBan bean2 = new WcmDataBan("FFM", "FFM03001", "edit_1")
	wcmDataBanList.add(bean1);
	wcmDataBanList.add(bean2);

	//[如果沒有參數可以不用寫] 設定文案中參數的對應值
	Map<String,String> params = new HashMap<>();
	params.put("title","測試")
	params.put("public_url","http://test.url.com")
	bean2.setParam(params);

	//取得並回傳案內容
	return B2CUtils.getWcmDataCache().getMContent(wcmDataBanList);
}
```
