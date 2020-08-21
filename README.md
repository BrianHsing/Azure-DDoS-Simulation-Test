# Azure DDoS Protection Standard Stress Testing<br>
	分散式阻斷服務 (DDoS) 攻擊是將應用程式移至雲端的客戶所面臨的最大可用性和安全性顧慮之一，規劃和準備 DDoS 攻擊對於瞭解實際攻擊期間應用程式的可用性和回應至關重要。Azure DDoS 主要緩和來自 Layer 3、4 的網路攻擊流量，
 但是還是會有來自 Layer 7 的攻擊，鎖定 Web 應用程式封包，我們可以使用 Azure 應用程式閘道來提供這些攻擊的防禦，本篇佈署的架構由 Web 應用程式、Azure 應用程式閘道組成的架構來讓大家練習。<br>
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
	- 啟用 CloudShell<br>
    - 輸入`Connect-AzAccount` 登入<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/loginCloudShell.PNG "loginCloudShell")<br>
	- 上傳 AppServiceAndAppGW.ps1，此檔案會建立資源群組、虛擬網路、應用程式服務、App Service 方案(S1)、應用程式閘道(WAF_v2)、公用 IP、網路安全性群組<br>
	![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/uploadps.PNG "uploadps")<br>
	- 輸入並執行 `./AppServiceAndAppGW.ps1` <br>
	- 將後端集區目標指向應用程式服務<br>
		- 選擇剛建立好的應用程式閘道 AppGW，點選設定類別下方的「後端集區」<br>
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/backendpool.PNG "backendpool")<br>
		- 目標類型選擇「應用程式服務」，目標選擇剛建立好的應用程式服務<br>
		![GITHUB](https://github.com/BrianHsing/Azure-DDoS-Stress-Testing/blob/master/DDoSImage/backendpool2.PNG "backendpool2")<br>

**參考來源與更詳細的說明**
