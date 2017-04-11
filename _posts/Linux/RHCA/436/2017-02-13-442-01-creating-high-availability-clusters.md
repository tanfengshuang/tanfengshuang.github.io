---
layout: post
title:  "建立 High-Availability 叢集(436-1)"
categories: Linux
tags: RHCA 436
---

### High-Availability 叢集

###### 什麼是叢集？

叢集是共同進行單一工作的一組電腦。至於執行何種工作及如何執行工作，則視叢集而有所不同。本課程說明兩種不同類型的叢集。

    High-Availability 叢集：High-Availability 叢集 (亦稱為 HA 叢集或容錯移轉叢集) 的目的是盡可能維持執行中服務的可用性。主要方法是讓 High-Availability 叢集的節點互相監控是否發生故障，然後在服務或節點故障時，將服務移轉至仍視為「狀況良好」的節點。High-Availability 叢集可再細分為兩種：

        主動-主動 High-Availability 叢集：服務會在多個節點上執行，縮短容錯移轉的時間。

        主動-被動 High-Availability 叢集：服務每次僅在單一節點上執行。

    High-Availability 叢集經常用於支援企業的任務關鍵服務。舉例來說，Pacemaker 與 Red Hat High Availability 外掛程式都是實作 High-Availability 叢集的軟體。

    儲存叢集：在儲存叢集中，所有成員皆會提供單一叢集檔案系統，可供各種伺服器系統存取。提供的檔案系統可能同時用於讀取與寫入資料。這在提供高可用性應用程式資料 (如 Web 伺服器內容) 時非常有用，無需相同資料的多個備援複本。Red Hat Resilient Storage 外掛程式提供的 GFS2 即為叢集檔案系統的一個實例。 
    

###### High-Availability 叢集的目的為何？

High-Availability 叢集的主要目的是，透過排除瓶頸與單一失敗點，盡可能維持服務的可用性。這與維持單一機器運作時間越高越好的策略不同。對於服務取用者而言，執行服務之伺服器的運作時間長短並不重要，重要的是服務可用性。High-Availability 叢集運用各種概念與技術，考慮到服務的完整性與可用性：

*    資源與資源群組: 在叢集術語中，基本工作單位稱為資源。單一 IP 位址、檔案系統或資料庫均可視為資源。通常會在這些資源之間定義關係，以建立用戶對應的服務。定義這些關係最常見的方法之一，是將一組資源合併為一個群組。如此會指定群組中的所有資源必須在同一節點上共同執行，並且建立固定 (線性) 的啟動與停止順序。 例如，叢集若要能提供 Web 伺服器服務，則 Web 伺服器 daemon、伺服器要共用的資料，以及 daemon 要接聽的 IP 位址，都必須在同一叢集節點上提供。
*    容錯移轉: 當 High-Aavailability 叢集注意到原來執行服務的節點沒有回應時，叢集會嘗試將服務移轉到其他節點，以維持服務的可用性，這稱為容錯移轉。
*    隔離: 隔離為確保發生問題的叢集節點不會造成損毀，以便其資源可以安全復原於叢集其他位置的機制。因為我們無法假設不能連線的節點確實關閉，因此這是必要措施。因為無作用的節點顯然無法做任何動作，隔離經常透過關閉節點電源的方式完成。在其他情況下，組合操作會用於切斷節點的網路連線 (以阻止新工作的到來) 或儲存裝置連線 (以阻止節點寫入共用式儲存裝置)。
*    共用式儲存裝置: 大部分 High-Availability 叢集還需要某種形式的共用式儲存裝置，亦即可從多個節點存取的儲存裝置。共用式儲存裝置可為叢集的多個節點提供相同的應用程式資料。 叢集上執行的應用程式可能循序或同時存取資料。High-Availability 叢集必須確保共用式儲存裝置的資料完整性。隔離可保證資料完整性。
*    仲裁: 仲裁描述了一種維持叢集完整性的投票系統。每一個叢集成員都有指定的投票數，預設為一票。視叢集組態而定，當存在半數或以上的投票數時，叢集即獲得仲裁。無法與其他叢集成員通訊，以及無法傳送投票的叢集成員，會遭到大部分正常運作的叢集成員隔離。叢集一般需要仲裁才能運作。若叢集失去仲裁或無法建立仲裁，依預設，不會啟動任何資源或資源群組，並會停止執行中的資源群組，以確保資料完整性。 

###### HA 叢集的使用時機

規劃 High-Availability 叢集時，必須先回答一個重要問題：將服務置於 HA 叢集會提高服務可用性嗎？

