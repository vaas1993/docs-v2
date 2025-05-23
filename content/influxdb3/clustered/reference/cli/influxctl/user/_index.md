---
title: influxctl user
description: >
  The `influxctl user` command and its subcommands manage users in InfluxDB clusters.
menu:
  influxdb3_clustered:
    parent: influxctl
weight: 201
---

The `influxctl user` command and its subcommands manage users in
{{< product-name omit="Clustered" >}} clusters.

## Usage

```sh
influxctl user [subcommand] [subcommand options] [arguments...]
```

## Subcommands

| Subcommand                                                         | Description         |
| :----------------------------------------------------------------- | :------------------ |
| [list](/influxdb3/clustered/reference/cli/influxctl/user/list/)     | List all users      |
| help, h                                                            | Output command help |

<!--
| [delete](/influxdb3/clustered/reference/cli/influxctl/user/delete/) | Delete a user       |
| [invite](/influxdb3/clustered/reference/cli/influxctl/user/invite/) | Invite a user       |
 -->

## Flags

| Flag |          | Description         |
| :--- | :------- | :------------------ |
| `-h` | `--help` | Output command help |
