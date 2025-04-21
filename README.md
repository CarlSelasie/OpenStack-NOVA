# OpenStack-NOVA
Heat Deployment 
Recipe - Deploying OpenStack Nova Using the Nova Function 


Open Stack Heat Recipe (Run all steps in root@controller)
Prerequisite

Before you install and configure Orchestration, you must create a database, service credentials, and API endpoints. Orchestration also requires additional information in the Identity service.
1.	To create the database, complete these steps:
•	Use the database access client to connect to the database server as the root user:
<pre><code class="language-sql">mysql</code></pre>                      #Execute in root@controller

•	Create the heat database:
<pre><code class="language-sql"> CREATE DATABASE heat; </code></pre>

<pre><code class="language-sql"> CREATE DATABASE heat; GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' IDENTIFIED BY 'plungers4900'; GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' IDENTIFIED BY 'plungers4900'; EXIT; </code></pre>


•	Grant proper access to the heat database:
<pre><code class="language-sql">GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' \
  IDENTIFIED BY 'plungers4900';
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' \
  IDENTIFIED BY 'plungers4900';
exit </code></pre>

2.	To create the service credentials, complete these steps:
•	Create the heat user:
<pre><code class="language-sql">openstack user create --domain default --password-prompt heat </code></pre>
#User Password:plungers4900
Repeat User Password:plungers4900
#Execute in root@controller

•	Add the admin role to the heat user:
openstack role add --project service --user heat admin
#This command provides no output.
#Execute in root@controller

•	Create the heat and heat-cfn service entities:
openstack service create --name heat \
  --description "Orchestration" orchestration

openstack service create --name heat-cfn \
  --description "Orchestration"  cloudformation
#Execute in root@controller

3.	Create the Orchestration service API endpoints:
openstack endpoint create --region RegionOne \
  orchestration public http://controller:8004/v1/%\(tenant_id\)s


openstack endpoint create --region RegionOne \
  orchestration internal http://controller:8004/v1/%\(tenant_id\)s

openstack endpoint create --region RegionOne \
  orchestration admin http://controller:8004/v1/%\(tenant_id\)s






openstack endpoint create --region RegionOne \
  cloudformation public http://controller:8000/v1

openstack endpoint create --region RegionOne \
  cloudformation internal http://controller:8000/v1

openstack endpoint create --region RegionOne \
  cloudformation admin http://controller:8000/v1



4.	Orchestration requires additional information in the Identity service to manage stacks. To add this information, complete these steps:
•	Create the heat domain that contains projects and users for stacks:
openstack domain create --description "Stack projects and users" heat

•	Create the heat_domain_admin user to manage projects and users in the heat domain:
openstack user create --domain heat --password-prompt heat_domain_admin
User Password: plungers4900
Repeat User Password: plungers4900

•	Add the admin role to the heat_domain_admin user in the heat domain to enable administrative stack management privileges by the heat_domain_admin user:
openstack role add --domain heat --user-domain heat --user heat_domain_admin admin

•	Create the heat_stack_owner role:
openstack role create heat_stack_owner

•	Add the heat_stack_owner role to the demo project and user to enable stack management by the demo user:
openstack role add --project myproject --user myuser heat_stack_owner
#changes made from original demo to myproject and demo to myuser. verify

•	Create the heat_stack_user role:
openstack role create heat_stack_user


Install and configure components
1.	Install the packages:
apt-get install heat-api heat-api-cfn heat-engine
#Execute in root@controller

2.	Configuration for Heat

mv /etc/heat/heat.conf /etc/heat/heat.conf.orig
mkdir -p ~/os-template-files
nano ~/os-template-files/heat.conf
[DEFAULT]
transport_url = rabbit://openstack:plungers4900@controller
heat_metadata_server_url = http://controller:8000
heat_waitcondition_server_url = http://controller:8000/v1/waitcondition
stack_domain_admin = heat_domain_admin
stack_domain_admin_password = plungers4900
stack_user_domain_name = heat
 
[database]
connection = mysql+pymysql://heat:plungers4900@controller/heat
 
[keystone_authtoken]
auth_type = password
auth_url = http://controller:5000
www_authenticate_uri = http://controller:5000
memcached_servers = controller:11211
project_domain_name = Default
user_domain_name = Default
project_name = service
username = heat
password = plungers4900
 
[trustee]
auth_type = password
auth_url = http://controller:5000
username = heat
password = plungers4900
user_domain_name = Default
 
[clients_keystone]
auth_uri = http://controller:5000

Then you save

cp ~/os-template-files/heat.conf /etc/heat/heat.conf
chown heat:heat /etc/heat/heat.conf
chmod 640 /etc/heat/heat.conf



3.	Populate the Orchestration database:
su -s /bin/sh -c "heat-manage db_sync" heat


