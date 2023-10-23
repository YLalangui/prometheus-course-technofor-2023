# prometheus-course-technofor-2023

## Introduction
This project was developed for the course offered by TechnoFor, focusing on Prometheus. The core objective of this project is to establish an instance of Prometheus and Grafana to monitor a MySQL server. While the included Docker Compose file displays a variety of exporters and servers, serving as examples from the course, the primary deliverable centers on the MySQL server monitoring. This documentation dives into the server configurations and elaborates on the creation of dashboards and alerts.

## Files involved in MySQL monitoring
1. `docker-compose.yml`: This Docker Compose includes extra services we made during the course, but the key ones for this project are prometheus, grafana, mysql, and the mysql-exporter services. Some of these services come with config files, which we'll dive into in this section.
2. `prometheus/prometheus.yml`: The scrape configuration lists the endpoints to be scraped, with `mysql-exporter` as the key `job_name` for this specific deliverable. Note that we've set the target as `mysql:3306` in this configuration. This represents the server name (or container name in this context) and the port where our MySQL server runs.
3. `mysql-exporter/.my.cnf`: In the latest version of mysql-exporter by Prometheus, you no longer need to specify the MySQL server connection using environment variables. Instead, you'll require a .my-cfn file that contains the database username and password for our MySQL server.

## Prometheus/MySQL deliverable
To set up the environment with Prometheus, Grafana, MySQL, and mysql-exporter for this project, run:
````
$ docker-compose up
````
Running the command will kick off all the servers. You might notice the alertmanager server keeps restarting, but don't worry, it's not an issue for this deliverabl: We'll use Grafana to set up the alerts anyway :D. The restart hiccup is interesting, though. According to the official documentation for alertmanager v0.26.0 ([link](https://prometheus.io/docs/alerting/latest/configuration/#msteams_config:~:text=%3Ctmpl_string%3E%2C%20...%20%7D%20%5D-,%3Cmsteams_config%3E,-Microsoft%20Teams%20notifications)), there's an official config called 'msteams_config' designed to send alerts to an MS Teams group. Oddly enough, alertmanager flags an error, claiming 'msteams_config' isn't an official config. Yet, it's clearly listed in the official docs. We can ignore this issue ;)

Once all the containers are running, head over to 'localhost:9090' and check out status/targets. Ensure every server is active, with the exception of alertmanager. Now, let's dive into creating the dashboard and alerts for our MySQL server.

Head to localhost:3000 to access our Grafana server and set a new password. After updating the password, navigate to the left sidebar and select 'Connections'. Choose 'Prometheus' from the Data Sources and click 'Add new data source'. Just update the 'Prometheus server URL' to 'http://prometheus:9090' and click on 'Save & test'. From now on Grafana is using our prometheus server as data source.


Let's set up a Dashboard. To begin, select 'Dashboards' from the left menu and then click 'new'. Instead of building a dashboard from scratch, Grafana provides us with a vast selection of pre-configured dashboards developed by both Grafana itself and third-party contributors. Check them out here: [https://grafana.com/grafana/dashboards/]. Notably, there are several interesting dashboards created for MySQL. To utilize these dashboards, click on 'new' followed by 'Import'. Next, we'll need to input an ID to specify the desired dashboard. For this MySQL instance, use the ID "7362". Enter this number into the "Import via grafana.com" field and click 'Load'. In the following window, scroll down and select 'Prometheus' as the Data Source, then click 'Import'. This will display a dashboard filled with various metrics. However, it lacks alerts, which we'll set up next.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/80c9f0ad-f7f3-484b-8157-c86aef93b046)


You might notice that certain metrics draw from nodeexporter data, leading to "no data" metrics. By default, this MySQL exporter assumes the MySQL server is housed on an on-premise machine. However, in our project, all components run in containers. This makes installing nodeexporter within one of these containers challenging. Instead, we'll utilize the already active node exporter container. This approach gives us metrics like CPU, disk, and memory from the host machine running the MySQL server container. For each metric, we need to update the promql query by replacing the '$host' variable with 'nodeexporter:9100'. It is recomensable to change from 'Graph' plot (Deprecated) to 'Time Series' plot. as well. 

Let's set up an alarm to send notifications to an MS Teams group. To do this, we'll modify the 'MySQL Connection' graph. We'll measure the number of instances, which will default to three. Thus, the promql command should be `sum(mysql_global_status_threads_connected)`, which will display a graph indicating three MySQL instances. Once done, click 'Apply'. Next, navigate to the 'Alerting' section in the left menu and select 'contact points'. Create a new contact point and assign it a unique name. For integration, choose 'Microsoft Teams', then input the webhook URL of your team group and finally click on 'Save contact point'.

![image](https://github.com/YLalangui/prometheus-course-technofor-2023/assets/24701538/035b058c-1928-44a0-836c-5ab4c334356f)





