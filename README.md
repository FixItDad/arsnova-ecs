# arsnova-ecs
Run a single server instance of [ARSnova](https://github/thm-projects/arsnova-mobile) in Amazon ECS

## General Notes
This initial version contains basic setup and teardown Ansible playbooks. There should be better separation of the VPC and ECS container construction and the actual app deployment. Also need to work on DNS, HTTPS configuration, health checks, etc.

AWS documentation on permissions needed for ECS containers seems to be incomplete and I ended up adding full EC2 and ECS permissions to the role for the container. I'd like to pare this down in the future.

Regarding the role file layout. To avoid having so many "main.yml" files open in my editor at once, I've been symlinks in the main role directory to reference files. This seems clearer to me than the standard subdirectory layout for small roles and it remains compatible with the standard layout.

## Instructions
You will need:
* local Docker install
* recent of boto3 (tested with 1.8.8)
* recent version of ansible (tested with 2.5.2)


Follow the instructions from [Docker version of ARSnova](https://github.com/thm-projects/arsnova-docker) to generate the Docker images that will be uploaded to AWS. I had an older version of Docker and ended up downgrading the docker-compose.yml file to version "2.0" from "3.4" and removing the volumes > data > name value to work in my installation. you should end up with 3 docker images (arsnova/arsnova-couchdb, arsnova/arsnova-webapp, arsnova/arsnova-nginx)

I stash my Ansible become (sudo) password in an environment variable so I don't have to constantly re-enter it. If you don't want to do this, remove/comment the **ansible_become_password** line in **hosts**. Otherwise, use the following bash shell function to setup up the env variable.
```
function ansiblepw {
  echo -n "become(sudo): "
  read -s CTPW
  export APV1=$(echo -n "$CTPW" |base64)
  echo
  unset CTPW
}
```
Copy the example-config.yml file to config.yml and edit it for your configuration. At a minimum you should edit the aws_account_id, ssh_keyname, and ssh_cidr_list.

Once configured, you should be able to run `ansible-playbook aws-docker-vpc.yml -b` to install in AWS. This creates a VPC with external access for anyone to HTTP on port 80 and SSH access from the addresses you specify in the config.yml file. One EC2 container instance (t2.small) is created to host the application containers. It may be possible to downsize this to a micro, but I need to take a look at resource usage for proper sizing.

Use `ansible-playbook rm-aws-docker-vpc.yml -b` to uninstall. **WARNING:** The Ansible ECS modules do not seem to properly deregister the container instances so the ECS cluster deletion does not work. You will have to manually deregister instances and remove the cluster (From the AWS ECS Clusters page).
