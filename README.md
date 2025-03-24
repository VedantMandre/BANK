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
```
SELECT otd.old_reference_number, tdr.reference_number
FROM deposit.test_recon_obs_time_deposit_data otd
LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
ON otd.old_reference_number = tdr.reference_number
WHERE otd.old_reference_number IS NOT NULL;
```
```
INSERT INTO deposit.test_recon_time_deposit_rollover (
    trade_number, reference_number, principal_amount, 
    maturity_date, currency_code, accrued_interest, 
    interest_amount, branch_code, funding_source, 
    obs_number, account_number, settlement_account, 
    maturity_status, status
)
SELECT 
    otd.trade_number, 
    otd.old_reference_number,  -- Insert old_reference_number as reference_number
    otd.time_deposit_amount, 
    otd.maturity_date, 
    otd.currency, 
    otd.interest_accrued_till_date, 
    otd.interest_at_maturity, 
    otd.branch, 
    otd.funding_source, 
    otd.obs_code, 
    otd.time_deposit_account_number, 
    otd.settlement_account_number, 
    otd.maturity_status, 
    'Finalized'  -- Mark newly inserted ones as Finalized
FROM deposit.test_recon_obs_time_deposit_data otd
LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
ON otd.old_reference_number = tdr.reference_number
WHERE otd.old_reference_number IS NOT NULL
AND tdr.reference_number IS NULL;  -- Only insert if not already present
```
```

CREATE OR REPLACE PROCEDURE deposit.sync_time_deposit_rollover()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Step 1: Debugging - Check which old_reference_numbers already exist
    RAISE NOTICE 'Checking existing references...';
    PERFORM otd.old_reference_number, tdr.reference_number
    FROM deposit.test_recon_obs_time_deposit_data otd
    LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
    ON otd.old_reference_number = tdr.reference_number
    WHERE otd.old_reference_number IS NOT NULL;

    -- Step 2: Insert new records into the rollover table if not already present
    RAISE NOTICE 'Inserting new records...';
    INSERT INTO deposit.test_recon_time_deposit_rollover (
        trade_number, reference_number, principal_amount, 
        maturity_date, currency_code, accrued_interest, 
        interest_amount, branch_code, funding_source, 
        obs_number, account_number, settlement_account, 
        maturity_status, status
    )
    SELECT 
        otd.trade_number, 
        otd.old_reference_number,  
        otd.time_deposit_amount, 
        otd.maturity_date, 
        otd.currency, 
        otd.interest_accrued_till_date, 
        otd.interest_at_maturity, 
        otd.branch, 
        otd.funding_source, 
        otd.obs_code, 
        otd.time_deposit_account_number, 
        otd.settlement_account_number, 
        otd.maturity_status, 
        'Finalized'  
    FROM deposit.test_recon_obs_time_deposit_data otd
    LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
    ON otd.old_reference_number = tdr.reference_number
    WHERE otd.old_reference_number IS NOT NULL
    AND tdr.reference_number IS NULL;

    -- Confirmation Message
    RAISE NOTICE 'Sync completed successfully!';
END;
$$;
```
```
UPDATED STORED PROC
```
```
CREATE OR REPLACE PROCEDURE deposit.sync_time_deposit_rollover()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Step 1: Debugging - Check which old_reference_numbers already exist
    RAISE NOTICE 'Checking existing references...';
    PERFORM otd.old_reference_number, tdr.reference_number
    FROM deposit.test_recon_obs_time_deposit_data otd
    LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
    ON otd.old_reference_number = tdr.reference_number
    WHERE otd.old_reference_number IS NOT NULL;

    -- Step 2: Only process records where old_reference_number is not NULL 
    -- (these are the rolled-over time deposits)
    RAISE NOTICE 'Processing rolled-over time deposits...';
    
    WITH rolled_over_data AS (
        SELECT 
            otd.trade_number, 
            otd.old_reference_number AS reference_number,  
            otd.time_deposit_amount AS principal_amount, 
            otd.maturity_date, 
            otd.currency AS currency_code, 
            otd.interest_accrued_till_date AS accrued_interest, 
            otd.interest_at_maturity AS interest_amount, 
            otd.branch AS branch_code, 
            otd.funding_source, 
            otd.obs_code AS obs_number, 
            otd.time_deposit_account_number AS account_number, 
            otd.settlement_account_number AS settlement_account, 
            otd.maturity_status, 
            'Finalized' AS status
        FROM deposit.test_recon_obs_time_deposit_data otd
        WHERE otd.old_reference_number IS NOT NULL
    )
    
    INSERT INTO deposit.test_recon_time_deposit_rollover (
        trade_number, reference_number, principal_amount, 
        maturity_date, currency_code, accrued_interest, 
        interest_amount, branch_code, funding_source, 
        obs_number, account_number, settlement_account, 
        maturity_status, status
    )
    SELECT 
        trade_number, reference_number, principal_amount, 
        maturity_date, currency_code, accrued_interest, 
        interest_amount, branch_code, funding_source, 
        obs_number, account_number, settlement_account, 
        maturity_status, status
    FROM rolled_over_data
    ON CONFLICT (reference_number) DO UPDATE SET
        trade_number = EXCLUDED.trade_number,
        principal_amount = EXCLUDED.principal_amount,
        maturity_date = EXCLUDED.maturity_date,
        currency_code = EXCLUDED.currency_code,
        accrued_interest = EXCLUDED.accrued_interest,
        interest_amount = EXCLUDED.interest_amount,
        branch_code = EXCLUDED.branch_code,
        funding_source = EXCLUDED.funding_source,
        obs_number = EXCLUDED.obs_number,
        account_number = EXCLUDED.account_number,
        settlement_account = EXCLUDED.settlement_account,
        maturity_status = EXCLUDED.maturity_status,
        status = 'Finalized';
        
    -- Step 3: Update status to 'Finalized' for all records that exist in both tables
    RAISE NOTICE 'Updating status for reconciled records...';
    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET status = 'Finalized'
    FROM deposit.test_recon_obs_time_deposit_data otd
    WHERE tdr.reference_number = otd.old_reference_number
    AND otd.old_reference_number IS NOT NULL;
    
    -- Confirmation Message
    RAISE NOTICE 'Sync completed successfully!';
END;
$$;
```
