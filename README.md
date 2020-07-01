# alertmanager_detail_exporter #

Export Prometheus Alertmanager alerts as metrics for consumption back into
Prometheus.

Prometheus has the synthentic ALERTS timeseries, but that doesn't indicate
what's suppressed, silenced, etc., since that is handled by the Alertmanager.

This creates an ALERTS_DETAIL timeseries, with detailed alert information, including
alert_state, alert_silenced and alert_inhibited labels.

Main use case is a Grafana dashboard using the worldmap panel - I want to just
show active alerts and group them by the 'location' label I include.

Not using the Prometheus python client since it doesn't want do do a variable
number of labels for the same timeseries.

Having variable number of labels would normally risk a cardinality explosion,
but total number of alerts is fairly low.  Another contemplated approach is to
just have a whitelist of labels to export.

Pass 'filter' to filter the alerts, using the same '<label>="<value>"' syntax as
Alertmanager.

This is very much a POC/work in progress - feedback welcome.

## Prometheus config ##

```
# add detailed alert info back into Prometheus -
# adds alert state (firing, etc.), whether it's silenced and
# whether it's inhibited
#
# Need to filter the alerts to match our external labels as Thanos
# see them, otherwise we get duplicates - so 
# pass 'filter=dc="<dc>"' as an argument
#
  - job_name: alertmanager_detail
    scrape_interval: 10s
    honor_labels: true
    metrics_path: /alerts_detail
    params:
      filter: [dc=dc1]
    static_configs:
    - targets:
        - am1:9334
        - am2:9334
        - am3:9334
```