### Linux Authentication Log Monitoring Pipeline with Logstash, Elasticsearch & Kibana (ELK)

## Project Overview

In a Security Operations Center (SOC), authentication logs are one of the most valuable sources of security telemetry. They provide visibility into user logins, failed authentication attempts, SSH activity, privilege escalation through sudo, and other security-related events that help analysts detect unauthorized access and account misuse.

This project demonstrates how to build a simple log ingestion pipeline that collects Linux authentication logs, parses them into structured fields using Logstash, stores them in Elasticsearch, and visualizes them in Kibana.

--------

## Objective

Build a centralized authentication log monitoring pipeline capable of:

* Collecting Linux authentication logs
* Parsing raw syslog events into structured data
* Storing searchable events in Elasticsearch
* Visualizing authentication activity in Kibana
* Provide structured authentication telemetry for SOC analysis
* Providing visibility for security investigations

------
## Lab Architecture
                    Ubuntu Linux Host 
                           │
                           │
                    /var/log/auth.log
                           │
                           ▼
            ┌─────────────────────────────┐
            │          Logstash           │
            │                             │
            │  • File Input               │
            │  • Grok Parsing             │
            │  • Date Normalization       │
            └──────────────┬──────────────┘
                           │
                           ▼
            ┌─────────────────────────────┐
            │       Elasticsearch         │
            │  auth-logs-YYYY.MM.dd       │
            └──────────────┬──────────────┘
                          │
                          ▼
            ┌─────────────────────────────┐
            │           Kibana            │
            │ Discover • Search • Analysis│
            └─────────────────────────────┘

---------

## Technologies Used
* Ubuntu Linux
* Logstash
* Elasticsearch
* Kibana
* Grok Filters
* Linux Authentication Logs
--------

## Authentication Log Source
Linux stores authentication events in: ``` /var/log/auth.log ```

This log contains events including:

* Successful logins
* Failed login attempts
* SSH authentication
* sudo commands
* User switching
* PAM authentication events
* Privilege escalation attempts

These logs are among the first data sources examined during incident investigations.

------
## Logstash Configuration
To configure Logstash, create a pipeline configuration file named auth.conf inside the Logstash pipeline configuration directory.
``` bash
cd /etc/logstash/conf.d

sudo nano auth.conf
```

The following sections build the complete Logstash pipeline. The input plugin ingests authentication logs from the Linux host, the filter stage parses and normalizes the events, and the output plugin forwards the structured data to Elasticsearch for indexing and analysis.

## 1. Input Configuration

The input plugin continuously monitors and ingests data from the system's authentication log.

```conf
input {
  file {
    path => "/var/log/auth.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
```

## Purpose
* Reads /var/log/auth.log
* Imports historical logs on first execution
* Monitors new events in real time
* Disables file position tracking for lab purposes

## 2. Filter Configuration
Linux authentication logs are initially stored as unstructured syslog messages. The Grok filter parses these messages into structured fields, while the Date filter converts the extracted timestamp into Logstash's standard @timestamp field for accurate time-based analysis.

```conf
 filter {
  grok {
    match => {
      "message" => "^%{TIMESTAMP_ISO8601:timestamp} %{NOTSPACE:[host][name]} %{WORD:program}(?:\[%{NUMBER:pid}\])?: %{GREEDYDATA:msg}"
    }
  }

  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }
}
  ```


## Grok Pattern Breakdown
|Pattern	| Extracted Field|
|---------| ---------------|
|%{TIMESTAMP_ISO8601}	| Timestamp|
|%{NOTSPACE:[host][name]}	| Hostname|
|%{WORD:program}	| Program name (e.g., sshd, sudo)|
|%{NUMBER:pid}	| Process ID|
|%{GREEDYDATA:msg}	| Authentication message|

## Example Parsed Event

After the Grok and Date filters are applied, a raw authentication log is transformed into structured fields that can be searched and analyzed in Elasticsearch.
| Field | Value |
|-------|-------|
| `@timestamp` | `2026-07-03T03:09:24.350Z` |
| `host.name` | `logstash` |
| `program` | `CRON` |
| `pid` | `1219` |
| `log.file.path` | `/var/log/auth.log` |
| `msg` | `pam_unix(cron:session): session closed for user ubuntu` |
| `_index` | `auth-logs-2026.07.03` |

This structured format enables analysts to quickly filter events by host, process, time, or message content, making authentication investigations significantly more efficient than searching through raw log files.

## 3. Output Configuration
After parsing and normalizing the authentication logs, the final step is to forward the structured events to Elasticsearch for indexing and analysis. The Elasticsearch output plugin securely sends each event to the local Elasticsearch instance, where daily indices are created for efficient storage, querying, and retention. Once indexed, the data can be explored in Kibana using Discover, dashboards, and visualizations.

