```
-- Create schema
CREATE SCHEMA IF NOT EXISTS bank;

-- Create table
CREATE TABLE bank.branches (
    branch_id INT PRIMARY KEY,
    branch_code VARCHAR(10) UNIQUE,
    description TEXT NOT NULL
);
```
```
INSERT INTO bank.branches (branch_id, branch_code, description) VALUES
(7832, 'BRZ', 'Banco Sumitomo Mitsui Brasileiro S.A.'),
(7860, 'CDA', 'Canada Branch'),
(8801, 'USA', 'NEW YORK BRANCH'),
(8802, 'LDN', 'SMBC Bank International plc'),
(8804, 'DUS', 'SMBC DUSSELDORF'),
(8807, 'BRU', 'BRUSSELS BRANCH'),
(8100, 'FFT', 'SMBC Bank EU - Frankfurt'),
(8792, NULL, 'SMBC LONDON BRANCH'),
(8815, NULL, 'CAYMAN'),
(8816, NULL, 'SMBC BANK EU - Milan'),
(8872, 'PAR', 'Paris Branch'),
(8874, 'UAE', 'SMBC DIFC BRANCH - DUBAI'),
(8881, NULL, 'SMFD IRELAND'),
(8902, NULL, 'THE SUMITOMO MITSUI BANKING CORP.');
```
```
<databaseChangeLog>
    <changeSet id="CMT-1593-01" author="vmandre">
        <comment>Create branch table</comment>
        <sql>
            CREATE TABLE reference.branche_details (
                branch_id INT UNIQUE,  -- Unique key but NOT primary key
                branch_code VARCHAR(10) UNIQUE,
                description TEXT NOT NULL
            );
        </sql>
        <rollback>
            DROP TABLE bank.branches;
        </rollback>
    </changeSet>

    <changeSet id="CMT-1593" author="vmandre">
        <comment>Insert branch data</comment>
        <sql>
            INSERT INTO reference.branche_details (branch_id, branch_code, description) VALUES
            (7832, 'BRZ', 'Banco Sumitomo Mitsui Brasileiro S.A.'),
            (7860, 'CDA', 'Canada Branch'),
            (8801, 'USA', 'NEW YORK BRANCH'),
            (8802, 'LDN', 'SMBC Bank International plc'),
            (8804, 'DUS', 'SMBC DUSSELDORF'),
            (8807, 'BRU', 'BRUSSELS BRANCH'),
            (8100, 'FFT', 'SMBC Bank EU - Frankfurt'),
            (8792, NULL, 'SMBC LONDON BRANCH'),
            (8815, NULL, 'CAYMAN'),
            (8816, NULL, 'SMBC BANK EU - Milan'),
            (8872, 'PAR', 'Paris Branch'),
            (8874, 'UAE', 'SMBC DIFC BRANCH - DUBAI'),
            (8881, NULL, 'SMFD IRELAND'),
            (8902, NULL, 'THE SUMITOMO MITSUI BANKING CORP.');
        </sql>
    </changeSet>
</databaseChangeLog>
```

