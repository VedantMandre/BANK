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
SELECT 
    otd.trade_number,
    otd.old_reference_number,
    tdr.reference_number,
    tdr.status
FROM deposit.test_recon_obs_time_deposit_data otd
JOIN deposit.test_recon_time_deposit_rollover tdr
ON otd.old_reference_number = tdr.reference_number;
```
```
SELECT 
    otd.old_reference_number, 
    COUNT(tdr.reference_number) AS matched_records
FROM deposit.test_recon_obs_time_deposit_data otd
LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
ON otd.old_reference_number = tdr.reference_number
GROUP BY otd.old_reference_number
ORDER BY matched_records DESC; 
```
