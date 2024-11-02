## Ansible Dynamic Assignment(Include) & Community Roles
Last 2 projects have already equipped us with some knowledge and skills on Ansible, so we can perform configurations using *playbooks*, *roles* and *imports*. Now we will continue configuring our UAT servers learning and practicing new Ansible concepts and modules.

In this project we will introduce dynamic assignments by using *include* module.
Now the different between Static and Dynamic Assignment is Static assignment uses *import* while the Dynamic Assignment uses *include*

When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute *site.yml* playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.

# Introducing Dynamic Assignment Into Our structure
1. In your *https://github.com/<your-name>/ansible-config-mgt* GitHub repository start a new branch and call it *dynamic-assignments*.
2. Create a new folder, name it *dynamic-assignments*. Then inside this folder, create a new file and name it *env-vars.yml*. We will instruct *site.yml* to include this playbook later. For now, let us keep building up the structure. ![reference image](/Pictures/pic1.PNG)
**NOTE**: Depending on what method you used in the previous project you may have or not have *roles* folder in your GitHub repository - if you used *ansible-galaxy*, then *roles* directory was only created on your *Jenkins-Ansible* server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case - it is up to you. ![reference image](/Pictures/pic2.PNG)

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as **servername**, **ip-address** etc., we will need a way to set values to variables per specific environment.

3. For this reason, we will now create a folder to keep each environment's variables file. Therefore, create a new folder *env-vars*, then for each environment, create new **YAML** files which we will use to set variables. Your layout should look like this ![reference image](/Pictures/pic3.PNG) ![reference image](/Pictures/pic4.PNG)
4. Now paste the instruction below into the *env-vars.yml* file. ![reference image](/Pictures/pic5.PNG)
5. Notice 3 things to notice here:
1. We used *include_vars* syntax instead of *include*, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the *include* module is deprecated and variants of *include_* must be used. These are:
- include_role
- include_tasks
- include_vars
In the same version, variants of **import** were also introduces, such as:
- import_role
- import_tasks
2. We made use of a special variables *{{ playbook_dir }}* and *{{ inventory_file }}*. *{{ playbook_dir }}* will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. *{{ inventory_file }}* on the other hand will dynamically resolve to the name of the inventory file being used, then append *.yml* so that it picks up the required file within the *env-vars* folder.
3. We are including the variables using a loop. *with_first_found* implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

# Update site.yml with dynamic assignments
1. Update site.yml file to make use of the dynamic assignment usind the code below ![reference image](/Pictures/pic6.PNG)

# Community Roles
Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.
1. Inside *roles* directory  create a new MySQL role with  *ansible-galaxy install geerlingguy* 
2. then  *install mysql* and rename the folder to *mysql* with the following commands: *ansible-galaxy install geerlingguy.mysql* and *mv geerlingguy.mysql/ mysql* ![reference image](/Pictures/pic7.PNG)
3. Read *README.md* file, and edit roles configuration to use correct credentials for MySQL required for the tooling website. ![reference image](/Pictures/pic9.PNG)
4. Inside *role* directory create a new *nginx* and *apache* role with *ansible-galaxy install geerlingguy* using the following commands *ansible-galaxy install geerlingguy.nginx* and move it to a new folder *mv geerlingguy.nginx/ nginx* do the same for apache *ansible-galaxy install geerlingguy.apache* then move it too *mv geerlingguy.apache/ apache* ![reference image](/Pictures/pic8.PNG)
**Important Hints**:
1. Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one - this is where you can make use of variables.
2. Declare a variable in *defaults/main.yml* file inside the Nginx and Apache roles. Name each variables *enable_nginx_lb* and *enable_apache_lb* respectively.!
3. Set both values to false like this *enable_nginx_lb*: false and *enable_apache_lb: false.[reference image](/Pictures/pic10.PNG) ![reference image](/Pictures/pic11.PNG)
4. Create a file in *static-assignment* and name it *loadbalancer.yml* and paste the follwing commands ![reference image](/Pictures/pic12.PNG) ![reference image](/Pictures/pic13.PNG)
5. Update  site.yml files  with the codes below ![reference image](/Pictures/pic14.PNG)
6. Now you can make use of *env-vars\uat.yml* file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true. Using the below code ![reference image](/Pictures/pic15.PNG). The same must work with *apache* LB, so you can switch it by setting respective environmental variable to *true* and other to *false*.
7. To test this, you can update inventory for each environment and run Ansible against each environment. Uisng this command *ansible-playbook -i inventory/uat.yml playbooks/site.yml* ![reference image](/Pictures/pic16.PNG)

**Congratulations**: We have learned and practiced how to use Ansible configuration management tool to prepare UAT environment for Tooling web solution.