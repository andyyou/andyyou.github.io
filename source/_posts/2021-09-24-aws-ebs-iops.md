---
title: 您必須了解關於 AWS 的 IOPS
date: 2021-09-24 17:16:11
tags:
  - aws
categories: Cloud
---

# 

假如您持續深入了解 AWS ，您肯定會發現系統層面比您想像的還要複雜。EBS 儲存空間也不例外！

掛載儲存體到一個伺服器或主機看似相對簡單，然後您發現您的應用程式有點慢。不知道到底是什麼原因直到您看到 Stack Overflow 的討論關於 IOPS 和吞吐量（Throughput）的問題。在面對這個問題幾次之後，我決定要研究一下這個問題。

下面是我嘗試理解為什麼 EBS 變成效能瓶頸的原因。

> AWS 有 SSD 和傳統的磁盤硬碟，這裡我們說的是 SSD 的部分，如果您不是很確定您使用的，大概多數都是 SSD。

<!-- more -->

## IOPS

當提到 EBS 效能，IOPS 是您在 AWS 文件和部落格幾乎都會看到的東西。當然我們會提到其他比 IOPS 還重要的東西，不過讓我們從這裡開始。

### IOPS （Input / Output Operations per Second）每秒的輸入輸出操作

* 每一個操作是指硬碟的讀寫
* 每一個操作的讀寫資料量可能有所不同

那麼 IOPS 是怎麼計算的？

### 計算 IOPS

> 如果一個操作的資料大於 `256KB` 則該操作會被切割成多個操作

在 AWS，每一個操作的資料上限是 256 KB，超過 256KB 會被分成多個操作。

意思是如果您執行一個 1024KB 資料的操作，那就是 4 個 IOPS `1024/256 = 4`

> 另外，隨機讀寫小於 256KB 資料的操作可能造成多個操作

### IOPS 限制

硬碟的 IOPS 是有限制的，大量的操作會快速增加 IOPS 導致達到上限。

### 吞吐量（Throughput ）限制

> 您很可能在達到 IOPS 限制之前，就先踩到吞吐量限制

這就是讓 IOPS 變的更複雜的地方。非常有可能因為一次讀取大量的資料，造成根本還沒達到 IOPS 的限制，先遇到吞吐量限制。

舉例來說：如果您的操作資料為 256KB，硬碟吞吐量上限為 250MB，也就是硬碟只有 1000 IOPS。

因為 `1000 * 256KB = 256MB`，也就是說 1000 個 256KB IOPS 讀寫操作會踩到 250MB 吞吐量限制

> 從上面範例得知：
>
> * 256KB 是一個操作被切割成多個 IOPS 的限制
>
> * 250MB 是這個 gp2 硬碟空間的吞吐量吞吐量限制

真實的吞吐量限制會根據 AWS 硬碟類型（gp2, gp3）和其設定而不同。gp2 的最大限制是 334GB。