```
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <changeSet id="CMT-671-01.01.02" author="tsnow">
        <comment>Create beneficiary table</comment>
        <sql>
        CREATE TABLE IF NOT EXISTS reference.beneficiary (
            beneficiary_id VARCHAR NOT NULL,
            sun_id VARCHAR UNIQUE NOT NULL,
            payment_id VARCHAR,
            branch_id VARCHAR,
            beneficiary_name VARCHAR NOT NULL,
            beneficiary_account_number VARCHAR NOT NULL,
            beneficiary_organization VARCHAR,
            beneficiary_department VARCHAR,
            beneficiary_street_name VARCHAR,
            beneficiary_building_number VARCHAR,
            beneficiary_building_name VARCHAR,
            beneficiary_floor VARCHAR,
            beneficiary_po_box VARCHAR,
            beneficiary_room VARCHAR,
            beneficiary_postal_code VARCHAR,
            beneficiary_city VARCHAR NOT NULL,
            beneficiary_town_location VARCHAR,
            beneficiary_district VARCHAR,
            beneficiary_state_province VARCHAR,
            beneficiary_country VARCHAR NOT NULL,
            bank_identifier VARCHAR NOT NULL,
            bank_name VARCHAR NOT NULL,
            bank_department VARCHAR,
            bank_sub_department VARCHAR,
            bank_street_name VARCHAR,
            bank_building_number VARCHAR,
            bank_building_name VARCHAR,
            bank_floor VARCHAR,
            bank_po_box VARCHAR,
            bank_room VARCHAR,
            bank_postal_code VARCHAR,
            bank_city VARCHAR NOT NULL,
            bank_town_location VARCHAR,
            bank_district_name VARCHAR,
            bank_state_province VARCHAR,
            bank_country VARCHAR NOT NULL,
            TSID BIGINT NOT NULL,
            created_by VARCHAR,
            created_at TIMESTAMP DEFAULT current_timestamp,
            updated_by VARCHAR,
            updated_at TIMESTAMP,
            record_status VARCHAR NOT NULL CHECK (record_status IN ('Draft', 'Complete')) DEFAULT 'Draft'
        );
        </sql>
    </changeSet>
</databaseChangeLog>
```
```
# **Beneficiaries Table Schema**

## **Primary Key**
| Column Name          | Data Type      | Description                              |
|----------------------|----------------|------------------------------------------|
| `beneficiary_id`      | BIGINT NOT NULL | Unique identifier for each beneficiary   |

---

## **Core Identifiers**
| Column Name          | Data Type              | Description                           |
|----------------------|------------------------|---------------------------------------|
| `sun_id`              | VARCHAR UNIQUE NOT NULL | System Unique Number (SUN) for identification |
| `payment_id`          | VARCHAR                 | Payment identifier                   |
| `branch_id`           | VARCHAR                 | Branch identifier                    |

---

## **Beneficiary Information**
| Column Name                   | Data Type         | Description                           |
|--------------------------------|-------------------|---------------------------------------|
| `beneficiary_name`             | VARCHAR            | Name of the beneficiary               |
| `beneficiary_account_number`   | VARCHAR NOT NULL   | Unique account number for the beneficiary |
| `beneficiary_organization`     | VARCHAR            | Organization name                     |
| `beneficiary_department`       | VARCHAR            | Department name                       |
| `beneficiary_street_name`      | VARCHAR            | Street address                        |
| `beneficiary_building_number`  | VARCHAR            | Building number                       |
| `beneficiary_building_name`    | VARCHAR            | Building name                         |
| `beneficiary_floor`            | VARCHAR            | Floor number                          |
| `beneficiary_po_box`           | VARCHAR            | PO Box number                         |
| `beneficiary_postal_code`      | VARCHAR            | Postal/Zip code                       |
| `beneficiary_city`             | VARCHAR NOT NULL   | City of the beneficiary               |
| `beneficiary_town_location`    | VARCHAR            | Town location                         |
| `beneficiary_district`         | VARCHAR            | District information                  |
| `beneficiary_state_province`   | VARCHAR            | State or province                     |
| `beneficiary_country`          | VARCHAR NOT NULL   | Country of the beneficiary            |

---

## **Bank Details**
| Column Name                       | Data Type         | Description                           |
|-----------------------------------|-------------------|---------------------------------------|
| `beneficiary_bank_identifier`     | VARCHAR NOT NULL   | Unique bank identifier                |
| `beneficiary_bank_name`           | VARCHAR NOT NULL   | Bank name                              |
| `beneficiary_bank_department`     | VARCHAR            | Bank department                        |
| `beneficiary_bank_street_name`    | VARCHAR            | Bank's street address                  |
| `beneficiary_bank_building_number`| VARCHAR            | Bank's building number                 |
| `beneficiary_bank_building_name`  | VARCHAR            | Bank's building name                   |
| `beneficiary_bank_floor`          | VARCHAR            | Bank's floor number                    |
| `beneficiary_bank_po_box`         | VARCHAR            | Bank's PO Box                          |
| `beneficiary_bank_postal_code`    | VARCHAR            | Bank's postal/zip code                 |
| `beneficiary_bank_city`           | VARCHAR NOT NULL   | Bank's city                            |
| `beneficiary_bank_town_location`  | VARCHAR            | Bank's town location                   |
| `beneficiary_bank_district_name`  | VARCHAR            | Bank's district                        |
| `beneficiary_bank_state_province` | VARCHAR            | Bank's state or province               |
| `beneficiary_bank_country`        | VARCHAR NOT NULL   | Bank's country                         |

---

## **Status and Audit Fields**
| Column Name        | Data Type                             | Description                                |
|--------------------|---------------------------------------|--------------------------------------------|
| `approval_status`   | VARCHAR DEFAULT 'Pending'             | Status of approval                         |
| `created_by`        | VARCHAR                               | User who created the record                |
| `created_at`        | TIMESTAMP DEFAULT `current_timestamp` | Creation timestamp                         |
| `updated_by`        | VARCHAR                               | User who last updated the record            |
| `updated_at`        | TIMESTAMP                             | Timestamp of last update                   |
| `record_status`     | VARCHAR NOT NULL CHECK (`record_status` IN ('Draft', 'Complete')) DEFAULT 'Draft' | Status of the record (e.g., Draft, Complete) |

---

## **SQL Table Definition**
```sql
CREATE TABLE IF NOT EXISTS beneficiaries (
    beneficiary_id BIGINT NOT NULL,
    sun_id VARCHAR UNIQUE NOT NULL,
    payment_id VARCHAR,
    branch_id VARCHAR,
    beneficiary_name VARCHAR,
    beneficiary_account_number VARCHAR NOT NULL,
    beneficiary_organization VARCHAR,
    beneficiary_department VARCHAR,
    beneficiary_street_name VARCHAR,
    beneficiary_building_number VARCHAR,
    beneficiary_building_name VARCHAR,
    beneficiary_floor VARCHAR,
    beneficiary_po_box VARCHAR,
    beneficiary_postal_code VARCHAR,
    beneficiary_city VARCHAR NOT NULL,
    beneficiary_town_location VARCHAR,
    beneficiary_district VARCHAR,
    beneficiary_state_province VARCHAR,
    beneficiary_country VARCHAR NOT NULL,
    beneficiary_bank_identifier VARCHAR NOT NULL,
    beneficiary_bank_name VARCHAR NOT NULL,
    beneficiary_bank_department VARCHAR,
    beneficiary_bank_street_name VARCHAR,
    beneficiary_bank_building_number VARCHAR,
    beneficiary_bank_building_name VARCHAR,
    beneficiary_bank_floor VARCHAR,
    beneficiary_bank_po_box VARCHAR,
    beneficiary_bank_postal_code VARCHAR,
    beneficiary_bank_city VARCHAR NOT NULL,
    beneficiary_bank_town_location VARCHAR,
    beneficiary_bank_district_name VARCHAR,
    beneficiary_bank_state_province VARCHAR,
    beneficiary_bank_country VARCHAR NOT NULL,
    approval_status VARCHAR DEFAULT 'Pending',
    created_by VARCHAR,
    created_at TIMESTAMP DEFAULT current_timestamp,
    updated_by VARCHAR,
    updated_at TIMESTAMP,
    record_status VARCHAR NOT NULL CHECK (record_status IN ('Draft', 'Complete')) DEFAULT 'Draft'
);
```
