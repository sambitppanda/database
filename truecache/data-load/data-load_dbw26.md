# Prepare and Warm True Cache

## Introduction

In this lab, you will work with the preloaded transaction schema, apply True Cache KEEP, and warm True Cache.

The DBW26 environment is already provisioned. You do not need to create the transactions user, create tables, or populate seed data.

*Estimated Time:* 20 minutes

<if type="nonsandbox">
Watch the video for a quick walk-through of Lab 4: Prepare and Warm True Cache.
[Lab 4](videohub:1_mz228rvo)
[Lab 4](videohub:1_yayzolzj)
</if>

### Objectives

In this lab, you will:
* Validate the preloaded transactions schema.
* Apply KEEP to selected transaction objects.
* Warm True Cache through the Java client application.
* Review True Cache warm-up progress, cache-hit ratios, and fetch-latency statistics.

### Prerequisites

This lab assumes you have:
* An Oracle Cloud account
* All previous labs successfully completed

## Task 1: Review Preloaded Transaction Objects

![Full LiveLab routing and cache warmup](images/full-livelab-routing-and-warmup.png " ")

1. Validate that the transaction tables already exist in the Primary database.

    ```
    <copy>
    sudo podman exec -it prod /bin/bash
    export ORACLE_SID=ORCLCDB
    sqlplus / as sysdba
    alter session set container=ORCLPDB1;
    set pages 100 lines 180
    select owner, table_name from dba_tables where owner='TRANSACTIONS' order by table_name fetch first 20 rows only;
    exit
    exit
    </copy>
    ```

2. The output should include owner `TRANSACTIONS` and tables such as `ACCOUNTS`, `PAYMENTS`, and `PAYMENT_VECTORS`. The exact row order can vary.

## Task 2: Apply KEEP and Verify the Keep List

1. Apply KEEP to the `TRANSACTIONS.ACCOUNTS` table.

    ```
    <copy>
    sudo podman exec -it truedb /bin/bash
    export ORACLE_SID=TRUEDB
    sqlplus / as sysdba
    alter session set container=ORCLPDB1;
    execute dbms_cacheutil.true_cache_keep('TRANSACTIONS','ACCOUNTS');
    execute dbms_cacheutil.true_cache_keep('TRANSACTIONS','ACCOUNTS_PK');
    execute dbms_cacheutil.true_cache_keep('TRANSACTIONS','PAYMENTS');
    execute dbms_cacheutil.true_cache_keep('TRANSACTIONS','PAYMENTS_PK');
    execute dbms_cacheutil.true_cache_keep('TRANSACTIONS','PAYMENTS_UK');
    set pages 100 lines 220
    select owner, object_name, object_type from dba_objects where data_object_id in (select data_object_id from v$true_cache_keep) order by owner, object_type, object_name;
    exit
    exit
    </copy>
    ```

2. Confirm that the kept objects are listed. The result should include `ACCOUNTS`, `ACCOUNTS_PK`, `PAYMENTS`, `PAYMENTS_PK`, and `PAYMENTS_UK` under owner `TRANSACTIONS`. `PAYMENT_VECTORS` is intentionally handled in the vector-search lab.

## Task 3: Warm True Cache

1. Run the warmup application from the app container.

    ```
    <copy>
    read -rsp 'Transactions password: ' DB_PASS; echo
    sudo podman exec -e DB_PASS="$DB_PASS" -it appclient /bin/bash
    cd /stage/clientapp
    USE_TC_CONN=Y METRICS_PORT=9091 ./TransactionsApp.sh warmup
    exit
    </copy>
    ```

2. Check True Cache warmup and hit-ratio statistics.

    ```
    <copy>
    sudo podman exec -it truedb /bin/bash
    export ORACLE_SID=TRUEDB
    sqlplus / as sysdba
    alter session set container=ORCLPDB1;
    set pages 100 lines 220
    select name, value, unit from v$true_cache_stat order by name;
    exit
    exit
    </copy>
    ```

    The query returns one row for each True Cache statistic. Review the prewarm progress, cache-hit ratios, and fetch-latency values; the numeric values depend on the current cache state and workload.

Continue to the next lab.

## Learn More
[True Cache documentation](https://docs.oracle.com/en/database/oracle/oracle-database/23/odbtc/overview-oracle-true-cache.html)

## Acknowledgements
* **Authors** - Sambit Panda, Consulting Member of Technical Staff, Oracle Database Product Management
* **Contributors** - Pankaj Chandiramani, Shefali Bhargava, Jyoti Verma, Nithin Thekkupadam Narayanan
* **Last Updated By/Date** - Sambit Panda, Consulting Member of Technical Staff, Sep 2026