*  GP2 的[吞吐量限制計算](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#GP2Throughput)

  ```
  Throughput in MiB/s = ((Volume size in GiB) × (IOPS per GiB) × (I/O size in KiB))
  ```

* GP3 的吞吐量須明確設定，如果您設定超過 125MB 則需額外付費。GP3 可以直接增加限制。

> EC2 也有屬於自己的吞吐量限制，可能您也會遇到。而在新的 Instance 類型比較不會遇到，但還是需要注意網路 IO 的流量。

### 儲存體佇列

儲存體還有可能讓 I/O 進入待辦狀態（Pending）。表示硬碟可能跟不上 IOPS 的請求。CloudWatch 有個稱為 `VolumeQueueLength` 的測量值就是用來描述上面的佇列值。下面我們會進一步解釋。

## GP2 vs GP3 EBS

接著我們來討論一下最常見的兩種硬碟類型 GP2, GP3。一般來說您最好選 GP3，但 GP2 類型在建立 EC2 還是維持預設，並且也是 RDS 資料庫目前唯一使用的類型。

下面是關於他們的介紹：

### GP2

GP2 類型是隨著空間大小來擴展 IOPS，每 GB 的空間取得 3 IOPS，對於低於 33.333GB 的硬碟則至少獲得 100 IOPS

吞吐量也是隨著硬碟空間擴展。如上提到的計算吞吐量限制的公式 `Throughput in MiB/s = ((Volume size in GiB) × (IOPS per GiB) × (I/O size in KiB))`。上限為 250MB。

突發機制信用（Burst Credits）會隨著時間跟著 IOPS 數量增加。（當您突然超過您原本用量限制，會有一個突發機制 Burst Credits 就是暫時會讓你付費使用超出的用量）

1000GB 之後，您會取得 3000 IOPS 並且不再有突發機制。您可以直接控制 GP2 硬碟的部分就只有空間大小。

長時間來說，AWS 當然會希望您購買更多空間大小來解決 IOPS 和吞吐量限制的問題。

### GP3

GP3 是新類型的硬碟類型。它們比 GP2 便宜並且提供更一致的效能。沒有突發機制的部分。**GP3 應該是您的首選硬碟類型。**

這種類型從 3000 IOPS 開始起跳，125MB 吞吐量。然後您可以直接[付費](https://aws.amazon.com/ebs/general-purpose/)調整 IOPS 和吞吐量。

> 從 GP2 轉移到 GP3 是非常簡單的 - 基本上就是調用一個 API 或在 Console 直接調整。AWS 在更改為 GP3 後“優化”EBS 驅動器時可能會有點降速。

這種設定的方式讓您能在調整 GP3 空間大小時還能控制預算。舉例來說您建立了 30GB 的硬碟但是讓吞吐量提高。

使用 GP3 可以比起預購 IOPS （io2）更能有效率的節省預算。[參考官方文件](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)

## EBS 是效能瓶頸嗎？

OK，那我們要怎麼知道 EBS 是造成效能瓶頸的地方？您需要觀察 CloudWatch。下面有些重要的指標

### 硬碟指標

下面在 CloudWatch 的指標是跟 EBS 相關而不是 EC2。

#### `BurstBalance`

該指標為百分比，應用在 GP2 硬碟上的，GP2 硬碟可以臨時增加到 3000 IOPS。當硬碟超出配給的 IOPS 時，突發機制的信用就會開始扣。到 0% 就表示硬碟達到 IOPS 的極限。

GP3 沒有這種機制因此沒有這個指標。

#### `VolumeQueueLength`

這是測量有多少操作 “Pending” 了；數字越大越不好。

如果佇列數量非常高，表示您的工作負載在執行超過硬碟能負擔的操作。這是過多操作和吞吐量結合的指標。

那是什麼造成高佇列？基本上官方表示取決不同情況，可[參考](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-io-characteristics.html#ebs-io-volume-queue)。

根據曾經管理一些超載硬碟的經驗，可以參考這裡的經驗比對您的狀況。

比較低負載的數值會小於 1，例如 0.00008，而超載的工作，數值通常是低於 2 ，偶爾出現 10 

#### `Volume[Read/Write][Bytes/Ops]`

這個指標是用來告訴您多少 Bytes 正在讀寫，以及多少操作正在發生。值很高的話不一定是不好，但肯定要注意。

### Instance 指標

下面的指標是在 CloudWatch 跟 EC2 有關的。Nitro/EBS 優化 EC2 實例有些和它們相關的指標。

下面的指標可以在 CloudWatch 找到，但不會出現在 EC2 管理介面的監測頁籤。

1. EBSByteBalance
2. ESBIOBalance
3. ESB[READ/Write] Bytes
4. ESB[Read/Write] Ops

上面這些指標是用來表示 EC2 相關的一些峰值。然而並沒有找到具體的說明。

下面是一些找到的參考資料：

* [This AWS blog post from 2018](https://aws.amazon.com/blogs/compute/improving-application-performance-and-reducing-costs-with-amazon-ebs-optimized-instance-burst-capability/)
* [CloudWatch Metrics for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html#ebs-metrics-nitro)
* [This page on EBS-Optimized instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-optimized.html#ebs-optimization-performance)

雖然找不到具體的說明，但最後一篇文章提供了一些有用的說明

> 您可以使用 EBSIOBalance 和 EBSByteBalance 指標來協助您確認 Instance 的規格是否正確。這些指標使用百分比表示。指標數值一直都很低的表示目標機器可以考慮提升規格（100% 是用扣的到 0% 等於用光）。如果數值都沒有低於 100% 的是可以考慮調降。

## RDS 空間和指標

RDS 相關的 CloudWatch 指標並沒有和 EBS 區分開來。值得注意的 RDS 指標（非 Aurora 數據庫）包括與我們上面看到的指標類似：

* Burst Balance (依據硬碟類型，例如 GP2 1000G 則有 3000 IOPS）
* Queue Depth
* Read/Write Throughput
* Read/Write/Total IOPS

## RDS 儲存空間

在紀錄本文的同時，RDS 並不支援 GP3，這表示您需要配置更多的空間來增加 IOPS 和 Throughput

> 您也可以使用 io2 - 預購 IOPS 類型的硬碟，這樣就可以付費購買特定數量的 IOPS，但當然它們比一般 GP2 貴。

**一個非常盛行的小技巧就是把 RDS 的硬碟升到 5334GB。這樣做不只消除了突發機制，也達到 GP2 IOPS 的上限 16000。吞吐量也到 334GB。**

您也可查詢 [官方表格](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#EBSVolumeTypes_gp2) 來確認各種規格的差異。

**擴展 RDS 儲存空間是單向的**。 一旦您升級就不能往下調，只能開新機器搬資料。

如果您正在查詢解決 RDS EBS 瓶頸的問題，您可能發現給資料庫 1000GB 就暫時夠了。每 GB `$0.115`，大小 5334GB 單一 AZ 一個月要 `$613.41`，多 AZ 一個月要 `$1226.82`。

## Aurora EBS 效能

Aurora 是一個特別的例子。當建立 Aurora 資料庫的時候，您會注意到您沒有任何關於儲存空間的設定。

那麼會有什麼效能上的差異？您可以從[這篇文章](https://aws.amazon.com/blogs/database/best-storage-practices-for-running-production-workloads-on-hosted-databases-with-amazon-rds-or-amazon-ec2/)看到一點線索。

文章似乎表示您的限制是根據實例的規格而不是 EBS 限制：

> 如果您工作負載需要高 IOPS 和高吞吐量，建議您移至 Aurora，其為針對高負載的需求提供了高效能，高可用性的解決方案。
>
> 當使用 Aurora，確保了技術上沒有 IOPS 的限制，但吞吐量還是會基於 Aurora Instance 規格的限制，如果要高吞吐量就選擇高規格的 Aurora Instance。

## 使用快照（Snapshots）

另外值得一提的是當從快照建立 EBS 的時候，您[不會立刻取得最大的效能](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSPerformance.html#initialize)。

您可以通過一些步驟設定，如果有需要的話。AWS 稱這個過程為初始化。您可以閱讀上面連結的文章或者使用 [Fast Restore](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-fast-snapshot-restore.html) 功能。

快照的這些狀況同樣適用在 RDS 資料庫從快照還原的時候。

## 參考資源

* [What You Need to Know About IOPS](https://cloudcasts.io/article/what-you-need-to-know-about-iops)

