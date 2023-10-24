# prometheus-course-technofor-2023

## Introduction
This project was developed for the course offered by TechnoFor, focusing on Prometheus. 

The primary goal of this project is divided into three main tasks:

1. Initial Setup: Establish an instance of both Prometheus and Grafana to monitor a single MySQL server.

2. Expansion & Alerts: Add two more MySQL servers. During this phase, we will also set up alerts. For instance, we'll monitor the total count of servers or compare the current number of used connections to the maximum allowed connections.

3. Federation Implementation: In this task, we will implement federation by introducing two additional Prometheus-MySQL pairs. The same Prometheus server from the first two tasks will be used to monitor these newly added MySQL servers via their respective Prometheus servers (Federation).


While the included Docker Compose file displays a variety of exporters and servers, serving as examples from the course, the primary deliverable centers on the MySQL server monitoring. This documentation dives into the server configurations and elaborates on the creation of dashboards and alerts.

## Files involved in MySQL monitoring
1. `docker-compose.yml`: This Docker Compose includes extra services we made during the course, but the key ones for this project are prometheus, grafana, three mysql servers, the mysql-exporter services and `federated-*` services. Some of these services come with config files, which we'll dive into in this section.
2. `prometheus/prometheus.yml`: The scrape configuration lists the endpoints to be scraped, with `mysql-exporter` as the key `job_name` for the second task. Note that we've set the target as `mysql-1:3306`, `mysql-2:3306` and `mysql-3:3306` in this configuration to simulate three MySQL servers. This represents the server name (or container name in this context) and the port where our MySQL servers runs. This approach, concerning MySQL and mysql-exporter, will be implemented in the third task focusing on federation.
3. `mysql-exporter/.my.cnf`: In the latest version of mysql-exporter by Prometheus, you no longer need to specify the MySQL server connection using environment variables. Instead, you'll require a .my.cfn file that contains the database username and password for our MySQL server. The .my.cnf file will remain consistent across all MySQL instances for this project.
4. `federated-prometheus-1` & `federated-prometheus-2`: These configuration files are for the third task focused on federation for the Prometheus services monitoring the MySQL servers. Their structure is similar to that of `prometheus/prometheus.yml`.

## Prometheus/MySQL deliverable
To set up the environment with Prometheus, Grafana, three MySQL servers, and mysql-exporter for this project, run:
````
$ docker-compose up
````
Running the command will kick off all the servers. You might notice the alertmanager server keeps restarting, but don't worry, it's not an issue for this deliverabl: We'll use Grafana to set up the alerts anyway :D. The restart hiccup is interesting, though. According to the official documentation for alertmanager v0.26.0 ([link](https://prometheus.io/docs/alerting/latest/configuration/#msteams_config:~:text=%3Ctmpl_string%3E%2C%20...%20%7D%20%5D-,%3Cmsteams_config%3E,-Microsoft%20Teams%20notifications)), there's an official config called 'msteams_config' designed to send alerts to an MS Teams group. Oddly enough, alertmanager flags an error, claiming 'msteams_config' isn't an official config. Yet, it's clearly listed in the official docs. We can ignore this issue ;)

Once all the containers are running, head over to 'localhost:9090' and check out status/targets. Ensure every server is active, with the exception of alertmanager. Now, let's dive into creating the dashboard and alerts for our MySQL server.

Head to localhost:3000 to access our Grafana server and set a new password. After updating the password, navigate to the left sidebar and select 'Connections'. Choose 'Prometheus' from the Data Sources and click 'Add new data source'. Just update the 'Prometheus server URL' to 'http://prometheus:9090' and click on 'Save & test'. From now on Grafana is using our prometheus server as data source.


Let's set up a Dashboard. To begin, select 'Dashboards' from the left menu and then click 'new'. Instead of building a dashboard from scratch, Grafana provides us with a vast selection of pre-configured dashboards developed by both Grafana itself and third-party contributors. Check them out here: [https://grafana.com/grafana/dashboards/]. Notably, there are several interesting dashboards created for MySQL. To utilize these dashboards, click on 'new' followed by 'Import'. Next, we'll need to input an ID to specify the desired dashboard. For this MySQL instance, use the ID "7362". Enter this number into the "Import via grafana.com" field and click 'Load'. In the following window, scroll down and select 'Prometheus' as the Data Source, then click 'Import'. This will display a dashboard filled with various metrics. However, it lacks alerts, which we'll set up next.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/80c9f0ad-f7f3-484b-8157-c86aef93b046)


You might notice that certain metrics draw from node-exporter data, leading to "no data" metrics. By default, this MySQL exporter assumes the MySQL server is housed on an on-premise machine. However, in our project, all components run in containers. This makes installing node-exporter within one of these containers challenging. Instead, we'll utilize the already active node exporter container. This approach gives us metrics like CPU, disk, and memory from the host machine running the MySQL server container. For each metric, we need to update the promql query by replacing the '$host' variable with 'node-exporter:9100'. It is recomensable to change from 'Graph' plot (Deprecated) to 'Time Series' plot. as well. 

Let's set up an alarm to send notifications to an MS Teams group. To do this, we'll modify the 'MySQL Connection' graph. We'll measure the number of instances, which will default to three. Thus, the promql command should be `sum(mysql_global_status_threads_connected)`, which will display a graph indicating three MySQL instances. Once done, click 'Apply'. Next, navigate to the 'Alerting' section in the left menu and select 'contact points'. Create a new contact point and assign it a unique name. For integration, choose 'Microsoft Teams', then input the webhook URL of your team group and finally click on 'Save contact point'.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/035b058c-1928-44a0-836c-5ab4c334356f)

Next, return to the dashboard and modify the metric we adjusted earlier, this time to introduce an alert rule. The rule will be activated when the number of servers drops below 3. In point 3 (Set alert evaluation behavior), create a folder and an evaluation group. Once done, click on 'save rule and exit'.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/9177b673-9242-41e7-8d70-71640f538a64)

Proceed to the 'Alerting' section in the left menu and select 'Notification policies'. Modify the existing policy and, for the 'Default contact point', choose our previously created contact point. Adjust any desired 'Timing options' and then finalize by clicking on 'Update default policy'.


![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/571ceee0-5c79-41d1-a091-e836f290304a)

After everything is set up, we can shut down one MySQL server to simulate a server outage. Given the dashboard's configuration, we may need to wait up to 5 minutes to observe Grafana triggering an alert and sending messages to our MS Teams group.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/81a29fac-2292-4644-950d-da25dd3b958c)

An interesting metrics could be monitoring the number of currently used connections in relation to the maximum allowed, especially if your application sees spikes in traffic. If the number of connections nears the maximum, you may face issues with new connections being denied.

To proceed, we'll create a new visualization using the PromQL query: (mysql_global_status_threads_connected / mysql_global_variables_max_connections) * 100. After setting up the visualization, establish an alert in the manner we've done before, and set it to activate when the value surpasses 80%.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/db74723e-64d9-4c90-bbbc-3705b6701f88)

In this instance, the alert won't be activated since this isn't a production MySQL server, and we're unlikely to exceed 80%. Nevertheless, creating a new visualization and establishing a beneficial alert is worthwhile.

