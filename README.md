# ESXi8u1 Automated Installation on Baremetal

This repository provides a fully automated solution for installing the ESXi8u1 operating system on HPE physical servers using iLO. With this code, you can effortlessly deploy ESXi servers by simply completing the variables and running the main playbook on your Ansible server. Say goodbye to manual installation hassles and hello to streamlined and efficient deployment!

## Installation Steps

Follow these steps to automate the installation of ESXi8u1 on your HPE products:

1. Clone this repository to your Ansible server.

2. Update the variables in the `vars.yml` file with the necessary information for your ESXi servers. Ensure that you provide accurate details, including network configuration, storage settings, and other relevant parameters.

3. Run the main playbook on your Ansible server:

   ```bash
   ansible-playbook main.yml
   ```

   This will initiate the automated installation process based on the provided variables.

4. Sit back and relax! After a while, your ESXi servers will be up and running, ready for use.

## Requirements

Before using this automation code, make sure you have the following:

- An Ansible server with Ansible installed. If you don't have Ansible installed, refer to the official [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/index.html).

- Access to HPE servers with iLO functionality. Ensure that you have the necessary credentials and network connectivity to interact with the iLO interface.

- Familiarity with ESXi and the specific configuration requirements for your environment.

## Customization

Feel free to customize the playbook and variables according to your specific needs. You can modify network settings, storage configurations, and other parameters to align with your infrastructure requirements.

## Contributing

Contributions are welcome! If you have any improvements or suggestions, please feel free to submit a pull request. Ensure that your changes align with the existing coding style and include relevant documentation.

## License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute this automation code as per the terms of the license.

---
