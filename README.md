# 實作 Azure DDoS Protection Standard 壓力測試<br>
分散式阻斷服務 (DDoS) 攻擊是將應用程式移至雲端的客戶所面臨的最大可用性和安全性顧慮之一，規劃和準備 DDoS 攻擊對於瞭解實際攻擊期間應用程式的可用性和回應至關重要。Azure DDoS 主要緩和來自 Layer 3、4 的網路攻擊流量，
 但是還是會有來自 Layer 7 的攻擊，鎖定 Web 應用程式封包，我們可以使用 Azure 應用程式閘道來提供這些攻擊的防禦，本篇佈署的架構由 Web 應用程式、Azure 應用程式閘道組成的架構來讓大家練習。<br><br>
微軟與 BreakingPoint Cloud 合作，讓開發商、開發者可以透過 BreakingPoint Cloud 驗證 Microsoft Azure DDoS 保護如何保護 Azure 資源，不過一個 Trial 帳號只有 5GB 的免費流量，如果長期測試還是需要購買付費方案。<br>
 您大約需要花費 ?? 分鐘完成此 Lab，透過手把手教學您將學會：<br>
 - 註冊、使用 BreakingPoint Cloud 驗證 Microsoft Azure DDoS Standard<br>
 - 學會如何啟用 Azure DDoS Protection Standard 保護 PaaS Web 應用程式<br>
 - 學會如何通過 Azure 監視器進行遙測、查看攻擊期間詳細資料、設定警示<br>
 - 學會如何求助專業快速回應團隊支援<br>
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

## 啟用 Azure DDoS Protection Standard 保護 PaaS Web 應用程式
 - 您如果需要啟用虛擬網路中的 DDoS Standard，您必須先建立 DDoS protection plans<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/createddosplan.PNG "createddosplan")<br>
 - 點選虛擬網路 myVNet，在設定的功能列表中點選「DDoS 保護」，DDoS 保護標準點選「啟用」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/enableddosstd.PNG "enableddosstd")<br>
 - 建立 Log Analytics 工作區，資源群組輸入 AppGW，名稱請自行定義，區域請選擇日本東部，完成後請按評論與建立<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/createloganalytic.PNG "createloganalytic")<br>
 - 在監視的功能列表中點選「診斷紀錄」，在右方頁面點選「新增診斷紀錄」<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/diagsetting3.png "diagsetting3")<br>
 - 請勾選「AllMetrics」、「傳送至 Log Analytics」，選擇您的訂閱與剛剛建立的工作區<br>
 ![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/diagsetting4.png "diagsetting4")<br>

## 使用 DDoS 保護遙測
## 檢視 DDoS 風險降低原則
## 設定 DDoS 保護計量的警示
## 設定 DDoS 攻擊分析
## 設定 DDoS 攻擊風險降低報告
## 設定 DDoS 攻擊風險降低流程記錄
## 驗證 DDoS 偵測
**參考來源與更詳細的說明**
https://docs.microsoft.com/zh-tw/azure/security/fundamentals/ddos-best-practices
https://docs.microsoft.com/zh-tw/azure/virtual-network/ddos-protection-overview
