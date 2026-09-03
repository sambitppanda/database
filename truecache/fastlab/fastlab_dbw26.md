# FastLab: Quick Guided Demo

## Introduction

FastLab is a visual, six-step demonstration of Oracle True Cache. The command center uses the pre-provisioned transaction environment and runs the database checks, JDBC workloads, cache warmup, availability test, and vector searches through guided controls.

The environment contains the `TRANSACTIONS` schema with preloaded `ACCOUNTS`, `PAYMENTS`, and payment-vector sample data. FastLab performs the demonstration without requiring terminal commands or manual database setup.

Estimated Time: 25 minutes.

## Objectives

- Confirm that the Primary database, True Cache, and application server are healthy.
- See read-only work routed to True Cache through one logical connection.
- Keep and warm the transaction tables and indexes in True Cache.
- Compare Primary and True Cache read throughput and latency while Primary receives update activity.
- Verify that True Cache continues serving reads while Primary is unavailable.
- Use semantic search to find similar payment profiles with Oracle AI Vector Search.

## Before You Begin

1. Open the LiveLabs desktop and launch the True Cache LiveLab UI.
2. Select **FastLab: Quick Guided Demo** from the experience chooser.
3. Keep the command center open while completing each step. The right-hand panel shows environment health, the current workload, and the command or query being demonstrated behind the active step.

![FastLab experience chooser](images/fastlab-experience-chooser.png " ")

## FastLab Validation and Recovery

FastLab uses a host-level web proxy in addition to the `prod`, `truedb`, and `appclient` containers. The proxy is not a `truecache-ui.service` systemd service, so a missing systemd unit does not indicate a database failure.

### If the FastLab page does not open

1. In the LiveLabs desktop terminal, confirm that the UI port is listening:

    ```
    <copy>
    sudo ss -lntp | grep ':8080'
    </copy>
    ```

2. If there is no listener, start the UI proxy from the supplied application directory:

    ```
    <copy>
    cd /home/opc/LiveLabs_clientapp_warmup_Jul2025-1
    sudo bash ./redeploy.sh --no-ui-build
    </copy>
    ```

3. Refresh `http://localhost:8080` in the LiveLabs desktop browser.
4. If the page opens but the environment is unhealthy, continue with the container validation below and refresh the page after the containers recover.

### If a FastLab service is not healthy

1. Check all three containers:

    ```
    <copy>
    sudo podman ps -a --format 'table {{.Names}}\t{{.Status}}'
    </copy>
    ```

2. Start any stopped lab containers:

    ```
    <copy>
    sudo podman start prod truedb appclient
    </copy>
    ```

3. Wait for the database containers to report **healthy**, then refresh the FastLab page. Oracle database startup can take several minutes after a host restart.
4. Do not begin the guided steps until **Primary DB**, **True Cache**, and **App Server** show **HEALTHY**.

If one of the named containers does not exist, stop the lab and contact the lab administrator. The instance does not contain the pre-provisioned environment required by FastLab.

## Ready Check

1. Wait for the environment check to finish.
2. Confirm that **Primary DB**, **True Cache**, and **App Server** show **HEALTHY**.
3. Select **Next step**.

The environment is pre-provisioned. This step confirms that the services needed by the rest of the demonstration are available.

![FastLab ready check](images/fastlab-ready-check.png " ")

## Routing Demo

1. Select **Run True Cache Validation**.
2. Select **Run BasicApp setReadOnly Test**.
3. Review the evidence panel.
4. Confirm that the read-only operation reaches True Cache and that the logical connection still uses Primary for write operations.
5. Select **Next step**.

This demonstrates the application behavior that makes True Cache transparent to the application: the application uses one connection, while the driver routes read-only work to True Cache and read-write work to Primary.

![FastLab routing demo](images/fastlab-routing-demo.png " ")

## Cache Warmup

1. Select **ACCOUNTS** and **PAYMENTS**.
2. Select **Apply KEEP**.
3. Confirm that each selected table and its available indexes appear as **KEPT** in the object list.
4. Select **Warm Up**.
5. Wait for the table, index, and overall progress indicators to complete.
6. Expand **Performance timelines** or the cache statistics panel to review the resulting hit-ratio and fetch statistics.
7. Select **Next step**.

