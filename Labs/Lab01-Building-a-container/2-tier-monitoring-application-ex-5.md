# Exercise 5 - Deploying a 2 Tier Monitoring Application

In this lab, you'll create a 2 tier water monitoring application using Grafana and InfluxDB.

## Scenario

The National Oceanic and Atmospheric Administration’s (NOAA) Center for Operational Oceanographic Products and Services has asked you to modernise their application stack to be able to monitor and analyse water data in two of their key stations - Santa Monica and Coyote Creek. They currently use a spreadsheet that records a series of metrics for things like water temperature, PH levels and water depth. The NOAA want to use a more Cloud Native approach and run some new software in containers and they have opted for a Grafana + InfluxDB stack to achieve this.

Your task is to create a Proof of Concept to show how NOAA can use Grafana and InfluxDB to display a dashboard based on some sample time series data they will provide.

## Components

Use the following specification for each component

**Container Network**
- Create a new network that all the application containers will use.

**InfluxDB**
- Use the container image `influxdb:1.7.10`
- Map the HOST port 8086 to the CONTAINER port 8086
- Use a local directory on your Workstation VM, mounted to `/var/lib/influxdb` in the InfluxDB container
- Download the sample data to your local machine using `curl https://s3.amazonaws.com/noaa.water-database/NOAA_data.txt -o NOAA_data.txt`. This should be mounted and accessible from within the container

Load the sample data into the InfluxDB database which you can do by executing the following command. Remember to change the `--path` variable to point to location where the NOAA_data.txt resides within the container.

```
influxdb influx -import -path=/path/to/NOAA_data.txt -precision=s -database=NOAA_water_database
```

**Grafana**
- Use the container image `grafana/grafana:7.0.0`
- Map the HOST port 3000 to the CONTAINER port 3000
- Use a local directory on your Workstation VM, mounted to `/var/lib/grafana` in the Grafana container
- The data source within Grafana will need to be set up when the Grafana server is running. To do this, go to Settings > Configuration > Data Sources
- Import the Grafana dashboard, located [here](resources/NOAA/grafana-dashboard.json). You can do this by selecting the + icon > import. Once the dashboard is imported, set the Absolute Time Range from 16/08/2019 to 17/09/2019.

Lab complete.






### Data sources and things to note

The sample data is publicly available data from the National Oceanic and Atmospheric Administration’s (NOAA) Center for Operational Oceanographic Products and Services. The data include 15,258 observations of water levels (ft) collected every six minutes at two stations (Santa Monica, CA (ID 9410840) and Coyote Creek, CA (ID 9414575)) over the period from August 18, 2015 through September 18, 2015.

