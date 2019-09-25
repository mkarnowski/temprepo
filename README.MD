# Avi Metrics Script

The Avi Metrics script was built with the intent to pull metrics from one or more Avi controllers and send these values to a centralized metrics database(s).
The script supports a number of different endpoints; current support includes AppDynamics, Datadog, Elasticsearch, Graphite, InfluxDB, Logstash, Splunk, and Wavefront.


This repository includes that necessary files to deploy a centralized metrics script
- **Requirements**
    - **python 3.6+**
    - **python3-yaml+**
    - **python3-requests**


- **Files**
    - **avimetrics.py**:  This is the script that will pull the values from the Avi Controller API and forward to the metrics enpoints
    - **metrics_endpoints.py**:  This file contains the methods for formatting the data and sending to the defined metrics endpoint(s)
    - **configuration_example.yaml**:  This is an example a <strong> required </strong> configuration.yaml file.  Further details around are found below.
    - **dockerfile**:  This file contains the commands to build a container  








# Installation
The following files are required and must exist within the same directory for successful metric gathering.
- **avimetrics.py**
- **configuration.yaml**
- **metrics_endpoints.py**




# Usage
## avimetrics.py

The metrics script will look in the local directory for the configuration.yaml file.  Using the values from this file the script will pull the relevant data from the Avi Controller API and forward the values to the defined metrics endpoints.

```sh
$ python3 avimetrics.py
```

# configuration.yaml

The configuration.yaml file will include the parameters to define what data values are desired for each Avi Controller cluster.  Within the configuration.yaml file, you will define controller(s) within a list under the key "controllers".  For each controller entry the following values are available:


 <table class="table table table-bordered table-hover">  
 <tbody>       
 <tr>   
 <th>Parameter
 </th>
 <th>Choices/<span style="color:#0000FF">Defaults</span>
 </th>
 <th>Comments
 </th>
 </tr>
 <tr>    
 <td><strong>controllers</strong><br>
 <span style="color:#810082">list</span>
     <table class="table table table-bordered table-hover">  
     <tbody> 
     <tr>   
     <td><strong>avi_cluster_name</strong><br>
     <span style="color:#810082">string</span> / <span style="color:green">required</span>
     </td>
     </tr>
    </tbody>
    </table>
 <td>
     <table class="table table table-bordered table-hover">  
     <tbody> 
     <tr>   
     <td><strong>avi_cluster_name</strong><br>
     <span style="color:#810082">string</span> / <span style="color:green">required</span>
     </td>
     </tr>
    </tbody>
    </table>
 </td>
 <td> List of Controllers and their relevant metrics configurations
 </td>
 </tr>
   </tbody>
 </table> 


 <tr>   
 </td>
 <td>Avi Vantage controller</td>
 <td>169.254.0.100</td>
 <td>tcp:80, tcp:443(GUI), tcp:5054, tcp:5098(ssh), tcp:8444, udp:161</td>
 </tr>
 <tr>    
 <td><strong>k8_(various)</strong></td>
 <td>Kubernetes VM</td>
 <td>various</td>
 <td>tcp:30000(GUI), tcp:8443</td>
 </tr>
 <tr>    
 <td><strong>kubeclient1</strong></td>
 <td>Client used for traffic generation</td>
 <td>169.254.8.8</td>
 <td>none</td>
 </tr>
  </tbody>
 </table>



## appdynamics_http.json

Define the AppDynamics Standalone Machine Agent IP address and the port that the HTTP listener is listening on

EXAMPLE:

```sh
{"appdynamics":
    {
	    "server":"169.254.0.1",
	    "server_port":8293
    }
}
```



## datadog.json

Define the URL and API key used to send metrics into Datadog

EXAMPLE:

```sh
{"datadog":
    {
        "api_url" : "app.datadoghq.com/api/v1/series?api_key=",
        "api_key" : "abcdefghijgklmnopqrstuvwxyz12345"
    }
}

```



## graphite_host.json

Define the grahite server host name/ip and the tcp port carbon cache is listening on

EXAMPLE:

```sh
{"graphite":
    {
	    "server":"127.0.0.1",
	    "server_port":2003
    }
}
```



