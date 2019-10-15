# Falcosidekick

![falcosidekick](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/falcosidekick_color.png)

![release](https://flat.badgen.net/github/release/falcosecurity/falcosidekick/latest?color=green) ![last commit](https://flat.badgen.net/github/last-commit/falcosecurity/falcosidekick) ![licence](https://flat.badgen.net/badge/license/MIT/blue) ![docker pulls](https://flat.badgen.net/docker/pulls/falcosecurity/falcosidekick?icon=docker)

## Description

A simple daemon to help you with falco's outputs (https://sysdig.com/opensource/falco/). It takes a falco's event and forwards it to different outputs.

## Outputs

Currently available outputs are :

* [**Slack**](https://slack.com)
* [**Teams**](https://products.office.com/en-us/microsoft-teams/group-chat-software)
* [**Datadog**](https://www.datadoghq.com/)
* [**AlertManager**](https://prometheus.io/docs/alerting/alertmanager/)
* [**Elasticsearch**](https://www.elastic.co/)
* [**Loki**](https://grafana.com/oss/loki)
* [**NATS**](https://nats.io/)
* [**Influxdb**](https://www.influxdata.com/products/influxdb-overview/)
* [**AWS Lambda**](https://aws.amazon.com/lambda/features/)
* [**AWS SQS**](https://aws.amazon.com/sqs/features/)
* **SMTP** (email)
* [**Opsgenie**](https://www.opsgenie.com/)

## Usage

Run the daemon as any other daemon in your architecture (systemd, k8s daemonset, swarm service, ...)

### With docker

```bash
docker run -d -p 2801:2801 -e SLACK_WEBHOOKURL=XXXX -e DATADOG_APIKEY=XXXX falcosecurity/falcosidekick
```

### With Helm

```bash

git clone https://github.com/falcosecurity/falcosidekick.git
cd ./falcosidekick/deploy/helm/falcosidekick/
helm install --name falcosidekick .
```

### Falco's config

Add this (adapted to your environment) in your *falco.yaml* :

```bash
json_output: true
json_include_output_property: true
program_output:
  enabled: true
  keep_alive: false
  program: "curl -d @- localhost:2801/"
```

### Configuration

Configuration is made by *file (yaml)* and *env vars*, both can be used but *env vars* override values from *file*.

#### YAML File

See **config_example.yaml** :

```yaml
#listenport: 2801 # port to listen for daemon (default: 2801)
debug: false # if true all outputs will print in stdout the payload they send (default: false)
customfields: # custom fields are added to falco events
  Akey: "AValue"
  Bkey: "BValue"
  Ckey: "CValue"

slack:
  webhookurl: "" # Slack WebhookURL (ex: https://hooks.slack.com/services/XXXX/YYYY/ZZZZ), if not empty, Slack output is enabled
  #footer: "" # Slack footer
  #icon: "" # Slack icon (avatar)
  outputformat: "text" # all (default), text, fields
  minimumpriority: "debug" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)
  messageformat: "Alert : rule *{{ .Rule }}* triggered by user *{{ index .OutputFields \"user.name\" }}*" # a Go template to format Slack Text above Attachment, displayed in addition to the output from `SLACK_OUTPUTFORMAT`, see [Slack Message Formatting](#slack-message-formatting) in the README for details. If empty, no Text is displayed before Attachment.

teams:
  webhookurl: "" # Teams WebhookURL (ex: https://hooks.slack.com/services/XXXX/YYYY/ZZZZ), if not empty, Teams output is enabled
  #activityimage: "" # Image for message section
  outputformat: "text" # all (default), text, facts
  minimumpriority: "debug" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

datadog:
  #apikey: ""  # Datadog API Key, if not empty, Datadog output is enabled
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

alertmanager:
  # hostport: "" # http://{domain or ip}:{port}, if not empty, Alertmanager output is enabled
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

elasticsearch:
  # hostport: "" # http://{domain or ip}:{port}, if not empty, Elasticsearch output is enabled
  # index: "falco" # index (default: falco)
  # type: "event"
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)
  # suffix: "daily" # date suffix for index rotation : daily (default), monthly, annually, none

influxdb:
  # hostport: "" # http://{domain or ip}:{port}, if not empty, Influxdb output is enabled
  # database: "falco" # Influxdb database (default: falco)
  # user: "" # user to use if auth is enabled in Influxdb
  # password: "" # pasword to use if auth is enabled in Influxdb
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

loki:
  # hostport: "" # http://{domain or ip}:{port}, if not empty, Loki output is enabled
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

nats:
  # hostport: "" # nats://{domain or ip}:{port}, if not empty, NATS output is enabled
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

aws:
  # accesskeyid: "" # aws access key (optionnal if you use EC2 Instance Profile)
  # secretaccesskey: "" # aws secret access key (optionnal if you use EC2 Instance Profile)
  # region : "" # aws region (optionnal if you use EC2 Instance Profile)
  lambda:
    # functionname : "" # Lambda function name, if not empty, AWS Lambda output is enabled
    # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)
  sqs:
    # url : "" # SQS Queue URL, if not empty, AWS SQS output is enabled
    # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

smtp:
  # hostport: "" # host:port address of SMTP server, if not empty, SMTP output is enabled
  # user: "" # user to access SMTP server
  # password: "" # password to access SMTP server
  # from: "" # Sender address (mandatory if SMTP output is enabled)
  # to: "" # comma-separated list of Recipident addresses, can't be empty (mandatory if SMTP output is enabled)
  # outputformat: "" # html (default), text
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)

opsgenie:
  # apikey: "" # Opsgenie API Key, if not empty, Opsgenie output is enabled
  # region: "eu" # (us|eu) region of your domain (default is 'us')
  # minimumpriority: "" # minimum priority of event for using this output, order is emergency|alert|critical|error|warning|notice|informational|debug or "" (default)
```

Usage :

```bash
usage: falcosidekick [<flags>]

Flags:
      --help                     Show context-sensitive help (also try --help-long and --help-man).
  -c, --config-file=CONFIG-FILE  config file
```

#### Env vars

Configuration of the daemon can be made also by *env vars*, these values override these from *yaml file*.

The *env vars* "match" field names in *yaml file with this structure (**take care of lower/uppercases**) : `yaml: a.b --> envvar: A_B` :

* **LISTENPORT** : port to listen for daemon (default: 2801)
* **DEBUG** : if *true* all outputs will print in stdout the payload they send (default: false)
* **CUSTOMFIELDS** : a list of comma separated custom fields to add to falco events, syntax is "key:value,key:value"
* **SLACK_WEBHOOKURL** : Slack Webhook URL (ex: https://hooks.slack.com/services/XXXX/YYYY/ZZZZ), if not `empty`, Slack output is *enabled*
* **SLACK_FOOTER** : Slack footer
* **SLACK_ICON** : Slack icon (avatar)
* **SLACK_OUTPUTFORMAT** : `all` (default), `text` (only text is displayed in Slack), `fields` (only fields are displayed in Slack)
* **SLACK_MINIMUMPRIORITY** : minimum priority of event for using use this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **SLACK_MESSAGEFORMAT** : a Go template to format Slack Text above Attachment, displayed in addition to the output from `SLACK_OUTPUTFORMAT`, see [Slack Message Formatting](#slack-message-formatting) in the README for details. If empty, no Text is displayed before Attachment.
* **TEAMS_WEBHOOKURL** : Teams Webhook URL (ex: https://outlook.office.com/webhook/XXXXXX/IncomingWebhook/YYYYYY"), if not `empty`, Teams output is *enabled*
* **TEAMS_ACTIVITYIMAGE** : Teams section image
* **TEAMS_OUTPUTFORMAT** : `all` (default), `text` (only text is displayed in Teams), `facts` (only facts are displayed in Teams)
* **TEAMS_MINIMUMPRIORITY** : minimum priority of event for using use this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **DATADOG_APIKEY** : Datadog API Key, if not `empty`, Datadog output is *enabled*
* **DATADOG_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **ALERTMANAGER_HOSTPORT** : AlertManager http://host:port, if not `empty`, AlertManager is *enabled*
* **ALERTMANAGER_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **ELASTICSEARCH_HOSTPORT** : Elasticsearch http://host:port, if not `empty`, Elasticsearch is *enabled*
* **ELASTICSEARCH_INDEX** : Elasticsearch index (default: falco)
* **ELASTICSEARCH_TYPE** : Elasticsearch document type (default: event)
* **ELASTICSEARCH_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **ELASTICSEARCH_SUFFIX** : date suffix for index rotation : `daily` (default), `monthly`, `annually`, `none`
* **INFLUXDB_HOSTPORT** : Influxdb http://host:port, if not `empty`, Influxdb is *enabled*
* **INFLUXDB_DATABASE** : Influxdb database (default: falco)
* **INFLUXDB_USER** : user to use if auth is enabled in Influxdb
* **INFLUXDB_PASSWORD** : user to use if auth is enabled in Influxdb
* **INFLUXDB_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **LOKI_HOSTPORT** : Loki http://host:port, if not `empty`, Loki is *enabled*
* **LOKI_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **NATS_HOSTPORT** : NATS nats://host:port, if not `empty`, NATS is *enabled*
* **NATS_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **AWS_ACCESSKEYID** : AWS Access Key Id (optionnal if you use EC2 Instance Profile)
* **AWS_SECRETACCESSKEY** : AWS Secret Access Key (optionnal if you use EC2 Instance Profile)
* **AWS_REGION** : AWS Region (optionnal if you use EC2 Instance Profile)
* **AWS_LAMBDA_FUNCTIONNAME** : AWS Lambda Function Name, if not empty, AWS Lambda output is enabled
* **AWS_LAMBDA_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **AWS_SQS_URL** : AWS SQS Queue URL, if not empty, AWS SQS output is enabled
* **AWS_SQS_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **SMTP_HOSTPORT** :  host:port address of SMTP server, if not empty, SMTP output is enabled
* **SMTP_USER** : user to access SMTP server
* **SMTP_PASSWORD** : password to access SMTP server
* **SMTP_FROM** : Sender address (mandatory if SMTP output is enabled)
* **SMTP_TO** : comma-separated list of Recipident addresses, can't be empty (mandatory if SMTP output is enabled)
* **SMTP_OUTPUTFORMAT** : "" # html (default), text
* **SMTP_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`
* **OPSGENIE_APIKEY** : Opsgenie API Key, if not empty, Opsgenie output is enabled
* **OPSGENIE_REGION** : "" # (us|eu) region of your domain (default is 'us')
* **OPSGENIE_MINIMUMPRIORITY** : minimum priority of event for using this output, order is `emergency|alert|critical|error|warning|notice|informational|debug or "" (default)`

#### Slack Message Formatting

The `SLACK_MESSAGEFORMAT` environment variable and `slack.messageformat` YAML value accept a [Go template](https://golang.org/pkg/text/template/) which can be used to format the text of a slack alert. These templates are evaluated on the JSON data from each Falco event - the following fields are available:
                                                                                                                   
| Template Syntax                              | Description      |
|----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `{{ .Output }}`                              | A formatted string from Falco describing the event.                                                                                                      |
| `{{ .Priority }}`                            | The priority of the event, as a string.                                                                                                                  |
| `{{ .Rule }}`                                | The name of the rule that generated the event.                                                                                                           |
| `{{ .Time }}`                                | The timestamp when the event occurred.                                                                                                                   |
| `{{ index .OutputFields \"<field name>\" }}` | A map of additional optional fields emitted depending on the event. These may not be present for every event, in which case they expand to the string `<no value>`   |

Go templates also support some basic methods for text manipulation which can be used to improve the clarity of alerts - see the documentation for details.

## Handlers

Different URI (handlers) are available :

* `/` : main and default handler, your falco config must be configured to use it
* `/ping` : you will get a  `pong` as answer, useful to test if falcosidekick is running and its port is opened (for healthcheck purpose for example)
* `/test` : (for debug only) send a test event to all enabled outputs.
* `/debug/vars` : get statistics from daemon (in JSON format), it uses classic `expvar` package and some custom values are added

## Logs

All logs are sent to `stdout`.

```bash
2019/05/10 14:32:06 [INFO] : Enabled Outputs : Slack Datadog
```

## Metrics

The daemon exposes the common *Golang* metrics and some custom values in JSON format. It's useful for monitoring purpose.

![expvar json](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/expvar_json.png)
![expvarmon](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/expvarmon.png)

## Examples

Run you daemon and try (from falco's documentation) :

```bash
curl "http://localhost:2801/" -d'{"output":"16:31:56.746609046: Error File below a known binary directory opened for writing (user=root command=touch /bin/hack file=/bin/hack)","priority":"Error","rule":"Write below binary dir","time":"2019-05-17T15:31:56.746609046Z", "output_fields": {"evt.time":1507591916746609046,"fd.name":"/bin/hack","proc.cmdline":"touch /bin/hack","user.name":"root"}}'
```

You should get :

### Slack

(SLACK_OUTPUTFORMAT="**all**")
![slack example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/slack.png)
(SLACK_OUTPUTFORMAT="**text**")
![slack no fields example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/slack_no_fields.png)
(SLACK_OUTPUTFORMAT="**fields**" and SLACK_MESSAGEFORMAT="**Alert : rule \*{{ .Rule }}\* triggered by user \*{{ index .OutputFields \"user.name\" }}\***")
![slack message format example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/slack_fields_messageformat.png)

### Teams

(TEAMS_OUTPUTFORMAT="**all**")
![teams example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/teams.png)
(TEAMS_OUTPUTFORMAT="**text**")
![teams facts only](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/teams_text.png)

### Datadog

*(Tip: filter on `sources: falco`)*
![datadog example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/datadog.png)

### AlertManager

![alertmanager example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/alertmanager.png)

### Elasticsearch (with Kibana)

![kibana example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/kibana.png)

### Influxdb

```bash
> use falco
Using database falco
> show series
key
---
events,akey=AValue,bkey=BValue,ckey=CValue,priority=Debug,rule=Testrule
events,akey=A_Value,bkey=B_Value,ckey=C_Value,priority=Debug,rule=Test_rule
> select * from events
name: events
time                akey    bkey    ckey    priority rule      value
----                ----    ----    ----    -------- ----      -----
1560433816893368400 AValue  BValue  CValue  Debug    Testrule  This is a test from falcosidekick
1560441359119741800 A_Value B_Value C_Value Debug    Test_rule This is a test from falcosidekick
```

### Loki (with Grafana)

![loki example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/loki.png)

### AWS SQS

![aws sqs example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/aws_sqs.png)

### SMTP

(SMTP_OUTPUTFORMAT="**html**")
![smtp html example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/smtp_html.png)
(SMTP_OUTPUTFORMAT="**text**")
![smtp plaintext example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/smtp_plaintext.png)

### Opsgenie

![opsgenie example](https://github.com/falcosecurity/falcosidekick/raw/master/imgs/opsgenie.png)

## Development

### Build

```bash
go build
```

### Test & Coverage

```bash
go test ./outputs -count=1 -cover -v
```

## Author

Thomas Labarussias (https://github.com/Issif)
