# website-monitoring

![web-1](screenshots/website-monitoring_1.png)

Monitore your websites availability, http status code (current and history), certificate, redirects and more with

* [Prometheus](https://github.com/prometheus/prometheus)
* [Prometheus blackbox exporter](https://github.com/prometheus/blackbox_exporter)
* [Grafana](https://github.com/grafana/grafana)

## Deploying

### Kubernetes

- Dependencies

  * [kubernetes](https://kubernetes.io/)
  * [kubectl](https://kubernetes.io/docs/tasks/tools/)

- Usage

  * `git clone https://github.com/cloudpilot-ai/website-monitoring.git && cd website-monitoring`
  * Edit `sample/prometheus-targets-cm.yaml` (see targets.yml.example) modify or add `baidu.com` as your desired service address
  * Create Namespace `kubectl create namespace prometheus`
  * Starting services `kubectl apply -n prometheus -f sample`
  * Visualize dashboards

    - Temporary access

      * `kubectl -n prometheus port-forward service/grafana 3000:3000`
      * [Visualize dashboards](http://localhost:3000)

    - Long-term access

      `kubectl -n prometheus get svc grafana` get request address http://192.168.49.2:3000/
      ```bash
      # kubectl -n prometheus get svc grafana
      NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
      grafana   ClusterIP   10.105.212.73   192.168.49.2  3000/TCP   9m51s
      ```

### Docker compose

- Dependencies

  * [docker](https://docs.docker.com/install/)
  * [docker-composer](https://docs.docker.com/compose/install/)

- Usage

  * `git clone https://github.com/cloudpilot-ai/website-monitoring.git && cd website-monitoring`
  * Edit `config/prometheus/targets.yml` (see targets.yml.example) or use `./gen_target.sh website-1.tld website-2.tld ...`
  * Create and start containers `docker-compose up -d`
  * [Visualize dashboards](http://localhost:3000/)

If you already have Prometheus and Prometheus blackbox exporter up and running just import the dashboards ([website-monitoring](dashboards/website-monitoring.json) or [overview](dashboards/overview.json)) and use the right [datasource](screenshots/import.png) and [jobs](screenshots/import.png) (http_job and icmp_job)

## Dashboards

###  Website monitoring

* HTTP status code
* HTTP redirects
* HTTP version
* TLS version
* Certificate validity
* ICMP
* DNS lookup time
* Availability over the last 24 hours, 3 days and 7 days
* Probe duration and status code history

![web-2](screenshots/website-monitoring_2.png)
![web-3](screenshots/website-monitoring_3.png)

### Overview

* Total number of targets
* Percentage of HTTP 200 status code
* Percentage of targets using SSL
* Global invalid status code history

![overview](screenshots/overview_1.png)

## Tips and tricks

### PromQL

Some useful PromQL queries

* Number of days till certificate expiration
  * `(probe_ssl_earliest_cert_expiry{instance=~"$target",job="$http_job"} - time()) / (60*60*24)`
* Display bad HTTP status code
  * `probe_http_status_code{job="$http_job",instance=~"$target"} != 200`
* Count the number of each status code
  * `count_values("code", probe_http_status_code)`
* Percentage of HTTP 200
  * `((count(count by (instance) (probe_http_status_code == 200))) / (count(count by (instance) (probe_http_status_code)))) * 100`

### Misc

- Deployed on kuberneres
  * Request blackbox exporter

    - Temporary access

      * `kubectl -n prometheus port-forward service/blackbox-exporter 9115:9115`
      * `curl -s "localhost:9115/probe?module=http_2xx&target=target.tld"`

    - Long-term access

      * `kubectl -n prometheus get svc blackbox-exporter` get request address
        ```bash
        # kubectl -n prometheus get svc blackbox-exporter
        NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
        blackbox-exporter   ClusterIP   10.97.216.40   192.168.49.2  9115/TCP   6m20s
        ```
      * `curl -s "192.168.49.2:9115/probe?module=http_2xx&target=target.tld"`

- Deployed on docker compose

  * Request blackbox exporter
    * `curl -s "localhost:9115/probe?module=http_2xx&target=target.tld"`