## splunk_host.json

Define the values for sending values to Splunk HTTP Endpoint Collector.

EXAMPLE:

```sh
{"splunk_server":{
    "server":"169.254.0.1",
    "_comment":"HEC_PROTOCOL OPTIONS ARE HTTP OR HTTPS",
    "hec_protocol":"https",
    "hec_port": 8088,
    "hec_token":"abcdefgh-ijkl-mnop-qrst-uvwxyz123456",
    "_comment":"INDEX TYPE EVENT OR METRIC",
    "index_type":"event",
    "index":"avimetrics"
    }

}
```


## logstash.json

Define the values for sending metrics to a Logstash endpoint.  The script will send values in a format that is expecting the configured logstash codec to be json_lines.

EXAMPLE:

```sh
{"logstash":
    {
        "_comment": "logstash codec => json_lines",
        "server": "169.254.0.1",
        "server_port": 517,
        "_comment": "tcp or udp",
        "protocol": "udp"
    }
}

```


## elasticsearch.json

Define the values for sending metrics to Elasticsearch via the document API.

EXAMPLE:

```sh
{"elasticsearch":
    {
        "server": "169.254.0.1",
        "server_port": 9200,
        "protocol" : "https",
        "index": "avimetrics",
        "_comment":"default time filter field name",
        "timestamp": "@timestamp",
        "_comment":"If using auth on elasticsearch set auth-enabled to true and modify the credential values",
        "auth-enabled": true,
        "username": "admin",
        "password": "password"
    }
}

```


## influxdb.json

Define the values for sending metrics to InfluxDB via HTTP API.  The script will send values using the InfluxDB's Line Protocol format.

EXAMPLE:

```sh
{"influxdb":
    {
        "server": "169.254.0.1",
        "server_port": 8086,
        "protocol": "https",
        "db": "avi"
    }
}
```


# Run as a container
To run this script as a container, modify the files as exampled above prior to building.

### Build the container
```sh
$ docker build -t avimetrics .
```
### Start the container
To start the container it is required to specify the metrics endpoint via the <strong>EN_METRIC_ENDPOINT</strong> environment variable.  To specify multiple endpoint seprate each with a colon, ":".  

Add the environment variable <strong>EN_NOREALTIME=True</strong> to disable retrieving realtime metrics if they are enabled within the controller, results in 5 min averages.

The example below specifies running the container with sending data to multiple endpoints.
```sh
$ docker run -d -e "EN_METRIC_ENDPOINT=graphite:datadog:appdynamics_http" --name avimetrics --restart always --log-opt max-size=1m avimetrics
```

# Metrics Being Collected


### Controller Cluster Metrics

- Cluster node roles
- Network pool Usage
- License Usage %
- Days until license(s) expire
- Current Avi Vantage version
- Cluster Node Performance Metrics
    - controller_stats.avg_cpu_usage
    - controller_stats.avg_disk_usage
    - controller_stats.avg_mem_usage



### Service Engine Metrics:

- Virtual Server count per Service Engine
- Service Engine count
- Service Engine / Controller missed heartbeats
- Service Engine connected state
- Service Engine healthscore
- Service Engine Virtual Service hosted used capacity
- Statistics for each Service Engine
    - se_if.avg_bandwidth
    - se_stats.avg_connection_mem_usage
    - se_stats.avg_connections
    - se_stats.avg_connections_dropped
    - se_stats.avg_cpu_usage
    - se_stats.avg_disk1_usage
    - se_stats.avg_mem_usage
    - se_stats.avg_dynamic_mem_usage
    - se_stats.avg_persistent_table_usage
    - se_stats.avg_rx_bandwidth
    - se_if.avg_rx_bytes
    - se_if.avg_rx_pkts
    - se_if.avg_rx_pkts_dropped_non_vs
    - se_if.avg_tx_pkts
    - se_if.avg_tx_bytes
    - se_stats.avg_ssl_session_cache_usage
    - se_if.avg_connection_table_usage
    - se_stats.max_se_bandwidth
    - se_stats.avg_eth0_bandwidth
    - se_stats.pct_syn_cache_usage
    - se_stats.avg_packet_buffer_usage
    - se_stats.avg_packet_buffer_header_usage
    - se_stats.avg_packet_buffer_large_usage
    - se_stats.avg_packet_buffer_small_usage





