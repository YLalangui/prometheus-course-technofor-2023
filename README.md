# prometheus-course-technofor-2023

## Introduction
This project was developed for the course offered by TechnoFor, focusing on Prometheus. The core objective of this project is to establish an instance of Prometheus and Grafana to monitor a MySQL server. While the included Docker Compose file displays a variety of exporters and servers, serving as examples from the course, the primary deliverable centers on the MySQL server monitoring. This documentation dives into the server configurations and elaborates on the creation of dashboards and alerts.

## Files involved in MySQL monitoring
1. `docker-compose.yml`: This Docker Compose includes extra services we made during the course, but the key ones for this project are prometheus, grafana, mysql, and the mysql-exporter services. Some of these services come with config files, which we'll dive into in this section.
2. `prometheus/prometheus.yml`: The scrape configuration lists the endpoints to be scraped, with `mysql-exporter` as the key `job_name` for this specific deliverable. Note that we've set the target as `mysql:3306` in this configuration. This represents the server name (or container name in this context) and the port where our MySQL server runs.
