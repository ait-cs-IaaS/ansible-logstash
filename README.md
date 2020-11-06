# Ansible installation

## Requirements
- ansible

## Defaults

The default config will start a logstash accepting input from beats and outputting
them to a elasticsearch instance.
```yaml
logstash_beats_port: 5044
logstash_elasticsearch_host: "http://localhost:9200"

logstash_config_files:
    - "{{ logstash_config_tmpl }}"
# override to use a different config template
logstash_config_tmpl: templates/logstash.conf.j2

logstash_inputs_config: |
    beats {
        port => {{ logstash_beats_port }}
    }

logstash_filters_config: ""

logstash_outputs_config: |
    elasticsearch {
        hosts => "{{ logstash_elasticsearch_host }}"
        index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    }
```
## Examples

Aminer configuration with beats and kafka input
```yaml
kafka_server_ip: "localhost"
kafka_server_port: 9092
kafka_topics: "aminer" 
input_beats_port: 5044
elasticsearch_host: "http://localhost:9200"

logstash_inputs_config: |
  kafka {
    bootstrap_servers => "{{ kafka_server_ip }}:{{ kafka_server_port }}"
    topics => "{{ kafka_topics }}"
    codec => "json"
  }

  beats {
    port => {{ input_beats_port }}
  }

logstash_filters_config: |
    if [LogData] {
        date {
        match => [ "[LogData][Timestamps][0]", "UNIX" ]
        target => "@logtimestamp"
        }
    }

    if [FromTime] {
        date {
            match => [ "[FromTime]", "UNIX" ]
            target => "@fromtimestamp"
        }
    }

    if [ToTime] {
        date {
            match => [ "[ToTime]", "UNIX" ]
            target => "@totimestamp"
        }
    }

logstash_outputs_config: |
    if [AnalysisComponent] {
        elasticsearch {
            hosts => {{ elasticsearch_host }}
            index => "aminer-anomaly-%{+YYYY.MM.dd}"
        }
    }

    if [StatusInfo] {
        elasticsearch {
            hosts => {{ elasticsearch_host }}
            index => "aminer-statusinfo-%{+YYYY.MM.dd}"
        }
    }

```