### Virtual Service Stats

- Virtual Service healthscore
- Virtual Service operational status (Up/Down)
- Pool member(s) operational status (Up/Down)
- Which Service Engine the Virtual Service is hosted on
- Statistics for each Virtual Service; total and broken down per Service Engine
    - l4_server.avg_errored_connections
    - l4_server.avg_rx_pkts
    - l4_server.avg_bandwidth
    - l4_server.avg_open_conns
    - l4_server.avg_new_established_conns
    - l4_server.avg_pool_complete_conns
    - l4_server.apdexc
    - l4_server.avg_total_rtt
    - l4_client.apdexc
    - l4_client.avg_bandwidth
    - l4_client.avg_application_dos_attacks
    - l4_client.avg_complete_conns
    - l4_client.avg_connections_dropped
    - l4_client.avg_new_established_conns
    - l4_client.avg_policy_drops
    - l4_client.avg_rx_pkts
    - l4_client.avg_tx_pkts
    - l4_client.avg_rx_bytes
    - l4_client.avg_tx_bytes
    - l4_client.max_open_conns
    - l4_client.avg_lossy_connections
    - l7_client.avg_complete_responses
    - l7_client.avg_client_data_transfer_time
    - l7_client.avg_client_txn_latency
    - l7_client.sum_application_response_time
    - l7_client.avg_resp_4xx_avi_errors
    - l7_client.avg_resp_5xx_avi_errors
    - l7_client.avg_resp_4xx
    - l7_client.avg_resp_5xx
    - l4_client.avg_total_rtt
    - l7_server.avg_resp_latency
    - l7_server.apdexr
    - l7_client.avg_page_load_time
    - l7_client.apdexr
    - l7_client.avg_ssl_handshakes_new
    - l7_client.avg_ssl_connections
    - l7_server.avg_application_response_time
    - l7_server.pct_response_errors
    - l7_server.avg_frustrated_responses
    - l7_server.avg_total_requests
    - l7_client.sum_get_reqs
    - l7_client.sum_post_reqs
    - l7_client.sum_other_reqs
    - l7_client.avg_frustrated_responses
    - l7_client.avg_waf_attacks
    - l7_client.pct_waf_attacks
    - l7_client.sum_total_responses
    - l7_client.avg_waf_rejected
    - l7_client.avg_waf_evaluated
    - l7_client.avg_waf_matched
    - l7_client.avg_waf_disabled
    - l7_client.pct_waf_disabled
    - l7_client.avg_http_headers_count
    - l7_client.avg_http_headers_bytes
    - l7_client.pct_get_reqs
    - l7_client.pct_post_reqs
    - l7_client.avg_http_params_count
    - l7_client.avg_uri_length
    - l7_client.avg_post_bytes
    - dns_client.avg_complete_queries
    - dns_client.avg_domain_lookup_failures
    - dns_client.avg_tcp_queries
    - dns_client.avg_udp_queries
    - dns_client.avg_udp_passthrough_resp_time
    - dns_client.avg_unsupported_queries
    - dns_client.pct_errored_queries
    - dns_client.avg_domain_lookup_failures
    - dns_client.avg_avi_errors
    - dns_server.avg_complete_queries
    - dns_server.avg_errored_queries
    - dns_server.avg_tcp_queries
    - dns_server.avg_udp_queries









### Virtual Service Pool Server Metrics

- Statistics for each invidual Pool Server
    - l4_server.avg_rx_pkts
    - l4_server.avg_tx_pkts
    - l4_server.avg_rx_bytes
    - l4_server.avg_tx_bytes
    - l4_server.avg_bandwidth
    - l7_server.avg_complete_responses
    - l4_server.avg_new_established_conns
    - l4_server.avg_pool_open_conns
    - l4_server.avg_pool_complete_conns
    - l4_server.avg_open_conns
    - l4_server.max_open_conns
