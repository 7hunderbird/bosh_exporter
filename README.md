# BOSH Prometheus Exporter [![Build Status](https://travis-ci.org/cloudfoundry-community/bosh_exporter.png)](https://travis-ci.org/cloudfoundry-community/bosh_exporter)

A [Prometheus][prometheus] exporter for [BOSH][bosh] metrics. Please refer to the [FAQ][faq] for general questions about this exporter.

## Architecture overview

![](https://cdn.rawgit.com/cloudfoundry-community/bosh_exporter/master/architecture/architecture.svg)

## Installation

### Binaries

Download the already existing [binaries][binaries] for your platform:

```bash
$ ./bosh_exporter <flags>
```

### From source

Using the standard `go install` (you must have [Go][golang] already installed in your local machine):

```bash
$ go install github.com/cloudfoundry-community/bosh_exporter
$ bosh_exporter <flags>
```

### Cloud Foundry

The exporter can be deployed to an already existing [Cloud Foundry][cloudfoundry] environment:

```bash
$ git clone https://github.com/cloudfoundry-community/bosh_exporter.git
$ cd bosh_exporter
```

Modify the included [application manifest file][manifest] to include your BOSH properties. Then you can push the exporter to your Cloud Foundry environment:

```bash
$ cf push
```

### BOSH

This exporter can be deployed using the [Prometheus BOSH Release][prometheus-boshrelease].

## Usage

### Flags

| Flag / Environment Variable | Required | Default | Description |
| --------------------------- | -------- | ------- | ----------- |
| `bosh.url`<br />`BOSH_EXPORTER_BOSH_URL` | Yes | | BOSH URL |
| `bosh.username`<br />`BOSH_EXPORTER_BOSH_USERNAME` | *[1]* | | BOSH Username |
| `bosh.password`<br />`BOSH_EXPORTER_BOSH_PASSWORD` | *[1]* | | BOSH Password |
| `bosh.uaa.url`<br />`BOSH_EXPORTER_BOSH_UAA_URL` | *[1]* | | BOSH UAA URL |
| `bosh.uaa.client-id`<br />`BOSH_EXPORTER_BOSH_UAA_CLIENT_ID` | *[1]* | | BOSH UAA Client ID |
| `bosh.uaa.client-secret`<br />`BOSH_EXPORTER_BOSH_UAA_CLIENT_SECRET` | *[1]* | | BOSH UAA Client Secret |
| `bosh.log-level`<br />`BOSH_EXPORTER_BOSH_LOG_LEVEL` | No | `ERROR` | BOSH Log Level (`DEBUG`, `INFO`, `WARN`, `ERROR`, `NONE`) |
| `bosh.ca-cert-file`<br />`BOSH_EXPORTER_BOSH_CA_CERT_FILE` | No | | BOSH CA Certificate file |
| `filter.deployments`<br />`BOSH_EXPORTER_FILTER_DEPLOYMENTS` | No | | Comma separated deployments to filter |
| `filter.azs`<br />`BOSH_EXPORTER_FILTER_AZS` | No | | Comma separated AZs to filter |
| `filter.collectors`<br />`BOSH_EXPORTER_FILTER_COLLECTORS` | No | | Comma separated collectors to filter (`Deployments`, `Jobs`, `ServiceDiscovery`) |
| `metrics.namespace`<br />`BOSH_EXPORTER_METRICS_NAMESPACE` | No | `bosh` | Metrics Namespace |
| `sd.filename`<br />`BOSH_EXPORTER_SD_FILENAME` | No | `bosh_target_groups.json` | Full path to the Service Discovery output file |
| `sd.processes_regexp`<br />`BOSH_EXPORTER_SD_PROCESSES_REGEXP` | No | | Regexp to filter Service Discovery processes names |
| `web.listen-address`<br />`BOSH_EXPORTER_WEB_LISTEN_ADDRESS` | No | `:9190` | Address to listen on for web interface and telemetry |
| `web.telemetry-path`<br />`BOSH_EXPORTER_WEB_TELEMETRY_PATH` | No | `/metrics` | Path under which to expose Prometheus metrics |

*[1]* When BOSH delegates user managament to [UAA][bosh_uaa], `bosh.uaa.url`, `bosh.uaa.client-id` and `bosh.uaa.client-secret` flags will be required; otherwise `bosh.username` and `bosh.password` will be required.

### Metrics

The exporter returns the following metrics:

| Metric | Description | Labels |
| ------ | ----------- | ------ |
| *metrics.namespace*_scrapes_total | Total number of times BOSH was scraped for metrics | |
| *metrics.namespace*_scrape_errors_total | Total number of times an error occured scraping BOSH | |
| *metrics.namespace*_last_scrape_error | Whether the last scrape of metrics from BOSH resulted in an error (`1` for error, `0` for success) | |
| *metrics.namespace*_last_scrape_timestamp | Number of seconds since 1970 since last scrape from BOSH | |
| *metrics.namespace*_last_scrape_duration_seconds | Duration of the last scrape from BOSH | |

The exporter returns the following `Deployments` metrics:

| Metric | Description | Labels |
| ------ | ----------- | ------ |
| *metrics.namespace*_deployment_release_info | Labeled BOSH Deployment Release Info with a constant `1` value | `bosh_deployment`, `bosh_release_name`, `bosh_release_version` |
| *metrics.namespace*_deployment_stemcell_info | Labeled BOSH Deployment Stemcell Info with a constant `1` value | `bosh_deployment`, `bosh_stemcell_name`, `bosh_stemcell_version`, `bosh_stemcell_os_name` |
| *metrics.namespace*_last_deployments_scrape_timestamp | Number of seconds since 1970 since last scrape of Deployments metrics from BOSH | |
| *metrics.namespace*_last_deployments_scrape_duration_seconds | Duration of the last scrape of Deployments metrics from BOSH | |

The exporter returns the following `Jobs` metrics:

| Metric | Description | Labels |
| ------ | ----------- | ------ |
| *metrics.namespace*_job_healthy | BOSH Job Healthy (1 for healthy, 0 for unhealthy) | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_load_avg01 | BOSH Job Load avg01 | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_load_avg05 | BOSH Job Load avg05 | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_load_avg15 | BOSH Job Load avg15 | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_cpu_sys | BOSH Job CPU System | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_cpu_user | BOSH Job CPU User | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_cpu_wait | BOSH Job CPU Wait | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_mem_kb | BOSH Job Memory KB | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_mem_percent | BOSH Job Memory Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_swap_kb | BOSH Job Swap KB | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_swap_percent | BOSH Job Swap Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_system_disk_inode_percent | BOSH Job System Disk Inode Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_system_disk_percent | BOSH Job System Disk Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_ephemeral_disk_inode_percent | BOSH Job Ephemeral Disk Inode Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_ephemeral_disk_percent | BOSH Job Ephemeral Disk Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_persistent_disk_inode_percent | BOSH Job Persistent Disk Inode Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_persistent_disk_percent | BOSH Job Persistent Disk Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip` |
| *metrics.namespace*_job_process_healthy | BOSH Job Process Healthy (1 for healthy, 0 for unhealthy) | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip`, `bosh_job_process_name` |
| *metrics.namespace*_job_process_uptime_seconds | BOSH Job Process Uptime in seconds | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip`, `bosh_job_process_name` |
| *metrics.namespace*_job_process_cpu_total | BOSH Job Process CPU Total | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip`, `bosh_job_process_name` |
| *metrics.namespace*_job_process_mem_kb | BOSH Job Process Memory KB | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip`, `bosh_job_process_name` |
| *metrics.namespace*_job_process_mem_percent | BOSH Job Process Memory Percent | `bosh_deployment`, `bosh_job_name`, `bosh_job_id`, `bosh_job_index`, `bosh_job_az`, `bosh_job_ip`, `bosh_job_process_name` |
| *metrics.namespace*_last_jobs_scrape_timestamp | Number of seconds since 1970 since last scrape of Job metrics from BOSH | |
| *metrics.namespace*_last_jobs_scrape_duration_seconds | Duration of the last scrape of Job metrics from BOSH | |

The exporter returns the following `ServiceDiscovery` metrics:

| Metric | Description | Labels |
| ------ | ----------- | ------ |
| *metrics.namespace*_last_service_discovery_scrape_timestamp | Number of seconds since 1970 since last scrape of Service Discovery from BOSH | |
| *metrics.namespace*_last_service_discovery_scrape_duration_seconds | Duration of the last scrape of Service Discovery from BOSH | |

### Service Discovery

If the `ServiceDiscovery` collector is enabled, the exporter will write a `json` file at the `sd.filename` location containing a list of static configs that can be used with the Prometheus [file-based service discovery][file_sd_config] mechanism:

```json
[
  {
    "targets": ["10.244.0.12"],
    "labels":
      {
        "__meta_bosh_job_process_name": "bosh_exporter"
      }
  },
  {
    "targets": ["10.244.0.11", "10.244.0.12", "10.244.0.13", "10.244.0.14"],
    "labels":
      {
        "__meta_bosh_job_process_name": "node_exporter"
      }
  }
]
```

The list of targets can be filtered using the `sd.processes_regexp` flag.

## Contributing

Refer to the [contributing guidelines][contributing].

## License

Apache License 2.0, see [LICENSE][license].

[binaries]: https://github.com/cloudfoundry-community/bosh_exporter/releases
[bosh]: https://bosh.io
[bosh_uaa]: http://bosh.io/docs/director-users-uaa.html
[cloudfoundry]: https://www.cloudfoundry.org/
[contributing]: https://github.com/cloudfoundry-community/bosh_exporter/blob/master/CONTRIBUTING.md
[faq]: https://github.com/cloudfoundry-community/bosh_exporter/blob/master/FAQ.md
[file_sd_config]: https://prometheus.io/docs/operating/configuration/#&lt;file_sd_config&gt;
[golang]: https://golang.org/
[license]: https://github.com/cloudfoundry-community/bosh_exporter/blob/master/LICENSE
[manifest]: https://github.com/cloudfoundry-community/bosh_exporter/blob/master/manifest.yml
[prometheus]: https://prometheus.io/
[prometheus-boshrelease]: https://github.com/cloudfoundry-community/prometheus-boshrelease
