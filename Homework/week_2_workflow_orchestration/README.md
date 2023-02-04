## Week 2 Homework

The goal of this homework is to familiarise users with workflow orchestration and observation. 


## Question 1. Load January 2020 data

Using the `etl_web_to_gcs.py` flow that loads taxi data into GCS as a guide, create a flow that loads the green taxi CSV dataset for January 2020 into GCS and run it. Look at the logs to find out how many rows the dataset has.

How many rows does that dataset have?

```python flows/02_gcp/etl_web_to_gcs.py```

Note: need to modify the `clean` function in `etl_web_to_gcs.py`:
```
    df["lpep_pickup_datetime"] = pd.to_datetime(df["lpep_pickup_datetime"])
    df["lpep_dropoff_datetime"] = pd.to_datetime(df["lpep_dropoff_datetime"])
```
and update parameters:
```
    color = "green"
    year = 2020
    month = 1
```

## Question 2. Scheduling with Cron

Cron is a common scheduling specification for workflows. 

Using the flow in `etl_web_to_gcs.py`, create a deployment to run on the first of every month at 5am UTC. What’s the cron schedule for that?

```
prefect deployment build flows/02_gcp/etl_web_to_gcs.py:etl_web_to_gcs -n "ETL web to GCS" --cron "0 5 1 * *"
prefect deployment apply etl_web_to_gcs-deployment.yaml
```


## Question 3. Loading data to BigQuery 

Using `etl_gcs_to_bq.py` as a starting point, modify the script for extracting data from GCS and loading it into BigQuery. This new script should not fill or remove rows with missing values. (The script is really just doing the E and L parts of ETL).

The main flow should print the total number of rows processed by the script. Set the flow decorator to log the print statement.

Parametrize the entrypoint flow to accept a list of months, a year, and a taxi color. 

Make any other necessary changes to the code for it to function as required.

Create a deployment for this flow to run in a local subprocess with local flow code storage (the defaults).

Make sure you have the parquet data files for Yellow taxi data for Feb. 2019 and March 2019 loaded in GCS. Run your deployment to append this data to your BiqQuery table. How many rows did your flow code process?

```
python flows/03_deployments/parameterized_flow.py   # color = 'yellow', months = [2, 3], year = 2019
prefect deployment build flows/02_gcp/etl_gcs_to_bq.py:etl_parent_flow -n "GCS to BQ"
prefect deployment apply etl_parent_flow-deployment.yaml
prefect agent start --work-queue "default"
# Go to the Orion UI and click 'Quick run" on the deployment
```

Logs:
```
Number of rows processed: 14851920
06:07:25 PM
Finished in state Completed('All states completed.')
06:07:25 PM
```


## Question 4. Github Storage Block

Using the `web_to_gcs` script from the videos as a guide, you want to store your flow code in a GitHub repository for collaboration with your team. Prefect can look in the GitHub repo to find your flow code and read it. Create a GitHub storage block from the UI or in Python code and use that in your Deployment instead of storing your flow code locally or baking your flow code into a Docker image. 

Note that you will have to push your code to GitHub, Prefect will not push it for you.

Run your deployment in a local subprocess (the default if you don’t specify an infrastructure). Use the Green taxi data for the month of November 2020.

How many rows were processed by the script?

```
# Run from the root of the repository
prefect deployment build -sb github/zoom-homework ./Homework/week_2_workflow_orchestration/etl_web_to_gcs_github.py:etl_web_to_gcs -n "Web to GCS - GitHub"
prefect deployment apply etl_web_to_gcs-deployment.yaml
prefect agent start --work-queue "default"
# Go to the Orion UI and click 'Quick run" on the deployment
```


## Question 5. Email or Slack notifications

Q5. It’s often helpful to be notified when something with your dataflow doesn’t work as planned. Choose one of the options below for creating email or slack notifications.

The hosted Prefect Cloud lets you avoid running your own server and has Automations that allow you to get notifications when certain events occur or don’t occur. 

Create a free forever Prefect Cloud account at app.prefect.cloud and connect your workspace to it following the steps in the UI when you sign up. 

Set up an Automation that will send yourself an email when a flow run completes. Run the deployment used in Q4 for the Green taxi data for April 2019. Check your email to see the notification.

Alternatively, use a Prefect Cloud Automation or a self-hosted Orion server Notification to get notifications in a Slack workspace via an incoming webhook. 

Join my temporary Slack workspace with [this link](https://join.slack.com/t/temp-notify/shared_invite/zt-1odklt4wh-hH~b89HN8MjMrPGEaOlxIw). 400 people can use this link and it expires in 90 days. 

In the Prefect Cloud UI create an [Automation](https://docs.prefect.io/ui/automations) or in the Prefect Orion UI create a [Notification](https://docs.prefect.io/ui/notifications/) to send a Slack message when a flow run enters a Completed state. Here is the Webhook URL to use: https://hooks.slack.com/services/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

Test the functionality.

Alternatively, you can grab the webhook URL from your own Slack workspace and Slack App that you create. 


How many rows were processed by the script?

```
prefect deployment build etl_web_to_gcs_github.py:etl_web_to_gcs -n "Web to GCS - Email"
prefect deployment apply etl_web_to_gcs-deployment.yaml     # parameters: {'color': 'green', 'year': 2019, 'month': 4}
prefect agent start --work-queue "default"
# Go to the Prefect Cloud UI, add Email block, create Automation for email notification, and click 'Quick run" on the deployment
```

Received email:
```
Flow run etl-web-to-gcs/delightful-mayfly entered state `Completed` at 2023-02-04T11:30:26.680937+00:00.
Flow ID: e7617aa1-d09a-47f1-96c5-7ee1ab9b65fd
Flow run ID: 5f2e35d1-d281-4751-afc1-09d313a61632
Flow run URL: https://app.prefect.cloud/account/d334efca-9642-45e7-8271-825168c2563a/workspace/b1f20e7e-ac96-4a8e-8a49-cf62f2bc5e3e/flow-runs/flow-run/5f2e35d1-d281-4751-afc1-09d313a61632
State message: All states completed.
```


## Question 6. Secrets

Prefect Secret blocks provide secure, encrypted storage in the database and obfuscation in the UI. Create a secret block in the UI that stores a fake 10-digit password to connect to a third-party service. Once you’ve created your block in the UI, how many characters are shown as asterisks (*) on the next page of the UI?

```
# Create Secret block from Prefect Cloud UI
```


## Submitting the solutions

* Form for submitting: https://forms.gle/PY8mBEGXJ1RvmTM97
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 6 February (Monday), 22:00 CET


## Solution

We will publish the solution here
