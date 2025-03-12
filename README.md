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

    <changeSet id="1" author="user">
        <createTable tableName="beneficiary">
            <column name="beneficiary_id" type="VARCHAR" constraints="nullable:false"/>
            <column name="sun_id" type="VARCHAR" constraints="nullable:false, unique:true"/>
            <column name="payment_id" type="VARCHAR"/>
            <column name="branch_id" type="VARCHAR"/>
            <column name="beneficiary_name" type="VARCHAR" constraints="nullable:false"/>
            <column name="beneficiary_account_number" type="VARCHAR" constraints="nullable:false"/>
            <column name="beneficiary_organization" type="VARCHAR"/>
            <column name="beneficiary_department" type="VARCHAR"/>
            <column name="beneficiary_street_name" type="VARCHAR"/>
            <column name="beneficiary_building_number" type="VARCHAR"/>
            <column name="beneficiary_building_name" type="VARCHAR"/>
            <column name="beneficiary_floor" type="VARCHAR"/>
            <column name="beneficiary_po_box" type="VARCHAR"/>
            <column name="beneficiary_room" type="VARCHAR"/>
            <column name="beneficiary_postal_code" type="VARCHAR"/>
            <column name="beneficiary_city" type="VARCHAR" constraints="nullable:false"/>
            <column name="beneficiary_town_location" type="VARCHAR"/>
            <column name="beneficiary_district" type="VARCHAR"/>
            <column name="beneficiary_state_province" type="VARCHAR"/>
            <column name="beneficiary_country" type="VARCHAR" constraints="nullable:false"/>
            <column name="bank_identifier" type="VARCHAR" constraints="nullable:false"/>
            <column name="bank_name" type="VARCHAR" constraints="nullable:false"/>
            <column name="bank_department" type="VARCHAR"/>
            <column name="bank_sub_department" type="VARCHAR"/>
            <column name="bank_street_name" type="VARCHAR"/>
            <column name="bank_building_number" type="VARCHAR"/>
            <column name="bank_building_name" type="VARCHAR"/>
            <column name="bank_floor" type="VARCHAR"/>
            <column name="bank_po_box" type="VARCHAR"/>
            <column name="bank_room" type="VARCHAR"/>
            <column name="bank_postal_code" type="VARCHAR"/>
            <column name="bank_city" type="VARCHAR" constraints="nullable:false"/>
            <column name="bank_town_location" type="VARCHAR"/>
            <column name="bank_district_name" type="VARCHAR"/>
            <column name="bank_state_province" type="VARCHAR"/>
            <column name="bank_country" type="VARCHAR" constraints="nullable:false"/>
            <column name="TSID" type="BIGINT" constraints="nullable:false"/>
            <column name="created_by" type="VARCHAR"/>
            <column name="created_at" type="TIMESTAMP"/>
            <column name="updated_by" type="VARCHAR"/>
            <column name="updated_at" type="TIMESTAMP"/>
            <column name="record_status" type="VARCHAR" constraints="nullable:false">
                <constraints checkConstraint="record_status IN ('Draft', 'Complete')" defaultValue="Draft"/>
            </column>
        </createTable>
    </changeSet>
</databaseChangeLog>
```
