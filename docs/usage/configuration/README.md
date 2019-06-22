---
description: Vector configuration
---

<!---
!!!WARNING!!!!

This file is autogenerated! Please do not manually edit this file.
Instead, please modify the contents of `scripts/schema.toml`.
-->


# Configuration

![](../../assets/configure.svg)

This section covers configuring Vector and creating [pipelines](../../about/concepts.md#pipelines) like the one shown above. Vector requires only a _single_ [TOML](https://github.com/toml-lang/toml) configurable file, which you can specify via the [`--config` flag](../administration/starting.md#options) when [starting](../administration/starting.md) vector:

```bash
vector --config /etc/vector/vector.toml
```

## Example

{% code-tabs %}
{% code-tabs-item title="vector.toml" %}
```coffeescript
data_dir = "/var/lib/vector"

# Ingest data by tailing one or more files
[sources.apache_logs]
    type         = "file"
    include      = ["/var/log/apache2/*.log"]
    ignore_older = 86400 # 1 day

# Structure and parse the data
[transforms.apache_parser]
    inputs        = ["apache_logs"]
  type            = "regex_parser"
  regex           = '^(?P<host>[w.]+) - (?P<user>[w]+) (?P<bytes_in>[d]+) [(?P<timestamp>.*)] "(?P<method>[w]+) (?P<path>.*)" (?P<status>[d]+) (?P<bytes_out>[d]+)$'

# Sample the data to save on cost
[transforms.apache_sampler]
    inputs       = ["apache_parser"]
    type         = "sampler"
    hash_field   = "request_id" # sample _entire_ requests
    rate         = 10 # only keep 10%

# Send structured data to a short-term storage
[sinks.es_cluster]
    inputs       = ["apache_sampler"]
    type         = "elasticsearch"
    host         = "79.12.221.222:9200"
    doc_type     = "_doc"

# Send structured data to a cost-effective long-term storage
[sinks.s3_archives]
    inputs       = ["apache_parser"] # don't sample
    type         = "aws_s3"
    region       = "us-east-1"
    bucket       = "my_log_archives"
    batch_size   = 10000000 # 10mb uncompressed
    gzip         = true
    encoding     = "ndjson"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Global Options

| Key  | Type  | Description |
| :--- | :---: | :---------- |
| `data_dir` | `string` | The directory used for persisting Vector state, such as on-disk buffers. Please make sure the Vector project has write permissions to this dir. See [Data Directory](#data-directory) for more info.<br />`no default` `example: "/var/lib/vector"` |

## How It Works

### Composition

The primary purpose of the configuration file is to compose pipelines. Pipelines are formed by connecting [sources][sources], [transforms][transforms], and [sinks][sinks] through the `inputs` option.

Notice in the above example each input references the `id` assigned to a previous source or transform.

### Config File Location

The location of your Vector configuration file depends on your [platform][platform] or [operating system][operating_system]. For most Linux based systems the file can be found at `/etc/vector/vector.toml`.

### Data Directory

Vector requires a `data_dir` value for on-disk operations. Currently, the only operation using this directory are Vector's on-disk buffers. Buffers, by default, are memory-based, but if you switch them to disk-based you'll need to specify a `data_directory`.

### Environment Variables

Vector will interpolate environment variables within your configuration file with the following syntax:

{% code-tabs %}
{% code-tabs-item title="vector.toml" %}
```coffeescript
[transforms.add_host]
    type = "add_fields"
    
    [transforms.add_host.fields]
        host = "${HOSTNAME}"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

The entire `${HOSTNAME}` variable will be replaced, hence the requirement of quotes around the definition.

#### Escaping

You can escape environment variable by preceding them with a `$` character. For example `$${HOSTNAME}` will be treated _literally_ in the above environment variable example.

### Format

The Vector configuration file requires the [TOML][toml] format for it's simplicity, explicitness, and relaxed white-space parsing. For more information, please refer to the excellent [TOML documentation][toml].

### Value Types

All TOML values types are supported. For convenience this includes:

* [Strings](https://github.com/toml-lang/toml#string)
* [Integers](https://github.com/toml-lang/toml#integer)
* [Floats](https://github.com/toml-lang/toml#float)
* [Booleans](https://github.com/toml-lang/toml#boolean)
* [Offset Date-Times](https://github.com/toml-lang/toml#offset-date-time)
* [Local Date-Times](https://github.com/toml-lang/toml#local-date-time)
* [Local Dates](https://github.com/toml-lang/toml#local-date)
* [Local Times](https://github.com/toml-lang/toml#local-time)
* [Arrays](https://github.com/toml-lang/toml#array)
* [Tables](https://github.com/toml-lang/toml#table)


[sources]: "../../../usage/configuration/sources"
[transforms]: "../../../usage/configuration/transforms"
[sinks]: "../../../usage/configuration/sinks"
[platform]: "../../setup/installation/platforms/"
[operating_system]: "../../setup/installation/operating_systems/"
[toml]: "https://github.com/toml-lang/toml"
