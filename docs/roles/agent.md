# AxonOps Agent Role

## Overview

The `agent` role installs and configures the AxonOps Agent on Cassandra nodes. The AxonOps Agent collects metrics, logs, and diagnostic information from your Apache Cassandra or DataStax Enterprise (DSE) clusters and sends them to the AxonOps Server for monitoring and analysis.

## Requirements

- Ansible 2.9 or higher
- Target system running a supported Linux distribution (RHEL, CentOS, Ubuntu, Debian)
- Apache Cassandra or DataStax Enterprise installed (or install it using the `cassandra` role)
- Network connectivity to AxonOps Server (self-hosted or SaaS)

## Role Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `axon_agent_server_host` | Server name or IP where the AxonOps server is running | `agents.axonops.cloud` or `192.168.1.10` |
| `axon_agent_customer_name` | Organization name to identify your deployment | `mycompany` |

### Java Agent Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `axon_java_agent` | `axon-cassandra5.0-agent-jdk17` | Java agent version to install. Options: `axon-cassandra3.11-agent`, `axon-cassandra4.1-agent`, `axon-cassandra5.0-agent-jdk17`, `axon-dse6.0-agent`, `axon-dse6.7-agent`, `axon-dse5.1-agent`, or empty string to skip |
| `axon_agent_version` | `""` (latest) | Version of the AxonOps agent to install. Required for offline installation |
| `axon_java_agent_version` | `1.0.10` | Version of the Java agent to install |
| `has_internet_access` | `true` | When `false`, install the agent and the Java agent from `axon_agent_download_path` and `axon_java_agent_download_path` instead of the repository |
| `axon_agent_download_path` | - | Offline install: path on the control node to the `axon-agent` `.deb` (Debian) or `.rpm` (RedHat) |
| `axon_java_agent_download_path` | - | Offline install: path on the control node to the Java agent `.deb` or `.rpm` |
| `axon_java_agent_force_offline_install` | `false` | Install the Java agent from `axon_java_agent_download_path` even when `has_internet_access` is `true` |

### Network Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `axon_agent_server_port` | `443` (SaaS) or `1888` (self-hosted) | Port to connect to AxonOps Server |
| `axon_agent_host_name` | `{{ ansible_hostname }}` | Override the agent hostname if needed |

### TLS Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `axon_agent_tls_mode` | undefined | TLS mode: `disabled`, `TLS`, or `mTLS`. Leave undefined for the SaaS or the default server settings |
| `axon_agent_tls_certfile` | - | Path to TLS certificate file (the role asserts it is set for TLS/mTLS; written to `axon-agent.yml` for mTLS only) |
| `axon_agent_tls_keyfile` | - | Path to TLS key file (the role asserts it is set for TLS/mTLS; written to `axon-agent.yml` for mTLS only) |
| `axon_agent_tls_cafile` | - | Path to CA certificate file |
| `axon_agent_tls_skipverify` | `false` | Skip server certificate verification (TLS/mTLS) |

### Cassandra Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `axon_agent_cassandra_config_directory` | - | Path to Cassandra configuration directory (e.g., `/opt/cassandra/conf`) to enable automatic agent configuration in `cassandra-env.sh` |
| `axon_agent_cassandra_config` | - | Configuration line to add to `cassandra-env.sh`. Preferred for agents >= 1.1.0 (Cassandra 3.11/4.0/4.1/5.0): `. /usr/share/axonops/axonops-jvm.options`. Older agents: `JVM_OPTS="$JVM_OPTS -javaagent:/usr/share/axonops/axon-cassandra4.0-agent.jar=/etc/axonops/axon-agent.yml"` |

