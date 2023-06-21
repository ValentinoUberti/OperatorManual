# Use main.yml to connect using the already present ~/.kube/config

## This is an example using k8s_auth module
```
- hosts: localhost
  module_defaults:
    group/k8s:
      host: https://k8s.example.com/
      ca_cert: ca.pem
  tasks:
  - block:
    # It's good practice to store login credentials in a secure vault and not
    # directly in playbooks.
    - include_vars: k8s_passwords.yml

    - name: Log in (obtain access token)
      community.kubernetes.k8s_auth:
        username: admin
        password: "{{ k8s_admin_password }}"
      register: k8s_auth_results

    # Previous task provides the token/api_key, while all other parameters
    # are taken from module_defaults
    - name: Get a list of all pods from any namespace
      community.kubernetes.k8s_info:
        api_key: "{{ k8s_auth_results.k8s_auth.api_key }}"
        kind: Pod
      register: pod_list

    always:
    - name: If login succeeded, try to log out (revoke access token)
      when: k8s_auth_results.k8s_auth.api_key is defined
      community.kubernetes.k8s_auth:
        state: absent
        api_key: "{{ k8s_auth_results.k8s_auth.api_key }}"
```
