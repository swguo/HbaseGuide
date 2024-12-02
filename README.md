
# **HBase 單機安裝與配置教學**
描述：
本篇教學提供在 Ubuntu 系統上安裝與配置 HBase 的詳細步驟，適合初學者和進階使用者參考。內容涵蓋系統依賴安裝、HBase 下載與配置、啟動服務，以及透過 HBase Shell 執行基本操作，幫助您快速上手分散式數據庫的部署與應用。

以下是 HBase 在 Ubuntu 系統上的完整安裝和配置教學，包含每一步的詳細操作指令：

---

### **步驟 1: 更新系統並安裝必要依賴**

#### **1. 更新系統軟件包**
```bash
sudo apt update && sudo apt upgrade -y
```

#### **驗證 Java 安裝：**
```bash
java -version
```

---

### **步驟 2: 下載和解壓 HBase**

#### **1. 下載 HBase**
訪問 Apache HBase 官網，選擇合適的版本下載（例如 2.4.16）：
```bash
wget https://downloads.apache.org/hbase/2.5.10/hbase-2.5.10-bin.tar.gz
```

#### **2. 解壓 HBase 安裝包**
```bash
tar -xvzf hbase-2.5.10-bin.tar.gz
sudo mv hbase-2.5.10 /usr/local/hbase
```

---

### **步驟 3: 配置 HBase**

#### **1. 配置環境變量**
編輯 `~/.bashrc` 文件，添加以下內容：
```bash
vim ~/.bashrc
```
```bash
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

加載環境變量：
```bash
source ~/.bashrc
```

#### **2. 編輯 HBase 配置文件**
進入 HBase 配置目錄：
```bash
cd $HBASE_HOME/conf
```

編輯 `hbase-site.xml`：
```bash
vim hbase-site.xml
```

添加以下配置：
```xml
    <property>
        <name>hbase.rootdir</name>
        <value>file:///usr/local/hbase/data</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/usr/local/hbase/zookeeper</value>
    </property>
```

---

### **步驟 4: 啟動 HBase**

#### **1. 啟動 HBase**
```bash
start-hbase.sh
```

#### **2. 驗證 HBase 是否成功啟動**
使用 JPS 檢查進程：
```bash
jps
```

應看到以下進程：
- `HMaster`
- `HRegionServer`

#### **3. 訪問 HBase Web 界面**
默認情況下，HBase 提供 Web 界面：
- **HMaster Web 界面**：`http://localhost:16010`

---

### **步驟 5: 使用 HBase Shell**

#### **1. 進入 HBase Shell**
```bash
hbase shell
```

#### **2. 創建一個表**
```bash
create 'test_table', 'cf'
```

#### **3. 插入數據**
```bash
put 'test_table', 'row1', 'cf:col1', 'value1'
```

#### **4. 查詢數據**
```bash
scan 'test_table'
```

#### **5. 刪除表**
```bash
disable 'test_table'
drop 'test_table'
```