```conf
output {
 elasticsearch {
   hosts => ["https://localhost:9200"]
   user => "elastic"
   password => "<REDACTED>"
   index => "auth-logs-%{+YYYY.MM.dd}"
   ssl_verification_mode => "none"
   ssl_enabled => true
 }
}
```
Daily indices are created using the format: ``` auth-logs-%{+YYYY.MM.dd} ```
This improves search performance and simplifies data retention.

## Complete Logstash Pipeline
After combining the input, filter, and output sections, the completed ``` auth.conf ``` file should resemble the following configuration. To recap, we are using the file plugin to read the auth.log file, filtering it with Grok and date, and sending the output to Elasticsearch on port 9200, where it will appear in the Kibana instance.

```conf
input {
  file {
    path => "/var/log/auth.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {

  grok {
    match => {
      "message" => "^%{TIMESTAMP_ISO8601:timestamp} %{NOTSPACE:[host][name]} %{WORD:program}(?:\[%{NUMBER:pid}\])?: %{GREEDYDATA:msg}"
    }
  }

  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }
}

output {

  elasticsearch {

  hosts => ["https://localhost:9200"]

   user => "elastic"

   password => "<REDACTED>"

   ssl_enabled => true

  ssl_verification_mode => "none"

  index => "auth-logs-%{+YYYY.MM.dd}"

  }

}
```
## Setting Permissions and Restarting Logstash

The configuration file is good to go, but before jumping into Kibana, we need to set the proper permissions for the logstash user so that it can read the auth.log file. Then restart Logstash and check its status.
```bash
sudo usermod -aG adm logstash

sudo systemctl restart logstash

sudo systemctl status logstash
```
After restarting, the service should report:
![https://github.com/KAIGAI-BRIAN/SOC-Analyst-Portfolio/blob/main/Linux%20Authentication%20Log%20Monitoring%20Pipeline%20with%20Logstash%2C%20Elasticsearch%20%26%20Kibana%20(ELK)/screenshots/permissions%26restarting%20logstash.png]

## Visualizing the Log Data
Once the configuration settings are ready, the permissions have been set, and Logstash has been restarted. The last thing to do is to hop into Kibana and take a look the logs. Set up a Data view to visualize the auth log index pattern (auth-logs-*).

1. Click the Kibana menu
2. Scroll down to Management
3. Select Stack Management
4. Click Data Views under Kibana
5. Choose Create data view

## Investigating Logs in Kibana

Using Kibana Discover allows authentication events to be searched and filtered.

Useful fields include:

@timestamp
host.name
program
pid
msg

Example searches:

program : sshd
program : sudo
msg : "Failed password"
msg : "Accepted password"

These filters allow analysts to quickly identify suspicious authentication activity.

## SOC Use Cases

Depending on the authentication events collected, this pipeline can support detection of:

* SSH brute-force attacks
* Successful SSH logins
* Failed authentication attempts
* Unauthorized sudo usage
* Privilege escalation
* Suspicious login behavior
* Account misuse
* Authentication anomalies
* User activity investigations

## Challenges Encountered

One issue encountered during deployment was that Logstash could not initially read /var/log/auth.log. The service runs under its own user account and did not have sufficient permissions to access the log file.

Adding the logstash user to the adm group and restarting the service resolved the issue.

This highlighted the importance of understanding Linux permissions when deploying log collection pipelines.
Lessons Learned

Through this project, I gained practical experience with:

* Building an ELK log ingestion pipeline
* Collecting Linux authentication logs
* Parsing unstructured syslog data with Grok
* Normalizing timestamps for Elasticsearch
* Managing Linux permissions for Logstash
* Creating searchable indices
* Investigating authentication events in Kibana
* Understanding how authentication telemetry supports SOC investigations
  
## Future Improvements

Potential enhancements include:

* Extracting usernames and source IP addresses from authentication events.
* Parsing sudo commands into individual fields
* Detecting brute-force attacks using threshold rules
* Creating Kibana dashboards for authentication activity
* Building visualizations for failed login trends
* Adding alerting with ElastAlert or Elastic Detection Rules
* Collecting logs from multiple Linux hosts
* Replacing direct file monitoring with Filebeat for scalable log shipping

## Conclusion

This project demonstrates the complete workflow of collecting, parsing, indexing, and visualizing Linux authentication logs using the ELK Stack. By transforming raw authentication events into structured, searchable data, the pipeline provides security analysts with the visibility needed to investigate login activity, detect unauthorized access attempts, and support incident response. It also reinforces core SOC concepts including log normalization, centralized logging, and security telemetry analysis.

## Skills Demonstrated
* Linux Administration
* Logstash Pipeline Configuration
* Grok Pattern Development
* Log Parsing
* Elasticsearch
* Kibana
* Linux Permissions
* Authentication Monitoring
* Security Telemetry
* Log Analysis
* SOC Fundamentals

