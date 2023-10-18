# SCD2
SCD Type 2 Documentation

I have recently implemented a solution for managing Type 2 slowly changing dimensions (SCD) in a data warehousing environment. SCD Type 2 is a technique used to track historical changes in dimension data over time. In this documentation, I will briefly explain the tables used in the task and provide the solution implemented.

Tables Used:

Staging Table:

ID: A numeric field representing the unique identifier for each record.
NAME: A field storing the name associated with the record.
ADDRESS: A field containing the address information for the record.
STARTDATE: A date field indicating the start date of the record.
Destination (DEST) Table:

SURROGATE: A numeric field serving as the surrogate key for each record in the destination table.
ID: A numeric field representing the unique identifier for each record.
NAME: A field storing the name associated with the record.
ADDRESS: A field containing the address information for the record.
IND: A numeric field indicating the status or indicator of the record.
STARTDATE: A timestamp field representing the start date and time of the record.
ENDDATE: A date field indicating the end date of the record.
Solution:

To handle the Type 2 SCD in this scenario, I have developed a PL/SQL procedure named SCD2. The procedure utilizes the MERGE statement to perform the necessary updates and inserts in the destination table based on the data in the staging table.

Below is the code for the SCD2 procedure:

sql
Copy
CREATE OR REPLACE PROCEDURE SCD2
AS
BEGIN
  MERGE INTO DEST t
  USING STAGING s
  ON (t.ID = s.ID)
  WHEN MATCHED THEN
    UPDATE SET
      t.ENDDATE = CASE WHEN t.ADDRESS <> s.ADDRESS THEN s.STARTDATE ELSE t.ENDDATE END,
      t.IND = CASE WHEN t.ADDRESS <> s.ADDRESS THEN 0 ELSE t.IND END
  WHEN NOT MATCHED THEN
    INSERT (ID, NAME, ADDRESS, IND, STARTDATE, ENDDATE)
    VALUES (s.ID, s.NAME, s.ADDRESS, 1, SYSDATE, TO_DATE('12/31/2050', 'MM/DD/YYYY'));

  MERGE INTO DEST t
  USING STAGING s
  ON (t.ID = s.ID AND t.ADDRESS = s.ADDRESS)
  WHEN MATCHED THEN
    UPDATE SET
      t.IND = CASE WHEN t.ADDRESS <> s.ADDRESS THEN 0 ELSE t.IND END
  WHEN NOT MATCHED THEN
    INSERT (ID, NAME, ADDRESS, IND, STARTDATE, ENDDATE)
    VALUES (s.ID, s.NAME, s.ADDRESS, 1, SYSDATE, TO_DATE('12/31/2050', 'MM/DD/YYYY'));
END SCD2;
/
The SCD2 procedure includes two MERGE statements. The first MERGE statement handles the updates and inserts for records that have a different address in the staging table compared to the destination table. It updates the ENDDATE field in the destination table based on the condition and sets the IND field accordingly.

The second MERGE statement handles updates and inserts for records that have the same ID and address in both tables. It only updates the IND field based on the condition.

This procedure can be executed to perform the SCD Type 2 operations on the destination table using the data from the staging table.

Please note that the code provided is a simplified representation and may require modifications to fit your specific requirements and environment. Also, ensure that you have the necessary permissions and privileges to execute the procedure and access the tables.
