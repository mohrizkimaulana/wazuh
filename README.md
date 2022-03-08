# wazuh
Install Wazuh SIEM with ELK Stack in Production Deployment to monitoring VPS

Clone the Wazuh repository to your system:
  
    git clone https://github.com/wazuh/wazuh-docker.git -b v4.1.5 --depth=1
    
Change to wazuh docker directory:

    cd wazuh-docker
    
Edit default configuration of production-cluster.yml. You can delete all default configuration and paste new configuration
  
    nano prodution-cluster.yml
