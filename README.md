# Oracle APEX Bulk Data Upload Via CSV

A simple and reusable **Oracle APEX solution for bulk uploading data from a CSV file into an Oracle database table** using `APEX_DATA_PARSER`.

This project demonstrates how to:

- Upload a CSV file using Oracle APEX
- Read the uploaded file from `APEX_APPLICATION_TEMP_FILES`
- Parse CSV data using `APEX_DATA_PARSER`
- Insert multiple CSV records into an Oracle table
- Track successful and failed records
- Calculate CSV upload execution time
- Display an upload summary to the user

---

## ⭐ Support

If you find this project useful, please consider giving it a ⭐ **Star**!

Your support helps others discover the solution and motivates me to share more technical knowledge with the Oracle APEX community.

---

# Features

- 📁 CSV File Upload
- 📊 Bulk Data Import
- ⚡ Fast CSV Parsing
- ✅ Success Record Count
- ❌ Failed Record Count
- ⏱️ Execution Time
- 🔄 Automatic ID Generation
- 🗄️ Oracle Database Integration
- 💻 Oracle APEX Based
- 🧩 Easy to Customize

---

# Technology Stack

- Oracle APEX
- Oracle Database
- PL/SQL
- APEX_DATA_PARSER
- APEX_APPLICATION_TEMP_FILES

---
# Demo Use Case

For demonstration purposes, this project uses a simple employee table.

## Demo Table

```sql
CREATE TABLE DEMO_EMPLOYEE
(
    EMP_ID          NUMBER PRIMARY KEY,
    EMP_NAME        VARCHAR2(100),
    DEPARTMENT      VARCHAR2(100),
    DESIGNATION     VARCHAR2(100),
    SALARY          NUMBER,
    JOINING_DATE    DATE
);
```

---

# Sample CSV File

Create a CSV file named:

```text
employees.csv
```

with the following data:

```csv
EMP_NAME,DEPARTMENT,DESIGNATION,SALARY,JOINING_DATE
John Smith,IT,Software Developer,100000,01-JAN-2026
David Lee,HR,HR Officer,80000,05-JAN-2026
Michael Brown,Finance,Accountant,90000,10-JAN-2026
Sarah Wilson,IT,APEX Developer,120000,15-JAN-2026
```

The first row contains the column headers.

---

# Setup 1: Create APEX Page Items

Create the following page items.

## 1. File Upload Item

Create a page item:

```text
Name: P5_FILE
Type: File Upload
Storage Type: APEX_APPLICATION_TEMP_FILES
```

Recommended settings:

```text
Allow Multiple Files: No
Required: Yes
```

---

## 2. Upload Button

Create a button:

```text
Name: UPLOAD
Label: Upload
Action: Submit Page
```

Recommended Font Awesome icon:

```text
fa-upload
```

---

# Setup 2: Create Page Process

Create a page process:

```text
Process Type: Execute Code
Processing Point: After Submit
Server-Side Condition:
When Button Pressed = UPLOAD
```

Use the following PL/SQL code:

```sql
DECLARE
    l_blob           BLOB;
    l_filename       VARCHAR2(200);

    l_curr_id        NUMBER;
    l_success        NUMBER := 0;
    l_failed         NUMBER := 0;

    -- Timing variables
    l_start_time     NUMBER;
    l_end_time       NUMBER;
    l_duration       NUMBER;
BEGIN

    ------------------------------------------------------------------
    -- Start Timer
    ------------------------------------------------------------------
    l_start_time := DBMS_UTILITY.GET_TIME;


    ------------------------------------------------------------------
    -- 1. Get Starting ID
    ------------------------------------------------------------------
    SELECT NVL(MAX(EMP_ID), 0)
      INTO l_curr_id
      FROM DEMO_EMPLOYEE;


    ------------------------------------------------------------------
    -- 2. Get Uploaded File
    ------------------------------------------------------------------
    SELECT blob_content,
           filename
      INTO l_blob,
           l_filename
      FROM apex_application_temp_files
     WHERE name = :P5_FILE;


    ------------------------------------------------------------------
    -- 3. Parse CSV File
    --
    -- First row is treated as the header.
    ------------------------------------------------------------------
    FOR rec IN
    (
        SELECT
            col001,
            col002,
            col003,
            col004,
            col005
        FROM TABLE
        (
            apex_data_parser.parse
            (
                p_content   => l_blob,
                p_file_name => l_filename,
                p_skip_rows => 1
            )
        )
    )
    LOOP

        BEGIN

            ------------------------------------------------------------------
            -- Generate New Employee ID
            ------------------------------------------------------------------
            l_curr_id := l_curr_id + 1;


            ------------------------------------------------------------------
            -- Insert CSV Data
            ------------------------------------------------------------------
            INSERT INTO DEMO_EMPLOYEE
            (
                EMP_ID,
                EMP_NAME,
                DEPARTMENT,
                DESIGNATION,
                SALARY,
                JOINING_DATE
            )
            VALUES
            (
                l_curr_id,
                rec.col001,
                rec.col002,
                rec.col003,
                rec.col004,
                TO_DATE(rec.col005, 'DD-MON-YYYY')
            );


            ------------------------------------------------------------------
            -- Success Counter
            ------------------------------------------------------------------
            l_success := l_success + 1;


        EXCEPTION
            WHEN OTHERS THEN

                ------------------------------------------------------------------
                -- If current row fails, continue processing remaining rows
                ------------------------------------------------------------------
                l_failed := l_failed + 1;

        END;

    END LOOP;


    ------------------------------------------------------------------
    -- Commit Uploaded Records
    ------------------------------------------------------------------
    COMMIT;


    ------------------------------------------------------------------
    -- Calculate Execution Time
    ------------------------------------------------------------------
    l_end_time := DBMS_UTILITY.GET_TIME;

    l_duration :=
        (l_end_time - l_start_time) / 100;


    ------------------------------------------------------------------
    -- Reset File Item
    ------------------------------------------------------------------
    :P5_FILE := NULL;


    ------------------------------------------------------------------
    -- Display Upload Summary
    ------------------------------------------------------------------
    apex_application.g_print_success_message :=
          '<b>CSV Upload Completed</b><br>'
       || 'File: ' || l_filename || '<br>'
       || 'Success: ' || l_success || ' rows<br>'
       || 'Failed: ' || l_failed || ' rows<br>'
       || 'Execution Time: '
       || TO_CHAR(l_duration, '990.00')
       || ' seconds';


EXCEPTION

    WHEN OTHERS THEN

        ROLLBACK;

        apex_application.g_print_success_message :=
            'Process Error: ' || SQLERRM;

END;
```


# Important Notes

## 1. CSV Header

The first row of the CSV file is treated as the header because the process uses:

```sql
p_skip_rows => 1
```

Therefore, the CSV should contain column headers.

---

# Recommended Production Architecture

For a production-level CSV upload application, the following architecture is recommended:

```text
                 CSV File
                     │
                     ▼
          APEX File Upload
                     │
                     ▼
      APEX_APPLICATION_TEMP_FILES
                     │
                     ▼
           APEX_DATA_PARSER
                     │
                     ▼
              Staging Table
                     │
                     ▼
               Validation
                /       \
               /         \
              ▼           ▼
        Valid Rows     Invalid Rows
              │           │
              ▼           ▼
        Target Table    Error Log
              │
              ▼
        Upload Summary
```

# License

This project is provided for educational and demonstration purposes.

You are free to modify and adapt it for your own Oracle APEX projects.

---

## Author

Developed for the Oracle APEX community.

If this project helped you, don't forget to ⭐ **Star the repository**!
