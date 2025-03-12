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
