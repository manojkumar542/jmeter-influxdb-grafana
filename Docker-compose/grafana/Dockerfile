FROM grafana/grafana:8.1.0

ENV GF_SECURITY_ADMIN_USER=admin
ENV GF_SECURITY_ADMIN_PASSWORD=grafana
ENV GF_INSTALL_PLUGINS=grafana-clock-panel,briangann-gauge-panel,natel-plotly-panel,grafana-simple-json-datasource

# Add provisioning
ADD ./provisioning /etc/grafana/provisioning
# Add configuration file
#ADD ./grafana.ini /etc/grafana/grafana.ini
