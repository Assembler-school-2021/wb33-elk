# wb33-elk

> Pregunta: Habilita el módulo de Nginx y comprueba que recibes los logs adecuadamente. ¿Qué búsqueda harías en Discover, si únicamente queremos mostrar los logs del módulo Nginx?

	event.module: "nginx"
	

> Pregunta: Inspecciona un poco, crea otra visualización que consideres útil y añade las dos visualizaciones a un Dashboard.
He creado un dashboard con el registro ssh y con las conexines a nginx:

![image](https://user-images.githubusercontent.com/65896169/125494255-1fcf9aa2-22c2-40c6-9600-a0d9770da0f2.png)

## Recolección de eventos remotos
Recupera la máquina de la sesión anterior, instala Filebeat, habilita los modulos de System, Nginx y Mysql y realiza las configuraciones necesarias para que Filebeat envíe dichos logs a nuestro Elasticsearch.
> Pregunta: Facilita las capturas que consideres oportunas que demuestren que estás recibiendo los logs de la máquina remota (Netdata).
Aquí tenemos una captura en la que podemos ver por ejemplo que recibimos los logs de nginx del servidor trouble.devops-alumno08.com

![image](https://user-images.githubusercontent.com/65896169/125494290-65b04ce5-ce2b-4ce1-b8bb-48564e30e97e.png)
![image](https://user-images.githubusercontent.com/65896169/125501358-84008f83-9fb7-48f0-8299-02352c22a1e4.png)
![image](https://user-images.githubusercontent.com/65896169/125501575-0479667d-392a-4da8-89e5-58b1b42e7f44.png)

> Pregunta: Añade los logs de php-fpm
Modificamos filebeat.yml para añadir este trozo en file.inputs:
```
filebeat.inputs:
# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
  - /var/log/php/*fpm.log
  close_inactive: 24h

  # Optional additional fields. These fields can be freely picked
  # to add additional information to the crawled log files for filtering
  fields:
    event.dataset: "php.error"
  # The Ingest Node pipeline ID associated with this input. If this is set, it
  # overwrites the pipeline option from the Elasticsearch output.
  pipeline: PHP-FPM
```
Se ha creado un pipeline de nombre PHP-FPM
![image](https://user-images.githubusercontent.com/65896169/125510250-20961daf-9200-469a-8630-ad81073caafb.png)
Este es el código:
```
{
  "description": "Pipeline for parsing PHP-FPM log",
  "processors": [
    {
      "grok": {
        "pattern_definitions": {
          "PHP_DATE": "%{MONTHDAY}[\/-]%{MONTH}[\/-]%{YEAR} %{HOUR}:%{MINUTE}:%{SECOND}"
        },
        "ignore_missing": true,
        "field": "message",
        "patterns": [
          "\\[%{PHP_DATE:php.time}\\] %{LOGLEVEL:log.level}: %{GREEDYDATA:message}"
        ]
      }
    },
    {
      "rename": {
        "field": "@timestamp",
        "target_field": "event.created"
      }
    },
    {
      "date": {
        "field": "php.time",
        "target_field": "@timestamp",
        "formats": [
          "dd-MMM-yyyy H:m:s"
        ],
        "timezone":"{{event.timezone}}",
        "on_failure": [
          {
            "append": {
              "field": "error.message",
              "value": "{{ _ingest.on_failure_message }}"
            }
          }
        ]
      }
    },
    {
      "remove": {
        "field": "php.time",
        "ignore_failure": true
      }
    },
    {
      "grok": {
        "pattern_definitions": {
          "PHP_ROOT_PREFIX": "\\\/var\\\/www\\\/",
          "REQUEST_PREFIX": "\\(request:\\ \\\""
        },
        "ignore_missing": true,
        "ignore_failure": true,
        "field": "message",
        "patterns": [
          "%{PHP_ROOT_PREFIX}%{HOSTNAME:url.domain}.*%{REQUEST_PREFIX}%{WORD:http.request.method} %{URIPATH:url.original}"
        ]
      }
    },
    {
      "grok": {
        "pattern_definitions": {
          "TIME_PREFIX": "too\\ slow\\ \\("
        },
        "ignore_missing": true,
        "ignore_failure": true,
        "field": "message",
        "patterns": [
          "%{TIME_PREFIX}.*%{BASE10NUM:php.duration:float}"
        ]
      }
    },
    {
      "urldecode" : {
        "field": "message",
        "target_field":"message",
        "ignore_missing": true
      }
    }
  ],
  "on_failure": [
    {
      "set": {
        "field": "error.message",
        "value": "{{ _ingest.on_failure_message }}"
      }
    }
  ]
}

```
