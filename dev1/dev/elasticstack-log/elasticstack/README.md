# Elastic Stack

This is a template for an Elastic Stack including:

* Master nodes
* Data nodes
* An indices curator

## Usage

The following tasks MUST be completed if using this template as a base:

### kustomization.yaml

Config should be added to any `kustomization.yaml` file using this template as a base. In this example we are creating a new stack called "log" in the "monitoring" namespace:

```yaml
namespace: monitoring
nameSuffix: -log
commonLabels:
  app.kubernetes.io/instance: log
  instance: log
```

### Curator Config

The default actions file for the curator (`./elasticsearch-curator/configmaps/action_file.yml`) should be checked and modified in the overlay if necessary. The default settings will delete items in all indices after 7 days for example so be aware of this if you need longer retention. More details on how to configure this are available [here](https://github.com/elastic/curator/blob/master/docs/asciidoc/configuration.asciidoc).

### Kibana Config

Changes may be required to the Kibana advanced settings (`./kibana/configmaps/setup-kibana.sh`) which are changed via a `curl` to the REST API. Details of available settings are on the Elastic site [here](https://www.elastic.co/guide/en/kibana/current/advanced-options.html).