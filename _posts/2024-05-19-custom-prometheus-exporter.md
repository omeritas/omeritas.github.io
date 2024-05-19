---
title: "Creating a Custom Prometheus Exporter (using Bash)"
date: 2024-05-19 17:34:00 +/-TTTT
---

One of the key features of Prometheus is the ability to create custom exporters to expose metrics from various sources. While most popular applications already have an existing prometheus exporter, you may face applications that do not, especially when it's an application in the company you're working at.

There are better choices than Bash for writing Prometheus exporters. Golang is a popular one, because of its Prometheus library. It's also easier to write it in the language the application is written in, but for simplicity and illustration purposes we will use Bash in this tutorial.

## Create the exporter script

To start, we will write a Bash script to collect memory usage metrics. Create a file named 'exporter.sh' with the following code:

```bash
#!/bin/bash

# get the memory usage in percentage
get_memory_usage() {
  mem_total=$(free | grep Mem | awk '{print $2}')
  mem_used=$(free | grep Mem | awk '{print $3}')
  mem_usage=$(echo "scale=2; $mem_used/$mem_total*100" | bc)
  echo "memory_usage $mem_usage"
}

# collect the metrics
collect_metrics() {
  echo "# HELP memory_usage Memory usage percentage."
  echo "# TYPE memory_usage gauge"
  get_memory_usage
}

# start http server using socat
while true; do
  { echo -e "HTTP/1.1 200 OK\nContent-Type: text/plain\n\n$(collect_metrics)"; } | socat TCP-LISTEN:9100,reuseaddr,fork -
done
```

In this script we will use the memory usage on the Linux host machine as an example for a metric. 

In the first function **get_memory_usage** we extract the total memory in kb and the used memory in kb, which we use to calculate the percentage of memory used. 

After that we have function **collect_metrics**  to collect this metric. The first line provides a description of the memory_usage metric (with # HELP) and in the following line we define the type of the metric as a gauge, which is used for metrics that can go up and down (like memory usage). Then we call the get_memory_usage function to generate this metric.

Finally, we start a HTTP server with socat, but you can also use other commands like **nc**.


Make the script executable and run it with the following command:
```bash
chmod +x exporter.sh && ./exporter.sh
```

## Configure Prometheus
Now we need to configure Prometheus to scrape the metrics from our custom exporter. Edit your existing Prometheus configuration file ('prometheus.yml') to add a new scrape job. This tutorial assumes you already set Prometheus up:
```
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: 'bash_process_exporter'
    metrics_path: '/'
    static_configs:
      - targets: ['localhost:9100']
```

Start Prometheus by using the following command:
```bash
./prometheus --config.file=prometheus.yml'
```

Now, when you visit localhost:9100 you should see similar output. This indicates that the memory usage on my system is 22%. Your output will most likely differ from mine:
```bash
# HELP memory_usage Memory usage percentage.
# TYPE memory_usage gauge
memory_usage 22.00
```

Now you have succesfully created a custom Prometheus exporter using Bash.