Finalize installation
1.	Restart the Orchestration services:
service heat-api restart
service heat-api-cfn restart
service heat-engine restart


Install Heat Plugin on Horizon
add-apt-repository cloud-archive:caracal
apt update
apt install openstack-dashboard python3-heat-dashboard
systemctl restart apache2
then recheck your horizon GUI



Task 2 – Magnum service deployment
https://docs.openstack.org/magnum/2025.1/install/install-ubuntu.html

1.	setup mysql database
mysql

2.	In the mysql prompt run 
CREATE DATABASE magnum;
GRANT ALL PRIVILEGES ON magnum.* TO 'magnum'@'localhost' IDENTIFIED BY 'plungers4900';
GRANT ALL PRIVILEGES ON magnum.* TO 'magnum'@'%' IDENTIFIED BY 'plungers4900';
exit;

3.	Create magnum and assign roles 
openstack user create --domain default --password-prompt magnum

Enter password: plungers4900
openstack role add --project service --user magnum admin

4.	Create the Magnum service
openstack service create --name magnum \
  --description "OpenStack Container Infrastructure Management Service" \
container-infra

5.	Create API endpoints
openstack endpoint create --region RegionOne \
container-infra public http://controller:9511/v1

openstack endpoint create --region RegionOne \
container-infra internal http://controller:9511/v1

openstack endpoint create --region RegionOne \
container-infra admin http://controller:9511/v1


6.	Create Magnum domain and domain admin
openstack domain create --description "Owns users and projects created by \ magnum" magnum

openstack user create --domain magnum --password-prompt magnum_domain_admin

Enter password: plungers4900
openstack role add --domain magnum --user-domain magnum –user \  magnum_domain_admin admin

7.	Install package
DEBIAN_FRONTEND=noninteractive apt-get install magnum-api magnum-conductor python3-magnumclient

8.	Apply Configuration
mv /etc/magnum/magnum.conf   /etc/magnum/magnum.conf.orig


mkdir -p ~/os-template-files
nano ~/os-template-files/magnum.conf

 
             Then p
             [DEFAULT]
transport_url = rabbit://openstack:plungers4900@controller
host = controller
state_path = /var/lib/magnum
debug = False
log_dir = /var/log/magnum
use_syslog = False

[api]
host = 0.0.0.0
port = 9511

[certificates]
cert_manager_type = barbican

[cinder_client]
region_name = RegionOne

[database]
connection = mysql+pymysql://magnum:plungers4900@controller/magnum

[keystone_authtoken]
auth_type = password
auth_url = http://controller:5000
www_authenticate_uri = http://controller:5000
memcached_servers = controller:11211
project_domain_name = Default
user_domain_name = Default
project_name = service
username = magnum
password = plungers4900

[trust]
trustee_domain_name = magnum
trustee_domain_admin_name = magnum_domain_admin
trustee_domain_admin_password = plungers4900
trustee_keystone_interface = public

[oslo_messaging_notifications]
driver = messaging
             aste this
[DEFAULT]
transport_url = rabbit://openstack:plungers4900@controller
host = controller
state_path = /var/lib/magnum
debug = False
log_dir = /var/log/magnum
use_syslog = False

[api]
host = 0.0.0.0
port = 9511

[certificates]
cert_manager_type = barbican

[cinder_client]
region_name = RegionOne

[database]
connection = mysql+pymysql://magnum:plungers4900@controller/magnum

[keystone_authtoken]
auth_type = password
auth_url = http://controller:5000
www_authenticate_uri = http://controller:5000
memcached_servers = controller:11211
project_domain_name = Default
user_domain_name = Default
project_name = service
username = magnum
password = plungers4900

[trust]
trustee_domain_name = magnum
trustee_domain_admin_name = magnum_domain_admin
trustee_domain_admin_password = plungers4900
trustee_keystone_interface = public

[oslo_messaging_notifications]
driver = messaging
Save and Exit

Run this After in root controller

cp ~/os-template-files/magnum.conf   /etc/magnum/magnum.conf
chown magnum:magnum   /etc/magnum/magnum.conf
chmod 640    /etc/magnum/magnum.conf




9.	Populate database
su -s /bin/sh -c "magnum-db-manage upgrade" magnum

10.	Finalize installation
service magnum-api restart
service magnum-conductor restart


To enable magnum on the Horizon dashboard
apt install python3-magnum-ui


After installation, restart the Horizon web server
systemctl restart apache2

Then log back in to Horizon if you get an error, run this command
nano /etc/openstack-dashboard/local_settings.py

Look for this line:
COMPRESS_OFFLINE = True

Change it to:
COMPRESS_OFFLINE = False

This will disable offline compression and let Horizon dynamically compress assets.

Then restart Apache again:
systemctl restart apache2




