# Run Prometheus and Grafana with docker-compose

This project shows how to run
[Prometheus](https://prometheus.io) and [Grafana](https://grafana.com)
together with [Docker Compose](https://docs.docker.com/compose).
You can use this setup on your workstation to scrape app metrics (disclaimer: this is not production-ready!).

## Prerequisites

You only need to install [docker-compose](https://docs.docker.com/compose/install)
on your workstation. If you have
[Docker Desktop](https://www.docker.com/products/docker-desktop) running,
`docker-compose` is already installed.

Make sure this command works:
```bash
$ docker-compose version
docker-compose version 1.24.0, build 0aa59064
docker-py version: 3.7.2
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
```

## Configuring Prometheus

Edit Prometheus configuration file in [prometheus.yml](prometheus.yml):
```yaml
global:
  scrape_interval: 10s
  evaluation_interval: 10s

scrape_configs:
- job_name: app1
  scheme: https
  metrics_path: /actuator/prometheus
  static_configs:
    - targets:
      - "app1.cfapps.io"
      labels:
        application: app1
```

You need to edit the `job_name` section, according to your app.

For example, if your app is running on your workstation, leveraging Spring Boot and Micrometer,
you may end up with this configuration:
```yaml
- job_name: local
  scheme: http
  metrics_path: /actuator/prometheus
  static_configs:
    - targets:
      - "192.168.0.10:8080"
      labels:
        application: local
```

Make sure you use **your public IP address** in case you bind a local app:
Prometheus and Grafana are running inside a VM managed by Docker, so you cannot use
`localhost` or `127.0.0.1` for the target address.

If you have several apps to monitor, you can add as many `job_name` sections as you need.
Make sure to use a different job name and an unique application label for each app.

## Starting Prometheus/Grafana

You are now ready to start Prometheus and Grafana.
Use `docker-compose` to start these components:
```bash
$ docker-compose up --force-recreate
```

Flag `force-recreate` is used to erase and recreate new containers with an
up-to-date configuration.

## Connecting to Grafana

Hit http://localhost:3000: you should see the Grafana login page.
Use `admin`/`admin` to sign-in.

At this point, a default datasource is running for Prometheus.
You just need to create a new dashboard for your apps, by selecting this datasource.

Beware that there is no persistence set for any data: you may loose
your configuration when you restart Prometheus/Grafana.

## Contribute

Contributions are always welcome!

Feel free to open issues & send PR.

## License

Copyright &copy; 2019 [Pivotal Software, Inc](https://pivotal.io).

This project is licensed under the [Apache Software License version 2.0](https://www.apache.org/licenses/LICENSE-2.0).
