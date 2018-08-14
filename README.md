## Description
Builds AS3 declarations based on inputs from variable files provided. Enables the ability to abstract complexities using AS3 and Ansible.

#### High level workflow
* User updates environment variables in **vars/main.yaml**
* User provides app variables in **app-inputs** directory
* **create_app.yaml** creates AS3 declaration segments based on templates within **j2** directory.
* **push_config.yaml** merges AS3 segements into single declarations and pushes across all BIG-IPs in the **vars/main.yaml** file

#### Setup

## Setup
1. Edit the **hosts** file.
    1. In the examples hosts file there are 3 BIG-IPs (Zone1, Zone2, aws). Update these to match the infrastructure of your orginization.
    1. Make sure to update the localhost device to use the correct python env interpreter.
1. Update **vars/main.yaml** to reference the `hosts` you created along       with the **Tenants** required for your applications.

    ```yaml
    datacenters:
      - aws
      - Zone1
      - Zone2
    tenants:
      - Tenant1
      - Tenant2
    f5_user: admin
    f5_password: admin
    ```
    1. Ensure that the password is encrypted with vault when used outside of this demo.
       ```
       ansible-vault encrypt_string admin --ask-vault-pass --name f5_password
       ```
1. Create var files that represent your application ADC parameters within the **app-inputs** directory. There are 3 example var files provided within the repo.

1. Create the AS3 declaration fragments using the **create_app.yaml** playbook.

    ```
    ansible-playbook -i hosts create_app.yaml -e "@app-inputs/App1.yaml"
    ```
   1. You will notice that this playbook will also create the **apps** folder and then tree out each Zone/Tenant combination.

   ```
    ├── apps
    │   ├── Zone1
    │   │   ├── Tenant1
    │   │   │   └── App1.json
    │   │   └── Tenant2
    │   │       └── App3.json
    │   ├── Zone2
    │   │   ├── Tenant1
    │   │   │   ├── App1.json
    │   │   │   └── App2.json
    │   │   └── Tenant2
    │   └── aws
    │       ├── Tenant1
    │       │   ├── App1.json
    │       │   └── App2.json
    │       └── Tenant2
    │           └── App3.json
   ```

1. *This is hopefully a temporary step.* As of right now one of the modules within **push_config.yaml** requires a full path and does not work with relative. Please update the `dest` line as below to match your environment. `/Users/test/Desktop/sc/ansible-as3-app-files/`

```yaml
  - name: Group all Application Files
    assemble:
        ....
      dest: /Users/test/Desktop/sc/ansible-as3-app-files/tmp/{{item[0]}}-{{item[1]}}_combined.yaml
```
1.  Run the **push_config.yaml** playbook to actually push the AS3 declarations to your BIG-IPs. Remember to use `--ask-vault-pass` parameter if you have encrypted the passwords for your BIG-IPs.
    ```
    ansible-playbook -i hosts push_config.yaml
    ```