# Initialize Environment

## Introduction

In this lab, you will validate the pre-provisioned Oracle True Cache environment and choose how you want to run the workshop.

The DBW26 workshop supports two equal paths:

- **FastLab - Quick guided demo:** a visual command-center experience for a short guided demonstration.
- **Full LiveLab - Command-line workshop:** a documentation-style path where you copy commands into the terminal and see the database, Java, and container operations directly.

*Estimated Time:* 10 Minutes.

<if type="nonsandbox">
Watch the video for a quick walk through of the Lab1.
[Lab1](videohub:1_y0sporip)
</if>

### Objectives
- Validate that the Primary database, True Cache, and app containers are running.
- Open the Oracle True Cache LiveLab UI.
- Choose either the FastLab quick guided demo or the Full LiveLab command-line workshop.

### Prerequisites
This lab assumes you have:
- A Free Tier, Paid or LiveLabs Oracle Cloud account
- You have completed:
    - Lab: Prepare Setup(*Free-tier* and *Paid Tenants* only)
    - Lab: Environment Setup((*Free-tier* and *Paid Tenants* only))

## Task 1: Validate That Required Processes Are Up and Running.
0. If you are unable to launch the remote desktop, Click on “View Login Info” on top lefthand side and select Open Link under Terraform Outputs section as shown in the below image.
    ![terraform url](https://oracle-livelabs.github.io/database/truecache/initialize-environment/images/terraformurl.png " ")
1. Access your remote desktop session and validate your environment before you start the subsequent labs. The following processes should be up and running:

    - Oracle primary database container
    - Oracle True Cache container
    - Client app container

2. Click on Activities (shown on top left corner) >> Terminal icon (shown on the bottom of the screen which is next to Chrome icon) to Launch the Terminal and follow these steps to validate the services.

    ![activities_terminal_icon](images/activities_terminal_icon.png " ")

3. Log in to Podman and check for podman containers.

        ```
        <copy>
        sudo podman ps -a
        </copy>
        ```
    ![podman containers](https://oracle-livelabs.github.io/database/truecache/initialize-environment/images/truecache-podman.png " ")

4. If a container is not running, restart it using the following commands.

        ```
        <copy>
        sudo podman start prod truedb appclient
        </copy>
        ```

5. Verify that all containers are running.

        ```
        <copy>
        sudo podman ps --format 'table {{.Names}}\t{{.Status}}'
        </copy>
        ```

6. Wait until `prod` and `truedb` report **healthy**. The `appclient` container may show only its running status because it does not expose a database health check.

### Recover the FastLab UI

The FastLab web proxy runs on the host and listens on port `8080`; it is separate from the three Podman containers. If `http://localhost:8080` is refused or shows a blank page, check the listener from the LiveLabs desktop terminal:

        ```
        <copy>
        sudo ss -lntp | grep ':8080'
        </copy>
        ```

If no process is listening, restart the supplied proxy from the application directory:

        ```
        <copy>
        cd /home/opc/LiveLabs_clientapp_warmup_Jul2025-1
        sudo bash ./redeploy.sh --no-ui-build
        </copy>
        ```

Refresh the browser after the command completes. The proxy is not managed by `truecache-ui.service`, so `Unit truecache-ui.service does not exist` is expected on images that use this deployment layout.

### Recover stopped or unhealthy containers

If `prod`, `truedb`, or `appclient` is stopped, start the pre-provisioned containers and check their status again:

        ```
        <copy>
        sudo podman start prod truedb appclient
        sudo podman ps --format 'table {{.Names}}\t{{.Status}}'
        </copy>
        ```

Allow several minutes for the Oracle database containers to become healthy after a restart. Do not continue until `prod` and `truedb` report **healthy** and all three containers are running. If a named container is missing from `sudo podman ps -a`, the instance was not provisioned with the required lab environment and must be restored by the lab administrator.

## Task 2: Open the True Cache LiveLab UI

1. Open a browser in the remote desktop and navigate to the LiveLab UI URL provided for your environment.

2. The first screen asks you to choose how to run the lab.

    ![True Cache LiveLab experience chooser](images/truecache-experience-chooser.png " ")

3. Select one of the following:

    - **FastLab** for the quick guided demo.
    - **Full LiveLab** for the command-line workshop.

4. You can switch between modes later from the left navigation.

![Full LiveLab environment](images/full-livelab-environment.png " ")

## Task 3: Validate True Cache Services from the Terminal

1. Confirm the Primary service.

    ```
    <copy>
    sudo podman exec -it prod /bin/bash
    export ORACLE_SID=ORCLCDB
    sqlplus / as sysdba
    set pages 100 lines 180
    select database_role, open_mode from v$database;
    alter session set container=ORCLPDB1;
    select name, network_name from v$services where lower(name) like 'sales1%' order by name;
    exit
    exit
    </copy>
    ```

2. Confirm the True Cache service and database role.

    ```
    <copy>
    sudo podman exec -it truedb /bin/bash
    export ORACLE_SID=TRUEDB
    sqlplus / as sysdba
    set pages 100 lines 180
    select database_role, open_mode from v$database;
    alter session set container=ORCLPDB1;
    select name, network_name from v$services where lower(name) like 'sales1%' order by name;
    exit
    exit
    </copy>
    ```

3. Run the BasicApp routing proof.

    ```
    <copy>
    read -rsp 'Transactions password: ' TC_DB_PASSWORD; echo
    sudo podman exec -e TC_DB_PASSWORD="$TC_DB_PASSWORD" -it appclient /bin/bash
    cd /stage/clientapp/BasicApp
    /stage/jdk-17.0.6/bin/java -cp ojdbc8.jar:. TrueCache 172.20.1.2:1521/sales1 transactions "$TC_DB_PASSWORD"
    exit
    </copy>
    ```

The output should show the Primary role first, then the True Cache role on service `SALES1_TC`.

You may now proceed to the next lab.

## Acknowledgements
* **Authors** - Sambit Panda, Consulting Member of Technical Staff , Vivek Vishwanathan Software Developer, Oracle Database Product Management
* **Contributors** - Pankaj Chandiramani, Shefali Bhargava, Jyoti Verma, Ilam Siva
* **Last Updated By/Date** - Sambit Panda, Consulting Member of Technical Staff, June 2026