要回答這個問題，必須瞭解服務的功能，以及可如何設定服務的用戶端：

*    視解決方案而定，像是 DNS 與 LDAP 等內建容錯移轉或負載平衡的服務，無法因置於 HA 叢集而獲益。例如，DNS 或 LDAP 服務可透過主要/從屬關係或多主機關係使用多部伺服器。這些服務可設定為在主要伺服器與從屬伺服器之間進行資料複寫。DNS 與 LDAP 的用戶端可使用多部伺服器。在主要/從屬或多主機組態中，與容錯移轉延遲的關聯性不大，因此將這些服務置於 HA 叢集並不會提高服務可用性。不過，對於 Openstack 平台解決方案而言，將 RabbitMQ 與 Galera 等資源置於 HA 叢集可能有所助益。與 Red Hat Openstack 平台 High Availability 的相關詳細資訊已超出本課程範圍。
*    未內建容錯移轉或負載平衡的服務可從 High-Availability 叢集組態獲益。例如 NFS 與 Samba 等服務。

並非每一種可用性問題皆可透過 High-Availability 叢集的方式解決。 一般而言，牽涉到應用程式損毀或網路失敗的問題無法透過 High-Availability 叢集的方式解決：

*    若讀取特定輸入時的錯誤造成應用程式損毀，則即使位於 High-Availability 叢集中一樣會損毀。 在此情況下，叢集會將服務容錯移轉至不同節點，但若再次讀取相同輸入，應用程式會再度損毀。
*    High-Availability 叢集也不提供端對端備援。若基礎結構中的網路錯誤造成無法連線至叢集，儘管叢集本身可完全運作，即使服務在 High-Availability 叢集上執行，用戶端仍無法與服務連線。因此，必須考慮叢集的架構，才能在部署設計上避免單一失敗點。這牽涉到一些取捨，叢集的結構設計師必須考量到叢集的各個元件可承受的風險等級。


### 架構概觀

###### 硬件

Hardware 	Purpose

*    Cluster Nodes 	    這些是執行叢集軟體與服務的機器。
*    Public Network 	此網路用於用戶端與叢集上執行之服務之間的通訊。 服務通常具備浮動 IP 位址，亦即 IP 位址會指派給目前執行對應服務的任一個節點。
*    Private Network 	此網路專門用於叢集通訊，以及重要叢集硬體的通訊，例如連網電源開關。
*    Networked Power Switch 	連網電源開關可用於遠端控制叢集節點的電源。 這是實作電源隔離的方法之一，本課程稍後會詳加說明。ILO 或 DRAC 等遠端管理卡也可以用於此用途。
*    Fibre Channel Switch 	    範例中所有節點皆連接至相同的共同式儲存裝置。 雖然此用途普遍使用光纖通道，也可使用獨立的乙太網路，搭配 iSCSI 或 FCoE。 

###### 软件

Component 	Purpose

1. corosync： 這是 Pacemaker 處理叢集節點間通訊所用的架構。corosync 也是 Pacemaker 之成員資格與仲裁資料的來源。
2. pcs： pcs RPM 套件包含兩個叢集組態工具：

*    pcs 命令提供了建立、設定及控制 Pacemaker/Corosync 叢集所有層面的命令列介面。
*    pcsd 服務提供了叢集組態同步處理，以及用於建立與設定 Pacemaker/Corosync 叢集的 Web 前端。 

3. pacemaker： 這是負責處理所有叢集相關活動的元件，例如監控叢集成員資格、管理服務與資源，以及隔離叢集成員。pacemaker RPM 套件包含三個重要設施：

*    叢集資訊庫 (CIB)：叢集資訊庫包含叢集與叢集資源的組態和狀態資訊，格式為 XML。Pacemaker 會選擇叢集中的某個叢集節點作為指定協調員 (DC)，然後將要同步處理的叢集和資源狀態及叢集組態，儲存到其他所有的主動叢集節點。
*    原則引擎 (PEngine)：原則引擎會運用叢集資訊庫 (CIB) 的內容，計算叢集最理想的狀態，並計算如何達到該狀態。
*    叢集資源管理 Daemon (CRMd)：叢集資源管理 Daemon 會協調資源啟動、停止及狀態查詢動作，並傳送至每一個叢集節點上執行的本機資源管理 Daemon (LRMd)。LRMd 會將從 CRMd 接收到的動作傳遞給資源代理程式。
*    STONITH (Shoot the Other Node in the Head)：STONITH 設施負責處理隔離要求，並將要求的動作轉送至 CIB 中設定的隔離裝置。 


