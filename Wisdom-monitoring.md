# 14.  Node Monitoring Service
## 14.1 Installation and Deployment
&#160;&#160;&#160;&#160;&#160;&#160;See： https://github.com/WisedomChainGroup/wisdom-operations

&#160;&#160;&#160;&#160;&#160;&#160;Pull the latest releases, unzip and modify the config/application.properties configuration file,java-jarmonitor.jar start the jar package directly.

|Parameter item | Parameter name|Effect
|:----:|:----:|:----:
|JDBC URL of the database | DATA_SOURCE_URL|Connect to database
|Database User name| DB_USERNAME|Connect to database
|Database Password|DB_PASSWORD|Connect to database
|Operating system password|System_Password|root password

## 14.2 Function Description
&#160;&#160;&#160;&#160;&#160;&#160;1) Bifurcation recovery

&#160;&#160;&#160;&#160;&#160;&#160;Real time detection of data synchronization, if there is a bifurcation, delete the corresponding bifurcation, and continue to synchronize the data. Listen to the bound node for 10s / time. When it is found that the block hash is not consistent with the 2/3 neighbor nodes, stop the node mirroring, delete the corresponding block and restart the node for resynchronization.

&#160;&#160;&#160;&#160;&#160;&#160;2) Fixture Block Monitoring

&#160;&#160;&#160;&#160;&#160;&#160;Real time monitoring of data synchronization, if found to stay at a high level for a long time, send e-mail to inform the fixture block.

&#160;&#160;&#160;&#160;&#160;&#160;3) Import Function

&#160;&#160;&#160;&#160;&#160;&#160;The official provides backup data, and operation and maintenance tools provide the function of directly importing backup data. Using pg_dump exports the SQL statement corresponding to the Copy and executes it directly in the corresponding database. (not yet open)

&#160;&#160;&#160;&#160;&#160;&#160;4) Export function

&#160;&#160;&#160;&#160;&#160;&#160;Directly export the transaction data queried by address to Excel. POI is used to export the query results. Export byte format data and restore to relational data.

&#160;&#160;&#160;&#160;&#160;&#160;5) Early warning notice

&#160;&#160;&#160;&#160;&#160;&#160;It is an optional function. When listening to a node and finding a bifurcation, CPU 100% occupancy rate, node fixture block, and node stop running will send email notification. JavaMail is used to send mail.

&#160;&#160;&#160;&#160;&#160;&#160;6) Log collection

&#160;&#160;&#160;&#160;&#160;&#160;The corresponding log data information can be viewed and exported according to the date and log keywords. (not yet open)

&#160;&#160;&#160;&#160;&#160;&#160;7) Authentication settings

&#160;&#160;&#160;&#160;&#160;&#160;The operation and maintenance tools need to have user permissions. By default, they are divided into three roles: administrator, operator and query only.The server of operation and maintenance tools needs to be authenticated, and arbitrary query is prohibited.

## 14.3 Front End Operation
&#160;&#160;&#160;&#160;&#160;&#160;1)Login page, enter the user name and password to login

![monitoring-login](../img/monitoring-login.png)

&#160;&#160;&#160;&#160;&#160;&#160;2)Information page, showing basic information

![monitoring-index](../img/monitoring-index.png)

&#160;&#160;&#160;&#160;&#160;&#160;3)Console, manage and bind nodes

![monitoring-node](../img/monitoring-node.png)

&#160;&#160;&#160;&#160;&#160;&#160;4)Early warning information, management of receiving and sending mail information

![monitoring-email](../img/monitoring-email.png)

&#160;&#160;&#160;&#160;&#160;&#160;5)Authentication setting, management user setting permissions

![monitoring-user](../img/monitoring-user.png)

&#160;&#160;&#160;&#160;&#160;&#160;6)Branch repair, manually delete blocks and branch recovery

![monitoring-user](../img/monitoring-repair.png)