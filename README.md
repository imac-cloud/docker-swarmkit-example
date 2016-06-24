# Learning how to use Docker SwarmKit

*SwarmKit* 是一個策劃任何分散式系統的工具包，包含節點發現、 raft-based consensus 與任務調度...等功能。

其主要行為可分成：

- **分散式** : *SwarmKit* 為了協調與不依賴未能進行決策的單一節點，使用 raft consensus 演算法。
- **安全** : 在 *Swarm* 內節點間的通訊與成員都是安全的，*SwarmKit* 使用相互的 TLS 來達到節點驗證、角色授權與傳輸加密，自動化憑證發佈與輪轉。
- **簡易** : *SwarmKit* 操作上簡易且最小的基礎設施依賴，在操作上也不必額外的資料庫。

## 概觀

運行 *SwarmKit* 可群組所有機器形成一個 *Swarm* ，且彼此間可任意調度任務。一旦有一台機器加入，則它將成為 *Swarm* 的一員。節點可設為工作節點（*worker* node）或管理節點 (*manager* node)。

- **工作節點** 主要是透過執行器 (*Executor*) 執行任務 (Tasks)，*SwarmKit* 的預設執行器是 *Docker Container Executor* ，但若需調整也可輕易變動。
- **管理節點** 主要負責使用者的需求與回應，將叢集狀態調整成如使用者預期。

後續也可動態修改節點角色，如工作節點提升為管理節點，或管理節點降級為工作節點。

服務由任務組成。服務定義了需要被建立的任務種類、如何運行（如：隨時執行多個副本）與如何更新（如：[滾動更新](https://zh.wikipedia.org/wiki/%E6%BB%9A%E5%8A%A8%E6%9B%B4%E6%96%B0)）。

## 特性

- **服務編排** 
	- **一致性** : *SwarmKit* 會不斷比對期望狀態與實際狀態，發現兩者不相符時（如：服務擴展、節點失效），則 *SwarmKit* 會自動將服務中的任務調度到其他節點。
	- **類型** : 目前 *SwarmKit* 支援兩種服務類型 : 
		- **複製型服務** : *SwarmKit* 會在該節點上啟動預期數量的副本。
		- **全局服務** : *SwarmKit* 會在叢集上的各個可用節點執行一個任務。
	- **配置升級** : 任何時候你都可以修改服務的一個或多個配置檔。當你更新完後，*SwarmKit* 會協調期望狀態，確保所有的任務皆已使用期望的設定。預設情況下會同時執行升級。你可以使用不同的方式進行配置 : 
		- **對比** : 定義同一時間可以執行多少的升級項目。
		- **延遲** : 設定最小更新延遲間隔時間。*SwarmKit* 等待上一個任務關閉後啟動，接著等待狀態為 RUNNING ，再等待額外配置的等待時間，最後，執行後續其他任務。
	- **重啟策略** : 使用者可自行定義重啟條件、延遲時間、限制（給定時間內最大的嘗試次數）。*SwarmKit* 可以決定在不同機器上重新啟動任務。換句話說，不合適的節點將被排除，不接受任何任務。
- **調度**
	- **資源感知** : *SwarmKit* 能察覺節點上的可用支援作為後續分配任務的依據。
	- **限制** : 使用者可定義限制表達式，限制任務所被調度的節點。限制可已對應多種節點的屬性，像是 IDs、名稱、標籤（如：`node.labels.foo!=bar1`）。
	- **策略** : 目前專案實現的調度策略為調度任務給負載最低的節點，提供合適的限制與資源需求。

##快速建置

| 節點名稱   | IP Address |
|:---------:|:----------:|
| manager-1 | 10.26.1.77 |
| worker-1  | 10.26.1.79 |


環境要求 : 
  
  - 已安裝 Docker

Manager Node 安裝 : 
  
```
$ ./manager-node.sh 

```

>  執行 install 內的 manager-node.sh 且輸入 Manager Node 名稱進行快速安裝。

Worker Node 安裝 :

```
$ ./worker-node.sh 

```

>  執行 install 內的 worker-node.sh 且輸入 Worker Node 名稱與 Manager Node IP Address 進行快速安裝。

若成功安裝後，你可在 Manager Node 上執行查詢所有節點指令：

```
$ swarmctl node ls

ID                         Name       Membership  Status  Availability  Manager Status
--                         ----       ----------  ------  ------------  --------------
9aqyyb00e252y2rr39kldhh06  manager-1  ACCEPTED    READY   ACTIVE        REACHABLE *
brm3eaxfd2iz9aeqk02ecvihf  worker-1   ACCEPTED    READY   ACTIVE
```
