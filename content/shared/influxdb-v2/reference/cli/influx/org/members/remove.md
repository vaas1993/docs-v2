
The `influx org members remove` command removes a member from an organization in InfluxDB.

## Usage
```
influx org members remove [flags]
```

## Flags
| Flag |                 | Description                                                | Input type | {{< cli/mapped >}}   |
|:-----|:----------------|:-----------------------------------------------------------|:----------:|:---------------------|
| `-h` | `--help`        | Help for the `remove` command                              |            |                      |
|      | `--host`        | HTTP address of InfluxDB (default `http://localhost:8086`) | string     | `INFLUX_HOST`        |
|      | `--http-debug`  | Inspect communication with InfluxDB servers.               | string     |                      |
| `-i` | `--id`          | Organization ID                                            | string     | `INFLUX_ORG_ID`      |
| `-m` | `--member`      | Member ID                                                  | string     |                      |
| `-n` | `--name`        | Organization name                                          | string     | `INFLUX_ORG`         |
|      | `--skip-verify` | Skip TLS certificate verification                          |            | `INFLUX_SKIP_VERIFY` |
| `-t` | `--token`       | API token                                                  | string     | `INFLUX_TOKEN`       |

## Examples

{{< cli/influx-creds-note >}}

##### Remove a member from an organization
```sh
influx org members remove \
  --member 00x0oo0X0xxxo000 \
  --name example-org
```
