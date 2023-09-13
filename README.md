# sleif.caddy_container

This role runs a Caddy instance on Podman.
By default ACME Letsencrypt will be used.

The role supports different `caddy_operations`.

```yaml
caddy_operations:
  - caddy_container
  - caddy_file_server
  - caddy_reverse_proxy
```

- caddy_container: pull, create and run from systemd units
- caddy_file_server: support simple web root; can be used to just fetch certificates for other services; certificates can be found below `caddy/data/caddy/certificates`
- caddy_reverse_proxy: redirects to other hosts and ports

## Requirements

- Podman installed
- ansible role sleif.podman installed

## Role Variables

- caddy_operation
- caddy_acme_staging: false

## Dependencies

This role depends on sleif.podman.

```sh
ansible-galaxy install sleif.podman --force
ansible-galaxy install sleif.caddy_container --force
```

## Example Playbook

```yaml
- name: VM caddy.example.com
  hosts: "caddy.example.com"
  user: root

  vars:
    podman_networks:
      podman_network_root:
        podman_network_name: 'podman_custom'
        podman_network_subnet: '10.0.0.0/16'
        podman_network_gateway: '10.0.0.1'
        # podman_network_iprange: '10.0.0.128/25'
      podman_network_rootless:
        podman_network_name: 'podman_custom_rootless'
        podman_network_subnet: '10.1.0.0/24'
        podman_network_gateway: '10.1.0.1'
        # podman_network_iprange: '10.0.1.128/25'
    podman_rootless: true
    podman_network_name: "{{ podman_networks.podman_network_rootless.podman_network_name }}"

  roles:
    - {role: sleif.podman, tags: "podman_role",
       podman_operation: "podman_install"}
    - {role: sleif.caddy_container, tags: "caddy, caddy_container", caddy_container_name: "caddy", caddy_operation: "caddy_container",
       container_name: caddy}
    - {role: sleif.caddy_container, tags: "caddy, caddy_container, caddy_certs_for_ldap",
       container_name: caddy,
       caddy_operation: caddy_file_server,
       caddy_target_uri: ldap.example.com}
    - {role: sleif.caddy_container, tags: "caddy, caddy_container, caddy_webservice, webservice",
       container_name: caddy,
       caddy_operation: caddy_reverse_proxy,
       caddy_target_uri: webservice.example.com,
       caddy_reverse_proxy_src: webservice,
       caddy_proxy_present: present,
       caddy_reverse_proxy_src_port: 8080}
```

## License

MIT

## Author Information

Created in 2023 by Sebastian Berthold
