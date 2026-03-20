---
Lab:
    Title: 'Create a medallion architecture in a Microsoft Fabric lakehouse'
    Module: 'Organize a Fabric lakehouse using medallion architecture design'
---

# Create a medallion architecture in a Microsoft Fabric lakehouse

In this exercise you will build out a medallion architecture in a Fabric lakehouse using notebooks. You will create a workspace, create a lakehouse, upload data to the bronze layer, transform the data and load it to the silver Delta table, transform the data further and load it to the gold Delta tables, and then explore the dataset and create relationships.

This exercise should take approximately **45** minutes to complete

> **Note**: You'll need a Microsoft Fabric license to complete this exercise. See [Getting started with Fabric](https://learn.microsoft.com/fabric/get-started/fabric-trial) for details of how to enable a free Fabric trial license. You will need a Microsoft *school* or *work* account to do this. If you don't have one, you can [sign up for a trial of Microsoft Office 365 E3 or higher](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans).

## Table of Contents

- [A. Create a workspace and enable data model editing](#a-create-a-workspace-and-enable-data-model-editing)
- [B. Create a lakehouse and upload data to bronze layer](#b-create-a-lakehouse-and-upload-data-to-bronze-layer)
- [C. Transform data and load to bronze Delta table](#c-transform-data-and-load-to-bronze-delta-table)
- [D. Explore data in the bronze layer using the SQL endpoint](#d-explore-data-in-the-bronze-layer-using-the-sql-endpoint)
- [E. Transform data for silver layer](#e-transform-data-for-silver-layer)
- [F. Create a dataset](#f-create-a-dataset)
- [G. Clean up resources](#g-clean-up-resources)

## A. Create a workspace

Before working with data in Fabric, create a workspace with the Fabric trial enabled.

1. Sign into [Microsoft Fabric](https://app.fabric.microsoft.com) at `https://app.fabric.microsoft.com` and select **Power BI**.
2. In the menu bar on the left, select **Workspaces** (the icon looks similar to &#128455;).
3. Create a new workspace with a name of your choice, selecting a licensing mode that includes Fabric capacity (*Trial*, *Premium*, or *Fabric*).
4. When your new workspace opens, it should be empty, as shown here:

    ![Screenshot of an empty workspace in Power BI.](./Images/new-workspace.png)

## B. Create a lakehouse and upload data to bronze layer

Now that you have a workspace, it's time to switch to the *Fabric* experience in the portal and create a data lakehouse for the data files you're going to analyze.
>If you already have a Lakehouse, you can reuse it, ensure: 
>1. Delete the content in Tables and Files folder from previous labs, simply choose **...** icon then **Delete**
>2. Lakehouse schemas is unselected when creating the Lakehouse for this training.

1. At the bottom left, ensure that you have the **Fabric** icon, if it's still in **Power BI** icon, simply click and switch to the **Fabric** icon.

2. In the **Workspace** page view, select **+New item** just below the workspace name, then choose **Lakehouse** from the variety of the **Fabric** items, provide the **Lakehouse** name of your choice, ensure that you unselect the option **Lakehouse schemas** (only apply to this training).

    After a minute or so, a new empty lakehouse will be created. You need to ingest some data into the data lakehouse for analysis. There are multiple ways to do this, but in this exercise you'll simply download a text file to your local computer (or lab VM if applicable) and then upload it to your lakehouse.

3. Download the data file for this exercise from `https://github.com/MicrosoftLearning/dp-data/blob/main/orders.zip`. Extract the files and save them with their original names on your local computer (or lab VM if applicable). There should be 3 files containing sales data for 3 years: 2019.csv, 2020.csv, and 2021.csv.

4. Return to the web browser tab containing your lakehouse, and in the **...** menu for the **Files** folder in the **Explorer** pane, select **New subfolder** and create a folder named **bronze**.

5. In the **...** menu for the **bronze** folder, select **Upload** and **Upload files**, and then upload the 3 files (2019.csv, 2020.csv, and 2021.csv) from your local computer (or lab VM if applicable) to the lakehouse. Use the shift key to upload all 3 files at once.

6. After the files have been uploaded, select the **bronze** folder; and verify that the files have been uploaded, as shown here:

    ![Screenshot of uploaded products.csv file in a lakehouse.](./Images/bronze-files.png)

## C. Transform data and load to bronze Delta table

Now that you have some data in the bronze layer of your lakehouse, you can use a notebook to transform the data and load it to a delta table in the bronze layer, which is the managed Tables/ folder in OneLake.

1. On the **Home** page while viewing the contents of the **bronze** folder in your data lake, in the **Open notebook** menu, select **New notebook**.

    After a few seconds, a new notebook containing a single *cell* will open. Notebooks are made up of one or more cells that can contain *code* or *markdown* (formatted text).

2. When the notebook opens, rename it to your preference by selecting the ⚙️ icon text at the top left of the notebook and entering the new name.

    ![Screenshot of a new notebook named Transform data for bronze.](./Images/sales-notebook-rename.png)

3. Select the existing cell in the notebook, which contains some simple commented-out code. Highlight and delete these two lines - you will not need this code.

   > **Note**: Notebooks enable you to run code in a variety of languages, including Python, Scala, and SQL. In this exercise, you'll use PySpark and SQL. You can also add markdown cells to provide formatted text and images to document your code.

4. **Paste** the following code into the cell:

    ```python
    from pyspark.sql.types import *
    
    # Create the schema for the table
    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])
    
    # Import all files from bronze folder of lakehouse
    df = spark.read.format("csv").option("header", "true").schema(orderSchema).load("Files/bronze/*.csv")
    
    # Display the first 10 rows of the dataframe to preview your data
    display(df.head(10))
    ```

5. Use the ****&#9655;** (*Run cell*)** button on the left of the cell to run the code.

    > **Note**: Since this is the first time you've run any Spark code in this notebook, a Spark session must be started. This means that the first run can take a minute or so to complete. Subsequent runs will be quicker.

6. When the cell command has completed, **review the output** below the cell, which should look similar to this:

    | Index | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | Email | Item | Quantity | UnitPrice | Tax |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO49172 | 1 | 2021-01-01 | Brian Howard | brian23@adventure-works.com | Road-250 Red, 52 | 1 | 2443.35 | 195.468 |
    | 2 |  SO49173 | 1 | 2021-01-01 | Linda Alvarez | linda19@adventure-works.com | Mountain-200 Silver, 38 | 1 | 2071.4197 | 165.7136 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    The code you ran loaded the data from the CSV files in the **bronze** folder into a Spark dataframe, and then displayed the first few rows of the dataframe.

    > **Note**: You can clear, hide, and auto-resize the contents of the cell output by selecting the **...** menu at the top left of the output pane.

7. Now you'll **add columns for data validation and cleanup**, using a PySpark dataframe to add columns and update the values of some of the existing columns. Use the + button to **add a new code block** and add the following code to the cell:

    ```python
    from pyspark.sql.functions import when, lit, col, current_timestamp, input_file_name
    
    # Add columns IsFlagged, CreatedTS and ModifiedTS
    df = df.withColumn("FileName", input_file_name()) \
        .withColumn("IsFlagged", when(col("OrderDate") < '2019-08-01',True).otherwise(False)) \
        .withColumn("CreatedTS", current_timestamp()).withColumn("ModifiedTS", current_timestamp())
    
    # Update CustomerName to "Unknown" if CustomerName null or empty
    df = df.withColumn("CustomerName", when((col("CustomerName").isNull() | (col("CustomerName")=="")),lit("Unknown")).otherwise(col("CustomerName")))
    ```

    The first line of the code imports the necessary functions from PySpark. You're then adding new columns to the dataframe so you can track the source file name, whether the order was flagged as being a before the fiscal year of interest, and when the row was created and modified.

    You're also adding columns for the CustomerID and ItemID, which will be populated later.

    Finally, you're updating the CustomerName column to "Unknown" if it's null or empty.

8. Run the cell to execute the code using the ****&#9655;** (*Run cell*)** button.

9. Next, you'll define the schema for the **sales_bronze** table in the database using Delta Lake format. Create a new code block and add the following code to the cell (Note to replace the "yourlakehouse" in the code):

    ```python
    # Define the schema for the sales_silver table
    
    from pyspark.sql.types import *
    from delta.tables import *
    
    #Replace "yourlakehouse" with your lakehouse name
    DeltaTable.createIfNotExists(spark) \
        .tableName("yourlakehouse.sales_bronze") \
        .addColumn("SalesOrderNumber", StringType()) \
        .addColumn("SalesOrderLineNumber", IntegerType()) \
        .addColumn("OrderDate", DateType()) \
        .addColumn("CustomerName", StringType()) \
        .addColumn("Email", StringType()) \
        .addColumn("Item", StringType()) \
        .addColumn("Quantity", IntegerType()) \
        .addColumn("UnitPrice", FloatType()) \
        .addColumn("Tax", FloatType()) \
        .addColumn("FileName", StringType()) \
        .addColumn("IsFlagged", BooleanType()) \
        .addColumn("CreatedTS", DateType()) \
        .addColumn("ModifiedTS", DateType()) \
        .execute()
    ```

10. Run the cell to execute the code using the ****&#9655;** (*Run cell*)** button.

11. Select the **...** in the Tables section of the lakehouse explorer pane and select **Refresh**. You should now see the new **sales_bronze** table listed. The **&#9650;** (triangle icon) indicates that it's a Delta table.

    ![Screenshot of the sales_bronze table in a lakehouse.](./Images/sales-silver-table.png)

    > **Note**: If you don't see the new table, wait a few seconds and then select **Refresh** again, or refresh the entire browser tab.

12. Now you're going to perform an **upsert operation** on a Delta table, updating existing records based on specific conditions and inserting new records when no match is found. Add a new code block and paste the following code:

    ```python
    # Update existing records and insert new ones based on a condition defined by the columns SalesOrderNumber, OrderDate, CustomerName, and Item.

    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'Tables/sales_bronze')
    
    dfUpdates = df
    
    deltaTable.alias('bronze') \
      .merge(
        dfUpdates.alias('updates'),
        'bronze.SalesOrderNumber = updates.SalesOrderNumber and bronze.OrderDate = updates.OrderDate and bronze.CustomerName = updates.CustomerName and bronze.Item = updates.Item'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "SalesOrderNumber": "updates.SalesOrderNumber",
          "SalesOrderLineNumber": "updates.SalesOrderLineNumber",
          "OrderDate": "updates.OrderDate",
          "CustomerName": "updates.CustomerName",
          "Email": "updates.Email",
          "Item": "updates.Item",
          "Quantity": "updates.Quantity",
          "UnitPrice": "updates.UnitPrice",
          "Tax": "updates.Tax",
          "FileName": "updates.FileName",
          "IsFlagged": "updates.IsFlagged",
          "CreatedTS": "updates.CreatedTS",
          "ModifiedTS": "updates.ModifiedTS"
        }
      ) \
      .execute()
    ```

    This operation is important because it enables you to update existing records in the table based on the values of specific columns, and insert new records when no match is found. This is a common requirement when you're loading data from a source system that may contain updates to existing and new records.

You now have data in your bronze delta table that is ready for further transformation and modeling.

## D. Explore data in the bronze layer using the SQL endpoint

Now that you have data in your bronze layer, you can use the SQL endpoint to explore the data and perform some basic analysis. This is a nice option for you if you're familiar with SQL and want to do some basic exploration of your data. In this exercise we're using the SQL endpoint view in Fabric, but note that you can also use other tools like SQL Server Management Studio (SSMS) and Azure Data Explorer.

1. Navigate back to your workspace and notice that you now have a few Fabric items listed. Select **SQL endpoint** with the same name of your Lakehouse to open it in the SQL endpoint view.

    ![Screenshot of the SQL endpoint in a lakehouse.](./Images/sql-endpoint-item.png)

2. Select **New SQL query** from the ribbon, which will open a SQL query editor. Note that you can rename your query using the **...** menu item next to the existing query name in the lakehouse explorer pane.

   We're going to run two sql queries to explore our data.

3. Paste the following query into the query editor and select **Run**:

    ```sql
    SELECT YEAR(OrderDate) AS Year
        , CAST (SUM(Quantity * (UnitPrice + Tax)) AS DECIMAL(12, 2)) AS TotalSales
    FROM sales_bronze
    GROUP BY YEAR(OrderDate) 
    ORDER BY YEAR(OrderDate)
    ```

    This query calculates the total sales for each year in the sales_bronze table. Your results should look like this:

    ![Screenshot of the results of a SQL query in a lakehouse.](./Images/total-sales-sql.png)

4. Now we'll take a look at which customers are purchasing the most (in terms of quantity). Paste the following query into the query editor and select **Run**:

    ```sql
    SELECT TOP 10 CustomerName, SUM(Quantity) AS TotalQuantity
    FROM sales_bronze
    GROUP BY CustomerName
    ORDER BY TotalQuantity DESC
    ```

      This query calculates the total quantity of items purchased by each customer in the sales_bronze table, and then returns the top 10 customers in terms of quantity.

Data exploration at the bronze layer is useful for basic analysis, but you'll need to transform the data further and model it into a star schema to enable more advanced analysis and reporting. You'll do that in the next section.

## E. Transform data for silver layer

You have successfully taken data from your bronze layer, transformed it, and loaded it into a bronze Delta table. Now you'll use a new notebook to transform the data further, model it into a star schema, and load it into silver Delta tables.

Note that you could have done all of this in a single notebook, but for the purposes of this exercise you're using separate notebooks to demonstrate the process of transforming data from bronze to silver, and silver to gold. This can help with debugging, troubleshooting, and reuse.

1. Return to the **Workspace** home page and select the notebook you used earlier.

2. Create new code cell and **add the following code** to load data to your dataframe and start building out your star schema, then run it:

   ```python
    # Load data to the dataframe as a starting point to create the silver layer
    # Replace "yourlakehouse" with the name of your lakehouse
    df = spark.read.table("yourlakehouse.sales_bronze")
    ```

3. **Add a new code block** and paste the following code to create your date dimension table and run it:

    ```python
    from pyspark.sql.types import *
    from delta.tables import*
    
    # Define the schema for the dimdate_silver table
    # Replace "yourlakehouse" with the name of your lakehouse
    DeltaTable.createIfNotExists(spark) \
        .tableName("yourlakehouse.dimdate_silver") \
        .addColumn("OrderDate", DateType()) \
        .addColumn("Day", IntegerType()) \
        .addColumn("Month", IntegerType()) \
        .addColumn("Year", IntegerType()) \
        .addColumn("mmmyyyy", StringType()) \
        .addColumn("yyyymm", StringType()) \
        .execute()
    ```

    > **Note**: You can run the `display(df)` command at any time to check the progress of your work. In this case, you'd run 'display(dfdimDate_silver)' to see the contents of the dimDate_silver dataframe.

4. In a new code block, **add and run the following code** to create a dataframe for your date dimension, **dimdate_silver**:

    ```python
    from pyspark.sql.functions import col, dayofmonth, month, year, date_format
    
    # Create dataframe for dimDate_silver
    
    dfdimDate_silver = df.dropDuplicates(["OrderDate"]).select(col("OrderDate"), \
            dayofmonth("OrderDate").alias("Day"), \
            month("OrderDate").alias("Month"), \
            year("OrderDate").alias("Year"), \
            date_format(col("OrderDate"), "MMM-yyyy").alias("mmmyyyy"), \
            date_format(col("OrderDate"), "yyyyMM").alias("yyyymm"), \
        ).orderBy("OrderDate")

    # Display the first 10 rows of the dataframe to preview your data

    display(dfdimDate_silver.head(10))
    ```

5. You're separating the code out into new code blocks so that you can understand and watch what's happening in the notebook as you transform the data. In another new code block, **add and run the following code** to update the date dimension as new data comes in:

    ```python
    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'Tables/dimdate_silver')
    
    dfUpdates = dfdimDate_silver
    
    deltaTable.alias('silver') \
      .merge(
        dfUpdates.alias('updates'),
        'silver.OrderDate = updates.OrderDate'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "OrderDate": "updates.OrderDate",
          "Day": "updates.Day",
          "Month": "updates.Month",
          "Year": "updates.Year",
          "mmmyyyy": "updates.mmmyyyy",
          "yyyymm": "yyyymm"
        }
      ) \
      .execute()
    ```

    Congrats! Your date dimension is all set up. Now you'll create your customer dimension.
6. To build out the customer dimension table, **add a new code block**, paste and run the following code:

    ```python
    from pyspark.sql.types import *
    from delta.tables import *
    
    # Create customer_silver dimension delta table
    # Replace "yourlakehouse" with the name of your lakehouse
    DeltaTable.createIfNotExists(spark) \
        .tableName("yourlakehouse.dimcustomer_silver") \
        .addColumn("CustomerName", StringType()) \
        .addColumn("Email",  StringType()) \
        .addColumn("First", StringType()) \
        .addColumn("Last", StringType()) \
        .addColumn("CustomerID", LongType()) \
        .execute()
    ```

7. In a new code block, **add and run the following code** to drop duplicate customers, select specific columns, and split the "CustomerName" column to create "First" and "Last" name columns:

    ```python
    from pyspark.sql.functions import col, split
    
    # Create customer_silver dataframe
    
    dfdimCustomer_silver = df.dropDuplicates(["CustomerName","Email"]).select(col("CustomerName"),col("Email")) \
        .withColumn("First",split(col("CustomerName"), " ").getItem(0)) \
        .withColumn("Last",split(col("CustomerName"), " ").getItem(1)) 
    
    # Display the first 10 rows of the dataframe to preview your data

    display(dfdimDate_silver.head(10))
    ```

     Here you have created a new DataFrame dfdimCustomer_silver by performing various transformations such as dropping duplicates, selecting specific columns, and splitting the "CustomerName" column to create "First" and "Last" name columns. The result is a DataFrame with cleaned and structured customer data, including separate "First" and "Last" name columns extracted from the "CustomerName" column.

8. Next we'll **create the ID column for our customers**. In a new code block, paste and run the following:

    ```python
    from pyspark.sql.functions import monotonically_increasing_id, col, when, coalesce, max, lit
    
    # Replace "yourlakehouse" with the name of your lakehouse
    dfdimCustomer_temp = spark.read.table("yourlakehouse.dimCustomer_silver")
    
    MAXCustomerID = dfdimCustomer_temp.select(coalesce(max(col("CustomerID")),lit(0)).alias("MAXCustomerID")).first()[0]
    
    dfdimCustomer_silver = dfdimCustomer_silver.join(dfdimCustomer_temp,(dfdimCustomer_silver.CustomerName == dfdimCustomer_temp.CustomerName) & (dfdimCustomer_silver.Email == dfdimCustomer_temp.Email), "left_anti")
    
    dfdimCustomer_silver = dfdimCustomer_silver.withColumn("CustomerID",monotonically_increasing_id() + MAXCustomerID + 1)

    # Display the first 10 rows of the dataframe to preview your data

    display(dfdimDate_silver.head(10))
    ```

    Here you're cleaning and transforming customer data (dfdimCustomer_silver) by performing a left anti join to exclude duplicates that already exist in the dimCustomer_silver table, and then generating unique CustomerID values using the monotonically_increasing_id() function.

9. Now you'll ensure that your customer table remains up-to-date as new data comes in. **In a new code block**, paste and run the following:

    ```python
    from delta.tables import *

    deltaTable = DeltaTable.forPath(spark, 'Tables/dimcustomer_silver')
    
    dfUpdates = dfdimCustomer_silver
    
    deltaTable.alias('bronze') \
      .merge(
        dfUpdates.alias('updates'),
        'bronze.CustomerName = updates.CustomerName AND bronze.Email = updates.Email'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "CustomerName": "updates.CustomerName",
          "Email": "updates.Email",
          "First": "updates.First",
          "Last": "updates.Last",
          "CustomerID": "updates.CustomerID"
        }
      ) \
      .execute()
    ```

10. Now you'll **repeat those steps to create your product dimension**. In a new code block, paste and run the following:

    ```python
    from pyspark.sql.types import *
    from delta.tables import *
    
    # Replace "yourlakehouse" with the name of your lakehouse
    DeltaTable.createIfNotExists(spark) \
        .tableName("yourlakehouse.dimproduct_silver") \
        .addColumn("ItemName", StringType()) \
        .addColumn("ItemID", LongType()) \
        .addColumn("ItemInfo", StringType()) \
        .execute()
    ```

11. **Add another code block** to create the **customer_silver** dataframe. You'll use this later on the Sales join.
  
    ```python
    from pyspark.sql.functions import col, split, lit
    
    # Create Customer_silver dataframe, this dataframe will be used later on on the Sales join
    
    dfdimProduct_silver = df.dropDuplicates(["Item"]).select(col("Item")) \
        .withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
        .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) 
    
    # Display the first 10 rows of the dataframe to preview your data

    display(dfdimProduct_silver.head(10))
       ```

12. Now you'll create IDs for your **dimProduct_silver table**. Add the following syntax to a new code block and run it:

    ```python
    from pyspark.sql.functions import monotonically_increasing_id, col, lit, max, coalesce
    
    #Replace "yourlakehouse" with the name of your lakehouse
    dfdimProduct_temp = spark.read.table("yourlakehouse.dimProduct_silver")
    
    MAXProductID = dfdimProduct_temp.select(coalesce(max(col("ItemID")),lit(0)).alias("MAXItemID")).first()[0]
    
    dfdimProduct_silver = dfdimProduct_silver.join(dfdimProduct_temp,(dfdimProduct_silver.ItemName == dfdimProduct_temp.ItemName) & (dfdimProduct_silver.ItemInfo == dfdimProduct_temp.ItemInfo), "left_anti")
    
    dfdimProduct_silver = dfdimProduct_silver.withColumn("ItemID",monotonically_increasing_id() + MAXProductID + 1)
    
    # Display the first 10 rows of the dataframe to preview your data

    display(dfdimProduct_silver.head(10))
    ```

13. Similar to what you've done with your other dimensions, you need to ensure that your product table remains up-to-date as new data comes in. **In a new code block**, paste and run the following:

    ```python
    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'Tables/dimproduct_silver')
            
    dfUpdates = dfdimProduct_silver
            
    deltaTable.alias('bronze') \
      .merge(
            dfUpdates.alias('updates'),
            'bronze.ItemName = updates.ItemName AND bronze.ItemInfo = updates.ItemInfo'
            ) \
            .whenMatchedUpdate(set =
            {
               
            }
            ) \
            .whenNotMatchedInsert(values =
             {
              "ItemName": "updates.ItemName",
              "ItemInfo": "updates.ItemInfo",
              "ItemID": "updates.ItemID"
              }
              ) \
              .execute()
      ```

      This calculates the next available product ID based on the current data in the table, assigns these new IDs to the products, and then displays the updated product information (if the display command is uncommented).

      **Now that you have your dimensions built out, the final step is to create the fact table.**

14. **In a new code block**, paste and run the following code to create the **fact table**:

    ```python
    from pyspark.sql.types import *
    from delta.tables import *
    
    # Replace "yourlakehouse" with the name of your lakehouse
    DeltaTable.createIfNotExists(spark) \
        .tableName("yourlakehouse.factsales_silver") \
        .addColumn("CustomerID", LongType()) \
        .addColumn("ItemID", LongType()) \
        .addColumn("OrderDate", DateType()) \
        .addColumn("Quantity", IntegerType()) \
        .addColumn("UnitPrice", FloatType()) \
        .addColumn("Tax", FloatType()) \
        .execute()
    ```

15. **In a new code block**, paste and run the following code to create a **new dataframe** to combine sales data with customer and product information include customer ID, item ID, order date, quantity, unit price, and tax:

    ```python
    from pyspark.sql.functions import col
    
    # Replace "yourlakehouse" with the name of your lakehouse
    dfdimCustomer_temp = spark.read.table("yourlakehouse.dimCustomer_silver")
    dfdimProduct_temp = spark.read.table("yourlakehouse.dimProduct_silver")
    
    df = df.withColumn("ItemName",split(col("Item"), ", ").getItem(0)) \
        .withColumn("ItemInfo",when((split(col("Item"), ", ").getItem(1).isNull() | (split(col("Item"), ", ").getItem(1)=="")),lit("")).otherwise(split(col("Item"), ", ").getItem(1))) \
    
    
    # Create Sales_silver dataframe
    
    dffactSales_silver = df.alias("df1").join(dfdimCustomer_temp.alias("df2"),(df.CustomerName == dfdimCustomer_temp.CustomerName) & (df.Email == dfdimCustomer_temp.Email), "left") \
            .join(dfdimProduct_temp.alias("df3"),(df.ItemName == dfdimProduct_temp.ItemName) & (df.ItemInfo == dfdimProduct_temp.ItemInfo), "left") \
        .select(col("df2.CustomerID") \
            , col("df3.ItemID") \
            , col("df1.OrderDate") \
            , col("df1.Quantity") \
            , col("df1.UnitPrice") \
            , col("df1.Tax") \
        ).orderBy(col("df1.OrderDate"), col("df2.CustomerID"), col("df3.ItemID"))
    
    # Display the first 10 rows of the dataframe to preview your data
    
    display(dffactSales_silver.head(10))
    ```

16. Now you'll ensure that sales data remains up-to-date by running the following code in a **new code block**:

    ```python
    from delta.tables import *
    
    deltaTable = DeltaTable.forPath(spark, 'Tables/factsales_silver')
    
    dfUpdates = dffactSales_silver
    
    deltaTable.alias('bronze') \
      .merge(
        dfUpdates.alias('updates'),
        'bronze.OrderDate = updates.OrderDate AND bronze.CustomerID = updates.CustomerID AND bronze.ItemID = updates.ItemID'
      ) \
       .whenMatchedUpdate(set =
        {
          
        }
      ) \
     .whenNotMatchedInsert(values =
        {
          "CustomerID": "updates.CustomerID",
          "ItemID": "updates.ItemID",
          "OrderDate": "updates.OrderDate",
          "Quantity": "updates.Quantity",
          "UnitPrice": "updates.UnitPrice",
          "Tax": "updates.Tax"
        }
      ) \
      .execute()
    ```

     Here you're using Delta Lake's merge operation to synchronize and update the factsales_silver table with new sales data (dffactSales_silver). The operation compares the order date, customer ID, and item ID between the existing data (bronze table) and the new data (updates DataFrame), updating matching records and inserting new records as needed.
17. On the notebook Home menu ribbon, select the ■ icon (**Stop session**) to end the session.

You now have a curated, modeled **silver** layer that can be used for reporting and analysis.

## F. Create a dataset

In your workspace, you can now use the silver layer to create a report and analyze the data. You can access via Semantic Model in your workspace to create relationships and measures for reporting.

1. In your workspace, navigate to your **lakehouse** .
2. Select **New Semantic Model** from the home menu ribbon.
3. Select your transformed silver tables to include in your dataset, name the Semantic Model to your preference, choose the **Direct Lake on OneLake** mode, and select **Confirm**.
   - dimdate_silver
   - dimcustomer_silver
   - dimproduct_silver
   - factsales_silver

    This will open the Semantic Model in Fabric where you can create relationships and measures, as show here:

    ![Screenshot of a dataset in Fabric.](./Images/dataset-relationships.png)

From here, you or other members of your data team can create reports and dashboards based on the same Semantic Model in your lakehouse. These reports will be connected directly to the silver layer of your lakehouse, so they'll always reflect the latest data.

## G. Clean up resources

In this exercise, you've learned how to create a medallion architecture in a Microsoft Fabric lakehouse.

If you've finished exploring your lakehouse, you can delete the workspace you created for this exercise.

1. In the bar on the left, select the icon for your workspace to view all of the items it contains.
2. In the **...** menu on the toolbar, select **Workspace settings**.
3. In the **Other** section, select **Remove this workspace**.
