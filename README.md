# IBM Assignment - Ansible
## Prerequisites
- Control Node (Ubuntu 22.04)
- Managed Node (Ubuntu 22.04) - Optional but recommended
- Private and public SSH key
- SSH (Port 22) and MySQL (Port 3306) enabled on managed node
- Network connectivity between control and managed nodes

## Ansible Task Summary
1. Create inventory file containing hostname of nodes hosting the MySQL DB
2. Create file containing variables for deployment
3. Create list of tasks to be executed
    - Install mysql-server using apt module
    - Install python3-mysqldb using apt module (required in order to use Ansible built-in mysql modules)
    - Start and enable mysql service
    - Create new database for new db user (I assumed the db user would be given privilege only to selected dbs. Database name is configurable in the variables file)
    - Create new database user and allow connection from all sources (Username and password is configurable in the variables file)
    - Connect to database using newly created user and password to verify status
        - Note that this checkpoint does not rule out the possibility of connectivity issues caused by other components such as firewall since the check is performed within the remote machine itself. I may look to improve this in future by running the check from control node instead in future
4. Create playbook

## Setting Up - Control Node
1. [Installing Ansible on Control Node](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu)
2. Generate new SSH key pair: ```ssh-keygen -t ed25519 -C "ansible"```
    - When prompted on the file location, enter "~/.ssh/ansible"
    - Leave passphrase empty
3. Copy the SSH public key: ```cat ~/.ssh/ansible.pub```. We will require this later on for the managed node

## Setting Up - Managed Node
> In the case where the DB is intended to be installed locally, the control node would double as the managed node. This means that this section will be executed on the control node instead.
1. Check if Python is installed: ```python3 --version```
2. If not, install Python by following the instructions [here](https://phoenixnap.com/kb/how-to-install-python-3-ubuntu)
3. Authorize the public key from the newly generated key pair: ```nano ~/.ssh/authorized_keys``` and append the public key into the file. Save and close the file.

## Configuring Ansible Variables
1. Clone Git repository and navigate into working directory
2. Configure the DB host address from ```inventory``` file
    - In the case where the DB is intended to be installed on the control node, leave the address as ```127.0.0.1```
3. Open the variable file located at ```roles/database/defaults/main.yml```. From here you can modify several variables including:
    - Database name (```db_add_user_db```)
    - Username for newly created user (```db_add_user_username```)
    - Password for newly created user (```db_add_user_password```)

## Checkpoint
1. Verify if the connection between control and managed node is up by running the following from the control node: ```ansible all  --key-file ~/.ssh/ansible -i inventory -m ping```
    - A success message should be returned if the connection is up

## Running Playbook
1. Within the working directory, run the Ansible playbook: ```ansible-playbook -i inventory 00-database.yml```
    - For verbose: ```ansible-playbook -i inventory 00-database.yml -vvv```