### Repository Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `axon_agent_public_repository` | `present` | State of the public apt repository line: `present` or `absent` |
| `axon_agent_beta_repository` | `absent` | State of the beta apt repository line |
| `axon_agent_dev_repository` | `absent` | State of the dev apt repository line |
| `axonops_manage_repository` | `true` | Add the AxonOps package repository (and its signing key on Debian). Set to `false` when the host already has the source configured, for example an internal mirror managed outside this role |
| `axonops_debian_repository` | `deb https://packages.axonops.com/apt axonops-apt main` | apt source line. Point it to an internal mirror for air-gapped hosts |
| `axonops_debian_repository_key_url` | `https://packages.axonops.com/apt/repo-signing-key.gpg` | URL of the key that signs the apt repository; the source line, written without `[options]`, then gets `signed-by`. Empty uses `axonops_debian_repository` exactly as written (for example with `[trusted=yes]` or your own `signed-by`) |
| `axonops_redhat_repository` | `https://packages.axonops.com/yum` | yum `baseurl` |
| `axon_agent_redhat_repository` | - | Legacy override of `axonops_redhat_repository`, still honoured when set |

### Other Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `axon_agent_start_at_boot` | `true` | Enable the agent to start at boot |
| `axon_agent_state` | `present` | State of the agent: `present` or `absent` |
| `axon_agent_ntp_server` | undefined | NTP server written to `axon-agent.yml`. Not set by default |
| `axon_agent_key` | - | Agent key for the AxonOps server or SaaS organisation |
| `axon_agent_cluster_name` | - | Cluster name reported by the agent |
| `axon_agent_limitnofile` | - | If set, adds a systemd drop-in with `LimitNOFILE` for `axon-agent` |
| `axon_agent_hostid` | - | If set, writes `/var/lib/axonops/hostId` |
| `axon_agent_kafka_config_directory` | - | Kafka `bin` directory; when set, adds `axon_agent_kafka_config` to `kafka-server-start.sh` |
| `axon_agent_kafka_config` | - | Line added to `kafka-server-start.sh` (e.g. `. /usr/share/axonops/axonops-jvm.options`) |
| `axon_agent_kafka_client_properties` | `""` | Path to a Kafka `client.properties` file used by the AxonOps Kafka admin client |
| `axon_agent_jvm_options` | - | If set, replaces the content of `/usr/share/axonops/axonops-jvm.options` |
| `axon_agent_install` / `axon_agent_configure` | `true` | Run the install and configure task files |

## Dependencies

None

## Example Playbooks

### Basic Agent Installation with SaaS

```yaml
- name: Install AxonOps Agent for SaaS
  hosts: cassandra
  become: true
  vars:
    axon_agent_customer_name: mycompany
    axon_java_agent: "axon-cassandra4.1-agent"

  roles:
    - role: axonops.axonops.agent
```

### Agent with Self-Hosted Server

```yaml
- name: Install AxonOps Agent with Self-Hosted Server
  hosts: cassandra
  become: true
  vars:
    axon_agent_server_host: "{{ groups['axon-server'] | first }}"
    axon_agent_customer_name: mycompany
    axon_java_agent: "axon-cassandra5.0-agent-jdk17"
    axon_agent_version: "2.0.21"
    axon_java_agent_version: "1.0.10"

  roles:
    - role: axonops.axonops.agent
      tags: agent
```

### Agent with TLS Enabled

```yaml
- name: Install AxonOps Agent with TLS
  hosts: cassandra
  become: true
  vars:
    axon_agent_server_host: axon-server.example.com
    axon_agent_customer_name: mycompany
    axon_java_agent: "axon-cassandra4.1-agent"
    axon_agent_tls_mode: "TLS"
    axon_agent_tls_certfile: /etc/axonops/certs/agent.crt
    axon_agent_tls_keyfile: /etc/axonops/certs/agent.key
    axon_agent_tls_cafile: /etc/axonops/certs/ca.crt

  roles:
    - role: axonops.axonops.agent
```

### Complete Stack: Java, Agent, and Cassandra

