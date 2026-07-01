# apim-with-elk-moesif-demo

This repository contains a Docker Compose-based project that starts WSO2 APIM + ELK to demonstrate that events are sent to Moesif + ELK simultaneously.

## Configuration

### Requirements

- You will need a Moesif Account. You can follow these steps: [Set Up your Moesif Account](https://apim.docs.wso2.com/en/4.6.0/monitoring/api-analytics/moesif-analytics/moesif-integration-guide/#step-1-set-up-your-moesif-account)

- Docker compose.

- Clone this repository :-)

### Configuration steps

1. Insert your moesif key in the deployment.toml file [ROOT_REPO]/apim/deployment.toml

```yaml
[apim.analytics.properties]
"publisher.reporter.class" = "com.wso2.apim.analytics.CompositeMetricReporter"
moesifKey = "paste here your moesif key"
moesif_base_url = "https://api.moesif.net"
```

2. Ensure you are logged in the wso2 docker repository, if not, you can login using this command:

```bash
docker login registry.wso2.com
```
Enter your WSO2 account credentials.

If you want to use the free WSO2 docker image change this line in the **docker-compose.yaml** file:

```yaml
  wso2apim:
    image: registry.wso2.com/wso2-apim/am:4.6.0.34
    container_name: apim
```

to

```yaml
  wso2apim:
    image: wso2/wso2am:4.6.0
    container_name: apim
```

3. Start the environment

```bash
docker-compose up -d
```

4. Monitor the APIM logs to ensure it starts correctly: 

```bash
docker-compose logs -f wso2apim
```

5. Enter to the publisher console, create an API (You can use pizzashack), deploy, publish, create app, suscribe and consume APIs

<https://localhost:9443/publisher>


6. Verify that events are reaching Elasticsearch: 

```bash
curl http://localhost:9200/apim-analytics-*/_search?pretty | head -50
```

5. Kibana will be located at <http://localhost:5601>. Create a data view using the `apim-analytics-*` pattern to explore the events.
   1. Open http://localhost:5601
   2. Left sidebar menu → Stack Management (gear icon at the bottom)
   3. Kibana section → Data Views
   4. Click the Create data view button
   5. Fill in the following:

   - Name: apim-analytics

   - Index pattern: apim-analytics-*

   - Timestamp field: @timestamp
   6. Save the data view to Kibana

   To view the data:

   - Sidebar menu → Discover

   - Data view selector (top left) → select apim-analytics

   - Adjust the time range in the top right (default is the last 15 minutes)

   If the apim-analytics-* index does not appear in step 5, it is because no events have been received yet — first verify that APIM is generating the apim_metrics.log:
```bash
docker-compose exec wso2apim
tail -f /home/wso2carbon/wso2am-4.6.0/repository/logs/apim_metrics.log
````
