# 實作 Azure DDoS Network Protection 模擬測試<br>
分散式阻斷服務 (DDoS) 攻擊是將應用程式移至雲端的客戶所面臨的最大可用性和安全性顧慮之一，規劃和準備 DDoS 攻擊對於瞭解實際攻擊期間應用程式的可用性和回應至關重要。Azure DDoS 主要緩和來自 Layer 3、4 的網路攻擊流量，
 但是還是會有來自 Layer 7 的攻擊，鎖定 Web 應用程式封包，我們可以使用 Azure 應用程式閘道來提供這些攻擊的防禦，本篇佈署的架構由 Web 應用程式、Azure 應用程式閘道組成的架構來讓大家練習。<br><br>
微軟與 BreakingPoint Cloud 合作，讓開發商、開發者可以透過 BreakingPoint Cloud 驗證 Azure DDoS 網路保護如何保護 Azure 資源，不過一個 Trial 帳號只有 5GB 的免費流量，如果長期測試還是需要購買付費方案。<br>
 您大約需要花費 30 分鐘完成此 Lab，透過手把手教學您將學會：<br>
 - 註冊、使用 BreakingPoint Cloud 驗證 Azure DDoS Network Protection<br>
 - 學會如何啟用 Azure DDoS Network Protection 保護 PaaS Web 應用程式<br>
 - 學會如何通過 Azure 監視器進行遙測、查看攻擊期間詳細資料、設定警示<br>
 - 學會如何查看攻擊後的完整摘要<br>

## 環境準備 <br>
 - Azure 訂用帳戶、Azure 訂用帳戶擁有者權限<br>
 - 準備 BreakingPoint Cloud 環境<br>
	- 註冊 BreakingPoint Cloud https://breakingpoint.cloud/login ，點選 Sign Up，輸入個人基本資料，完成後您就可以獲取到一個擁有 5GB 頻寬量的測試帳號。<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/signup.PNG "signup")<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/signup2.PNG "signup2")<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/signup3.PNG "signup3")<br>
	- 點選上方導覽列齒輪，選擇 [Azure Subscription] 輸入您的 Azure Subscription ID，並驗證您的身分，確保此服務不用於惡意攻擊。<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/signup4.PNG "signup4")<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/signup5.PNG "signup5")<br>
 - 使用 CloudShell 快速部署 Lab 環境
	- 架構圖示
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/lab-architecture.PNG "lab-architecture")<br>
	- 啟用 CloudShell<br>
    - 輸入`Connect-AzAccount` 登入<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/loginCloudShell.PNG "loginCloudShell")<br>
	- 上傳 AppServiceAndAppGW.ps1，此檔案會建立資源群組、虛擬網路、應用程式服務、App Service 方案(S1)、應用程式閘道(WAF_v2)、公用 IP、網路安全性群組<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/uploadps.PNG "uploadps")<br>
	- 輸入並執行 `./AppServiceAndAppGW.ps1` <br>
	- 完成後，您將可以在資源群組 AppGW，找到以下服務資源<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/services-list.PNG "services-list")<br>
	- 將後端集區目標指向應用程式服務<br>
		- 選擇剛建立好的應用程式閘道 AppGW，點選設定類別下方的「後端集區」<br>
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/backendpool.PNG "backendpool")<br>
		- 目標類型選擇「應用程式服務」，目標選擇剛建立好的應用程式服務<br>
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/backendpool2.PNG "backendpool2")<br>
	- 確認流量能夠由應用程式閘道前端公用 IP 位置進入
		- 選擇應用程式閘道，在概觀中複製前端公用 IP 位置
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/appgwinfo.PNG "appgwinfo")<br>
		- 直接將前端公用 IP 位置貼上瀏覽器執行
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/browsercheck.PNG "browsercheck")<br>

## 啟用 Azure DDoS Network Protection 保護 PaaS Web 應用程式
 - 您如果需要啟用虛擬網路中的 DDoS Standard，您必須先建立 DDoS protection plans<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/createddosplan.PNG "createddosplan")<br>
 - 點選虛擬網路 myVNet，在設定的功能列表中點選「DDoS 保護」，DDoS 保護標準點選「啟用」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/enableddosstd.PNG "enableddosstd")<br>
 - 建立 Log Analytics 工作區，資源群組輸入 AppGW，名稱請自行定義，區域請選擇日本東部，完成後請按評論與建立<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/createloganalytic.PNG "createloganalytic")<br>
 - 在 Portal 上方搜尋監視，並點選此服務，在服務項目選單中選擇「診斷設定」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric4.PNG "metric4")<br>
 - 單獨選擇公用 IP 位置「myAGPublicIPAddress」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric6.png "metric6")<br>
 - 選擇「新增診斷設定」，勾選「DDoSMitigationFlowlogs」、「DDoSMitigationReports」、「傳送至 Log Analytics」，選擇您剛剛所建立的 Log Analytics 工作區<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/diag.PNG "diag")<br>
 - 之後您驗證 DDoS 後，就可以從 Log Analytics 查詢到您的紀錄 <br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/DDoSMitigationFlowLogs5.PNG "DDoSMitigationFlowLogs5")<br>
