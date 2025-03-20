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
CREATE OR REPLACE PROCEDURE reconcile_time_deposits()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Step 1: Identify rolled over TDs
    WITH rolled_over_tds AS (
        SELECT 
            otd.trade_number,
            otd.reference_number AS new_reference_no,
            otd.old_reference_no AS old_reference_no,
            otd.time_deposit_amount,
            otd.maturity_date,
            otd.currency,
            -- Generate Internal_RF_no if not available
            CONCAT(otd.reference_number, '_RF') AS internal_rf_no 
        FROM 
            deposit.test_recon_obs_time_deposit_data otd
        WHERE 
            otd.old_reference_no IS NOT NULL
    )

    -- Step 2: Insert or Update records in time_deposit_rollover
    INSERT INTO deposit.test_recon_time_deposit_rollover (
        reference_number,
        trade_number,
        principal_amount,
        maturity_date,
        currency_code,
        status,
        internal_rf_no
    )
    SELECT 
        source.old_reference_no,
        source.trade_number,
        source.time_deposit_amount,
        source.maturity_date,
        source.currency,
        'finalized',
        source.internal_rf_no
    FROM rolled_over_tds AS source
    ON CONFLICT (reference_number)
    DO UPDATE SET
        reference_number = EXCLUDED.reference_number,
        status = 'finalized';
END;
$$;
```
```
CREATE OR REPLACE PROCEDURE reconcile_time_deposits()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Step 1: Identify rolled over TDs
    WITH rolled_over_tds AS (
        SELECT 
            otd.trade_number,
            otd.reference_number AS new_reference_no,
            otd.old_reference_no AS old_reference_no,
            otd.time_deposit_amount,
            otd.maturity_date,
            otd.currency,
            -- Generate Internal_RF_no if not available
            CONCAT(otd.reference_number, '_RF') AS internal_rf_no 
        FROM 
            deposit.test_recon_obs_time_deposit_data otd
        WHERE 
            otd.old_reference_no IS NOT NULL
    )

    -- Step 2: Insert or Update records in time_deposit_rollover
    INSERT INTO deposit.test_recon_time_deposit_rollover (
        obs_booking_id,
        reference_number,
        trade_number,
        principal_amount,
        maturity_date,
        currency_code,
        status,
        internal_rf_no
    )
    SELECT 
        gen_random_uuid(),  -- Generating unique booking ID
        source.old_reference_no,
        source.trade_number,
        source.time_deposit_amount,
        source.maturity_date,
        source.currency,
        'finalized',
        source.internal_rf_no
    FROM rolled_over_tds AS source
    ON CONFLICT (reference_number)
    DO UPDATE SET
        reference_number = EXCLUDED.reference_number,
        status = 'finalized';
END;
$$;

```
