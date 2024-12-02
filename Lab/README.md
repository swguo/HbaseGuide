# **從 MySQL 到 Hive 再到 HBase 的數據流轉全攻略**
描述：
本篇教學詳細說明如何實現從 MySQL 的北風資料庫導入數據到 Hive，使用 HiveQL 進行數據處理，並將結果存儲到 HBase。通過結合 Sqoop、Hive 和 HBase，完成數據提取、轉換和存儲（ETL）的一體化流程，適合大數據開發者快速部署高效數據管道。

實現從 **MySQL 的北風資料庫**中導入數據到 **Hive**，執行查詢操作後將結果存儲到 **HBase**：

---

### **步驟 1: 從 MySQL 導入數據到 Hive**

#### **1. 使用 Sqoop 將 MySQL 表導入到 HDFS**

首先，確保 MySQL 和 Hive 環境已配置，然後使用 **Sqoop** 將 MySQL 表（如 Customers 和 Orders）導入 HDFS。

- 導入 Customers 表：
  ```bash
  sqoop import \
      --connect jdbc:mysql://localhost/northwind \
      --username root --password "hadoop_csim" \
      --table Customers \
      --target-dir /user/hive/northwind/customers \
      --m 1 \
      --driver com.mysql.cj.jdbc.Driver
  ```

- 導入 Orders 表：
  ```bash
  sqoop import \
      --connect jdbc:mysql://localhost/northwind \
      --username root --password "hadoop_csim" \
      --table Orders \
      --target-dir /user/hive/northwind/orders \
      --m 1 \
      --driver com.mysql.cj.jdbc.Driver
  ```

#### **2. 在 Hive 中創建外部表**
進入 Hive shell
```bash
hive
```
使用 Hive 創建與 Sqoop 導入的 HDFS 目錄相關聯的外部表：

- Customers 表：
  ```sql
  CREATE EXTERNAL TABLE customers (
      CustomerID STRING,
      CompanyName STRING,
      ContactName STRING,
      ContactTitle STRING,
      Address STRING,
      City STRING,
      Region STRING,
      PostalCode STRING,
      Country STRING,
      Phone STRING,
      Fax STRING
  )
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/user/hive/northwind/customers';
  ```

- Orders 表：
  ```sql
  CREATE EXTERNAL TABLE orders (
      OrderID INT,
      CustomerID STRING,
      EmployeeID INT,
      OrderDate STRING,
      RequiredDate STRING,
      ShippedDate STRING,
      ShipVia INT,
      Freight DOUBLE,
      ShipName STRING,
      ShipAddress STRING,
      ShipCity STRING,
      ShipRegion STRING,
      ShipPostalCode STRING,
      ShipCountry STRING
  )
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/user/hive/northwind/orders';
  ```

### **步驟 2: 在 Hive 中執行 Join 操作**

編寫 **HiveQL** 查詢以連接 `Customers` 和 `Orders` 表，獲取每位客戶的訂單資料。

#### **1. 查詢每位客戶的訂單資料**
執行以下查詢將結果存儲在 Hive 的內部表中：
```sql
CREATE TABLE customer_orders AS
SELECT
    c.CustomerID,
    c.CompanyName,
    c.ContactName,
    c.Phone,
    o.OrderID,
    o.OrderDate,
    o.Freight
FROM
    customers c
JOIN
    orders o
ON
    c.CustomerID = o.CustomerID;
```

此查詢會生成一個包含客戶與其訂單數據的表 `customer_orders`。

---

---
離開 Hive shell
```bash
exit;
```
### **步驟 3: 將查詢結果存儲到 HBase**


#### **1. 在 HBase 中創建目標表**
進入 HBase shell，創建一個表以存儲查詢結果：
```bash
hbase shell
```
執行以下命令創建表：
```bash
create 'customer_orders', 'cf'
```

#### **2. 在 Hive 中創建與 HBase 對應的外部表**
使用 `HBaseStorageHandler` 創建外部表：
```sql
CREATE TABLE hbase_customer_orders (
    rowkey STRING,
    CustomerID STRING,
    CompanyName STRING,
    ContactName STRING,
    Phone STRING,
    OrderID INT,
    OrderDate STRING,
    Freight DOUBLE
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES (
    "hbase.columns.mapping" = ":key,cf:CustomerID,cf:CompanyName,cf:ContactName,cf:Phone,cf:OrderID,cf:OrderDate,cf:Freight"
)
TBLPROPERTIES (
    "hbase.table.name" = "customer_orders"
);
```
```bash
exit
```

#### **3. 將 Hive 結果寫入 HBase**
進入 Hive shell
```bash
hive
```
使用 `INSERT INTO TABLE` 將 `customer_orders` 表的數據插入到 HBase：
```sql
INSERT INTO TABLE hbase_customer_orders
SELECT
    CONCAT(CustomerID, '_', OrderID) AS rowkey, -- 合成行鍵
    CustomerID,
    CompanyName,
    ContactName,
    Phone,
    OrderID,
    OrderDate,
    Freight
FROM
    customer_orders;
```
離開 Hive
```bash
exit;
```
---

### **步驟 4: 驗證數據**

#### **1. 在 HBase 中檢查數據**
進入 HBase shell
```bash
hbase shell
```

檢查數據是否已正確插入：
```bash
scan 'customer_orders'
```
離開 hbase
```bash
exit
```
#### **2. 在 Hive 中查詢 HBase 表**
```bash
hive
```
可以在 Hive 中直接查詢與 HBase 關聯的表：
```sql
SELECT * FROM hbase_customer_orders LIMIT 10;
```
離開 hive
```bash
exit;
```
---

### **總結**

這套流程實現了以下功能：
1. 從 MySQL 中提取數據，並導入到 Hive 表。
2. 使用 HiveQL 執行數據處理操作（如 Join）。
3. 將處理後的結果存儲到 HBase，實現實時查詢需求。