# Elasticsearch, Logstash, Kibana (ELK) Docker image with x-pack

### USAGE

1. docker build -t crack/elk .

2. docker run --rm -d -p 5601:5601 -p 9200:9200 -p 5044:5044 -v /home/elk-crack-data:/var/lib/elasticsearch --name elkc crack/elk

3. visit http://elk:5601/app/kibana#/dev_tools/console?_g=()
   
   POST _xpack/license
{
  "license": {
    "uid": "b5dccd00-b476-4b69-8763-cf22fc136822",
    "type": "platinum",
    "issue_date_in_millis": 1541462400000,
    "expiry_date_in_millis": 2855980923000,
    "max_nodes": 100,
    "issued_to": "li qq (dahe)",
    "issuer": "Web Form",
    "signature": "AAAAAwAAAA0ys5CCV21m9ZmWGtrlAAABmC9ZN0hjZDBGYnVyRXpCOW5Bb3FjZDAxOWpSbTVoMVZwUzRxVk1PSmkxaktJRVl5MUYvUWh3bHZVUTllbXNPbzBUemtnbWpBbmlWRmRZb25KNFlBR2x0TXc2K2p1Y1VtMG1UQU9TRGZVSGRwaEJGUjE3bXd3LzRqZ05iLzRteWFNekdxRGpIYlFwYkJiNUs0U1hTVlJKNVlXekMrSlVUdFIvV0FNeWdOYnlESDc3MWhlY3hSQmdKSjJ2ZTcvYlBFOHhPQlV3ZHdDQ0tHcG5uOElCaDJ4K1hob29xSG85N0kvTWV3THhlQk9NL01VMFRjNDZpZEVXeUtUMXIyMlIveFpJUkk2WUdveEZaME9XWitGUi9WNTZVQW1FMG1DenhZU0ZmeXlZakVEMjZFT2NvOWxpZGlqVmlHNC8rWVVUYzMwRGVySHpIdURzKzFiRDl4TmM1TUp2VTBOUlJZUlAyV0ZVL2kvVk10L0NsbXNFYVZwT3NSU082dFNNa2prQ0ZsclZ4NTltbU1CVE5lR09Bck93V2J1Y3c9PQAAAQAC/v5fLuwsjo419HowJT/4JK4a9/O0ejYR6gvwR2Ss0EgfGkLawQCiMyW6Z0hZtGRgWLZ4/kDL9JBRBD0Mc4KL+uHty8keBDLC+vzHmEEBezI/AWiRyDTt1yq7F9X8GsbgnzkjccnmnkSopHxa3AQfKPjNm4uFdV01d5ud3gZy+vRfY43RsB0QIyC9E/LTBb/+M6gTYWwzuck+InNK3XPK5zU1ouMnWCnFHb97pPNIA6MiNG2x4ekekTA5aJtd7q+9BGhP7srFK6Itywcrh2uJ1bn2yFF8QSRWmP7z3BD+ddBdp4y1Sn597ld61zh2G9rF1sct2nYKeWh4Ctm11tvu",
    "start_date_in_millis": 1541462400000
  }
}

4. docker exec -it /bin/bash 
   
   /opt/elasticsearch/bin/elasticsearch-setup-passwords interactive
   
   enter passwords: gogogo
   
5. restart elk as step 2

6. visit http://elk:5601/app/kibana#/management/logstash/pipelines/?_g=()
   
   create pipeline (id: main) .i.g 
   
   input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/etc/pki/tls/certs/logstash-beats.crt"
    ssl_key => "/etc/pki/tls/private/logstash-beats.key"
  }
}
filter {
    grok {
        match => { "message" => "\(%{TIMESTAMP_ISO8601:time}\)\[%{LOGLEVEL:log_level}\]\[%{NUMBER:pid}-%{WORD:tid}\](?<msg>(.*))\[(?<file>(.*))\]:%{NUMBER:line}" }
        remove_field => ["offset", "message", "tags", "input", "prospector", "beat", "host", "@version"]
    }
    date {
        match => ["time", "yyyy-MM-dd HH:mm:ss.SSS"]
        remove_field => ["time"]
        timezone => "Asia/Shanghai"
    }
    mutate {
        add_field => { "[@metadata][index]" => "filebeat" }
    }
}
output {
    elasticsearch {
        hosts => ["localhost"]
        manage_template => false
        index => "%{[@metadata][index]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
        user => "elastic"
        password => "gogogo"
  }
}
   
7. done