```yaml
- name: Deploy Complete Cassandra Stack with AxonOps
  hosts: cassandra
  become: true
  vars:
    axon_agent_server_host: "{{ groups['axon-server'] | first }}"
    axon_agent_customer_name: mycompany
    axon_java_agent: "axon-cassandra4.1-agent"
    java_pkg: java-11-openjdk-headless
    cassandra_dc: "DC1"
    cassandra_cluster_name: "production"
    cassandra_seeds: "{{ groups['cassandra'] | map('extract', hostvars, ['ansible_default_ipv4', 'address']) | list | first }}"

  roles:
    - role: axonops.axonops.java
      tags: java

    - role: axonops.axonops.agent
      tags: agent

    - role: axonops.axonops.cassandra
      tags: cassandra
```

### Agent with Automatic Cassandra Configuration

```yaml
- name: Install AxonOps Agent with Auto-Configuration
  hosts: cassandra
  become: true
  vars:
    axon_agent_server_host: "{{ groups['axon-server'] | first }}"
    axon_agent_customer_name: mycompany
    axon_java_agent: "axon-cassandra4.1-agent"
    # Automatically configure cassandra-env.sh
    axon_agent_cassandra_config_directory: /opt/cassandra/conf
    # Preferred for AxonOps agents >= 1.1.0 (3.11/4.0/4.1/5.0 all ship this file):
    axon_agent_cassandra_config: '. /usr/share/axonops/axonops-jvm.options'
    # Older agents that do not ship the options file use the raw -javaagent line:
    # axon_agent_cassandra_config: 'JVM_OPTS="$JVM_OPTS -javaagent:/usr/share/axonops/axon-cassandra4.0-agent.jar=/etc/axonops/axon-agent.yml"'

  roles:
    - role: axonops.axonops.agent
```

## Notes

- **Sudoers Configuration**: The role automatically configures sudo permissions for the axonops user to manage the Cassandra service
- **Automatic Cassandra Configuration**: You can optionally configure the role to automatically add the AxonOps agent configuration to Cassandra's `cassandra-env.sh` file by setting `axon_agent_cassandra_config_directory` and `axon_agent_cassandra_config`. Prefer sourcing the options file (`. /usr/share/axonops/axonops-jvm.options`), shipped by AxonOps Cassandra agents >= 1.1.0 for 3.11/4.0/4.1/5.0; older agents use the raw `-javaagent:` line. The role matches any existing AxonOps line and replaces it, so switching methods on an existing install does not add the agent twice. Check the [AxonOps documentation](https://axonops.com/docs/get_started/agent_setup/) for the correct configuration line for your Cassandra version
- **Service Configuration**: By default, the role does not include the service-specific configuration in `axon-agent.yml` unless `axon_agent_include_service_config` is set to `true`
- **Repository Management**: The role supports public, beta, and dev repositories. Use `axon_agent_public_repository`, `axon_agent_beta_repository`, and `axon_agent_dev_repository` to control which repositories are enabled. To install from an internal mirror, set `axonops_debian_repository` / `axonops_redhat_repository` (and `axonops_debian_repository_key_url` on Debian), or set `axonops_manage_repository: false` when the source is already configured on the host. On RedHat the install only enables repositories whose id starts with `axonops`, so a repository you configure yourself must use such an id. When an existing Debian host moves to a mirror, the role adds the new source but does not remove the previous `packages.axonops.com` source file under `/etc/apt/sources.list.d/`; remove it by hand
- **Offline Installation**: For air-gapped environments, set `has_internet_access: false` with pinned `axon_agent_version` and `axon_java_agent_version`, and `axon_agent_download_path` / `axon_java_agent_download_path` pointing to the `.deb` (Debian) or `.rpm` (RedHat) files on the control node. The role copies and installs them and, on Debian, does not set up the apt repository. `axon_java_agent_force_offline_install: true` installs only the Java agent from `axon_java_agent_download_path` while the agent itself comes from the repository

## Tags

The role defines no tags of its own. Apply a tag to the role in your playbook (for example `tags: axonops-agent`) to run it selectively.

## License

See the main collection LICENSE file.

## Author

AxonOps Limited