## 設定 DDoS 保護計量的警示
 - 在 Portal 上方搜尋監視，並點選此服務<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric4.PNG "metric4")<br>
 - 在服務項目選單中選擇「計量」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric5.PNG "metric5")<br>
 - 單獨選擇公用 IP 位置「myAGPublicIPAddress」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metricsalert1.PNG "metricsalert1")<br>
 - 在度量欄位的下拉式選單選擇 Under DDoS attack or not，值為 1 時，代表正在遭受攻擊，完成後請點選上方「新增警示規則」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metricsalert2.PNG "metricsalert2")<br>
 - 將條件設定為大於等於閾值 1 時觸發，完成後儲存<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metricsalert3.PNG "metricsalert3")<br>
 - 新增動作群組，在這個步驟中，您可以選擇觸發條件後要使用何種方式通知，在這之前您必須先建立動作群組<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metricsalert4.PNG "metricsalert4")<br>
 - 本範例選擇 Email 方式通知<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metricsalert5.PNG "metricsalert5")<br>
 - 最後請將您的警示規則命名、給予適當描述，並將嚴重性選擇 Sev 0，嚴重程度由大到小排序為 0- 5<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metricsalert6.PNG "metricsalert6")<br>
 - 在下個小節中您將會使用 BreakingPoint Cloud 驗證 DDoS 保護，您可以您指定的信箱中發現通知信件，以及在 Azure Portal 中的警示計數與詳細資訊<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/alert1.png "alert1")<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/alert2.png "alert2")<br>
 - 警示紀錄觸發統計<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/alert3.PNG "alert3")<br>

## 驗證 DDoS 偵測
 - 登入您剛註冊的帳號 BreakingPoint Cloud https://breakingpoint.cloud/login ，並依序填入與選擇<br>
	- Target IP Address : 請填入應用程式閘道 AppGW 的公用 IP 位置<br>
	- Port Number : 請填入 80 <br>
	- DDoS Profile : 請在下拉式選單中選擇您要的攻擊型式，此範例選擇 TCP SYN Flood<br>
	- Test Size : 請選擇 100K pps, 50 Mbps and 4 source IPs，因為您得到的 Trial 流量僅只有 5 GB<br>
	- Test Duration : 選擇此攻擊驗證持續 10 分鐘<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/DDoSTest1.PNG "DDoSTest1")<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/DDoSTest2.PNG "DDoSTest2")<br>

## 使用 Azure Monitor 查看計量
 - 在 Portal 上方搜尋監視，並點選此服務<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric4.PNG "metric4")<br>
 - 在服務項目選單中選擇「計量」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric5.PNG "metric5")<br>
 - 單獨選擇公用 IP 位置「myAGPublicIPAddress」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric6.png "metric6")<br>
 - 確認服務是否遭受到攻擊 Under DDoS attack or not 值為 1 時，代表正在遭受攻擊<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric9.PNG "metric9")<br>
 - 我們可以監控幾個度量值，來觀察在攻擊過程中近似即時的封包變化<br>
	- Inbound Packets Dropped DDoS : DDoS 保護系統在攻擊期間丟棄的封包<br>
	- Inbound Packets Forwarded DDoS : DDoS 保護系統在攻擊期間轉發的封包，代表未被過濾的流量<br>
	- Inbound Packets DDoS : 進入 DDoS 保護系統總封包數量，上面兩個度量封包總和<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric8.PNG "metric8")<br>
 - 在這三個度量中，可以知道目前通過 Azure 機器學習後自動設置的閾值指標，只有當達到閾值時，才會觸發 DDoS 保護系統的緩和機制<br>
	- Inbound TCP packets to trigger DDoS mitigation
	- Inbound UDP packets to trigger DDoS mitigation
	- Inbound SYN packets to trigger DDoS mitigation
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/metric7.PNG "metric7")<br>

## 攻擊後查看相關紀錄(待完成)
 - 建立 Azure Analytics Dashboard
	- 下載 Azure analytics dashboard sample https://github.com/Anupamvi/Azure-DDoS-Protection/raw/master/flowlogsbyip.zip <br>
	- 到您的 Log analytics 檢視表設計工具，上傳 FlowlogsbyIP.omsview <br>
	 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/report1.png "report1")<br>
	- Azure analytics dashboard
		- 左邊的圖表顯示您的那些公用 IP 位置正在遭遇到 DDoS 攻擊，中間的圖表顯示 DDoS 攻擊來源與流量，右邊的圖表顯示 DDoS 保護系統在攻擊期間總量、丟棄、轉發的流量<br>
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/DDoSMitigationFlowLogs3.PNG "DDoSMitigationFlowLogs3")<br>
		- 左邊的圖表顯示丟棄封包的原因計數，中間的圖表顯示攻擊期間是通過哪些 Port 的攻擊流量，右邊的圖表顯示所有丟棄的封包流量統計<br>
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/DDoSMitigationFlowLogs4.PNG "DDoSMitigationFlowLogs4")<br>

 - DDoSMitigationReports 欄位指標<br>
	- Attack vectors 攻擊媒介<br>
	- Traffic statistics 流量統計資料<br>
	- Reason for dropped packets 丟棄封包的原因<br>
	- Protocols involved 通訊協定 <br>
	- Top 10 source countries or regions 前 10 個來源國家/地區或區域<br>
	- Top 10 source ASNs 前 10 個來源的 ASN<br>

 - DDoSMitigationReports 欄位指標 (只有在公用 IP 位址的虛擬網路上啟用了 DDoS 保護標準 時，攻擊分析才會生效)<br>
	- Source IP 來源 IP<br>
	- Destination IP 目的地 IP<br>
	- Source Port 來源連接埠<br>
	- Destination port 目的地連接埠<br>
	- Protocol type 通訊協定類型<br>
	- Action taken during mitigation 在風險降低期間所採取的動作<br>


**參考來源與更詳細的說明**<br>
https://docs.microsoft.com/zh-tw/azure/security/fundamentals/ddos-best-practices <br>
https://docs.microsoft.com/zh-tw/azure/virtual-network/ddos-protection-overview <br>
https://docs.microsoft.com/zh-tw/azure/virtual-network/manage-ddos-protection <br>
https://azure.microsoft.com/zh-tw/blog/ddos-protection-attack-analytics-rapid-response <br>