Keeping the objects tells True Cache which transaction data should remain in its memory cache. The warmup reads the objects before the performance demonstration, so the comparison measures a useful cached-read path rather than an empty cache.

![FastLab cache warmup](images/fastlab-cache-warmup.png " ")

## Primary vs True Cache Performance and Lag

1. Leave the default thread and duration values selected.
2. FastLab starts three update-only background jobs on Primary automatically. These jobs update existing `ACCOUNTS` rows and create write and replication activity; they do not add rows to the dataset.
3. Allow the background jobs to run briefly before starting the read workload.
4. Select **Run: Primary Only** and review the Primary read TPS and latency.
5. Select **Run: True Cache** and review the True Cache read TPS and latency.
6. Select **Run Parallel Comparison** when you want to see both read paths during the same interval. Compare the live read chart, read TPS cards, and latency table.
7. Expand **Performance timelines** to review:
   - Transport lag and apply lag from replication.
   - True Cache, RAM, and flash hit ratios.
   - Single-block, multiblock, and list-of-blocks fetch latency.
8. Select **Next step**.

The read comparison is intentionally shown separately from the write activity. Primary continues to handle the updates, while the True Cache read path can serve kept data from memory and receive the replicated changes.

![FastLab performance proof](images/fastlab-performance-proof.png " ")

## Availability: Primary Down, True Cache Still Serving

1. Select **Start Comparison** and allow both read paths to begin.
2. Select **Stop Primary**.
3. Confirm that Primary changes to an unavailable state while True Cache remains healthy.
4. Review the True Cache read TPS and the logical-connection status in the right-hand panel.
5. Select **Restore Primary**.
6. Wait for Primary and True Cache to return to **HEALTHY**.
7. Select **Next step**.

True Cache is a read-only replica. It can continue to answer eligible read-only requests from its cache while the Primary database is temporarily unavailable. Writes still require the Primary database.

![FastLab failover demo](images/fastlab-failover-demo.png " ")

## Semantic Cache Using Vector Search

1. Select a payment from the reference-payment list. The selected payment supplies the vector used for the search.
2. Choose an investigation:
   - **Find similar payments** compares payment profiles across the vector sample.
   - **Account behavior** restricts the candidates to the selected account.
   - **Cross-border risk** restricts the candidates to a different country.
   - **Recent activity** restricts the candidates to recent payments.
   - **Similar amount profile** finds payments with a similar amount and vector profile.
3. Read the explanation below the investigation selector. It describes the filter and the vector query used for the selected payment.
4. Expand **Behind this step** to see the actual SQL and the True Cache route for the request.
5. Review the result table. FastLab returns the five closest matching payments.
6. Read **Vector distance** as a similarity score: a smaller cosine distance means the payment profiles point in a more similar direction. The distance is not a currency amount or a percentage.
7. Try another reference payment or investigation and compare how the candidate filter changes the results.

This step connects the existing payment workflow to Oracle AI Vector Search. The vector represents useful payment attributes such as amount, account behavior, country, and transaction time. True Cache keeps frequently accessed vector data available in memory, allowing eligible semantic retrieval requests to be served close to the application while the Primary database remains the system of record.

![FastLab semantic cache using vector search](images/fastlab-vector-search.png " ")

## Completion

The FastLab is complete when:

- All three environment services show **HEALTHY**.
- The routing evidence identifies True Cache for read-only work.
- `ACCOUNTS` and `PAYMENTS`, including their selected indexes, show **KEPT**.
- Cache warmup completes and statistics are visible.
- Primary and True Cache read TPS and latency have been reviewed.
- Replication lag, cache hit ratios, and fetch latency have been reviewed.
- True Cache remains available while Primary is stopped, and Primary is restored afterward.
- At least one vector investigation returns five ranked payment results and its SQL is visible in **Behind this step**.

## Learn More

[Oracle True Cache documentation](https://docs.oracle.com/en/database/oracle/oracle-database/23/odbtc/using-oracle-true-cache-your-applications.html)

[Using Vectors with Oracle True Cache](https://blogs.oracle.com/database/using-vectors-with-oracle-true-cache)
