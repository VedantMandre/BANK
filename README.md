```
SELECT
    obs.trade_number,
    obs.old_reference_number,
    obs.time_deposit_amount AS obs_amount,
    rollover.principal_amount AS rollover_amount,
    obs.interest_rate AS obs_rate,
    rollover.interest_rate AS rollover_rate,
    obs.start_date AS obs_start,
    rollover.start_date AS rollover_start,
    CASE 
        WHEN rollover.reference_number IS NULL THEN 'MISSING_IN_INTERNAL'
        WHEN ABS(obs.time_deposit_amount - rollover.principal_amount) > 1 THEN 'AMOUNT_MISMATCH'
        WHEN ABS(obs.interest_rate - rollover.interest_rate) > 0.01 THEN 'RATE_MISMATCH'
        WHEN obs.maturity_status IS DISTINCT FROM rollover.maturity_status THEN 'STATUS_MISMATCH'
        ELSE 'MATCHED'
    END AS recon_status
FROM deposit.test_recon_obs_time_deposit_data obs
LEFT JOIN deposit.test_recon_time_deposit_rollover rollover
    ON obs.old_reference_number = rollover.reference_number;

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
```
OBS Table (test_recon_obs_time_deposit_data)	Rollover Table (test_recon_time_deposit_rollover)	Purpose / Notes
trade_number	trade_number	Trade reference
reference_number	reference_number	New reference for term deposit
old_reference_number	reference_number (in rollover)	Links to previous record
start_date	start_date	Term deposit start
maturity_date	maturity_date	Term deposit end
tenor (as integer)	tenor (as varchar)	Duration — may need type casting
currency	currency_code	Currency
interest_rate	interest_rate	Agreed rate
interest_at_maturity	interest_amount	Expected return
interest_accrued_till_date	accrued_interest	Accrued till date
maturity_status	maturity_status	Active / Matured / Rolled-over
time_deposit_amount	principal_amount	Principal / deposit amount
settlement_account_number	settlement_account	Payout account
funding_source	funding_source	Source of funds
sun_id	sun_id	Customer identifier
account_official_name	account_official_name	Customer account name
trade_type	trade_type	Type of deposit trade
branch	branch_code	Branch information (naming might differ)
done_time	creation_date	Record creation time
update_time	updated_date	Record update time
created_by	created_by	Record creator
updated_by	updated_by	Record modifier
is_active	active	Boolean flag for logical deletion
```
