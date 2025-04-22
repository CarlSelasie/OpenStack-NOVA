# OpenStack-NOVA
Task 1- Heat Deployment 

OpenStack Heat Recipe (Run all steps in root@controller)
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
<pre><code class="language-sql">openstack role add --project service --user heat admin </code></pre>
#This command provides no output.
#Execute in root@controller

•	Create the heat and heat-cfn service entities:
<pre><code class="language-sql">openstack service create --name heat \
  --description "Orchestration" orchestration </code></pre>

<pre><code class="language-sql">openstack service create --name heat-cfn \
  --description "Orchestration"  cloudformation </code></pre>
#Execute in root@controller

3.	Create the Orchestration service API endpoints:
<pre><code class="language-sql">openstack endpoint create --region RegionOne \
  orchestration public http://controller:8004/v1/%\(tenant_id\)s </code></pre>


<pre><code class="language-sql">openstack endpoint create --region RegionOne \
  orchestration internal http://controller:8004/v1/%\(tenant_id\)s </code></pre>

<pre><code class="language-sql">openstack endpoint create --region RegionOne \
  orchestration admin http://controller:8004/v1/%\(tenant_id\)s </code></pre>



<pre><code class="language-sql">openstack endpoint create --region RegionOne \
  cloudformation public http://controller:8000/v1 </code></pre>

<pre><code class="language-sql">openstack endpoint create --region RegionOne \
  cloudformation internal http://controller:8000/v1 </code></pre>

<pre><code class="language-sql">openstack endpoint create --region RegionOne \
  cloudformation admin http://controller:8000/v1</code></pre>



4.	Orchestration requires additional information in the Identity service to manage stacks. To add this information, complete these steps:
•	Create the heat domain that contains projects and users for stacks:
<pre><code class="language-sql">openstack domain create --description "Stack projects and users" heat </code></pre>

•	Create the heat_domain_admin user to manage projects and users in the heat domain:
<pre><code class="language-sql">openstack user create --domain heat --password-prompt heat_domain_admin </code></pre>
User Password: plungers4900
Repeat User Password: plungers4900

•	Add the admin role to the heat_domain_admin user in the heat domain to enable administrative stack management privileges by the heat_domain_admin user:
<pre><code class="language-sql">openstack role add --domain heat --user-domain heat --user heat_domain_admin admin </code></pre>

•	Create the heat_stack_owner role:
<pre><code class="language-sql">openstack role create heat_stack_owner </code></pre>

•	Add the heat_stack_owner role to the demo project and user to enable stack management by the demo user:
<pre><code class="language-sql">openstack role add --project myproject --user myuser heat_stack_owner </code></pre>
#changes made from original demo to myproject and demo to myuser. verify

•	Create the heat_stack_user role:
<pre><code class="language-sql">openstack role create heat_stack_user</code></pre>


Install and configure components
1.	Install the packages:
<pre><code class="language-sql">apt-get install heat-api heat-api-cfn heat-engine</code></pre>
#Execute in root@controller

2.	Configuration for Heat

<pre><code class="language-sql"> mv /etc/heat/heat.conf /etc/heat/heat.conf.orig
mkdir -p ~/os-template-files
nano ~/os-template-files/heat.conf </code></pre>
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

<pre><code class="language-sql">cp ~/os-template-files/heat.conf /etc/heat/heat.conf
chown heat:heat /etc/heat/heat.conf
chmod 640 /etc/heat/heat.conf </code></pre>



3.	Populate the Orchestration database:
<pre><code class="language-sql"> su -s /bin/sh -c "heat-manage db_sync" heat </code></pre>


Finalize installation
1.	Restart the Orchestration services:
<pre><code class="language-sql">service heat-api restart
service heat-api-cfn restart
service heat-engine restart </code></pre>


Install Heat Plugin on Horizon
<pre><code class="language-sql">add-apt-repository cloud-archive:caracal
apt update
apt install openstack-dashboard python3-heat-dashboard
systemctl restart apache2</code></pre>
then recheck your horizon GUI



Task 2 
Magnum service deployment
https://docs.openstack.org/magnum/2025.1/install/install-ubuntu.html

1.	Setup mysql database
<pre><code class="language-sql">mysql</code></pre>

2.	In the mysql prompt run 
<pre><code class="language-sql">CREATE DATABASE magnum;
GRANT ALL PRIVILEGES ON magnum.* TO 'magnum'@'localhost' IDENTIFIED BY 'plungers4900';
GRANT ALL PRIVILEGES ON magnum.* TO 'magnum'@'%' IDENTIFIED BY 'plungers4900';
exit;</code></pre>

3.	Create magnum and assign roles 
<pre><code class="language-sql">openstack user create --domain default --password-prompt magnum </code></pre>

Enter password: plungers4900
<pre><code class="language-sql">openstack role add --project service --user magnum admin </code></pre>

4.	Create the Magnum service
<pre><code class="language-sql">openstack service create --name magnum \
  --description "OpenStack Container Infrastructure Management Service" \
container-infra </code></pre>

5.	Create API endpoints
<pre><code class="language-sql">openstack endpoint create --region RegionOne \
container-infra public http://controller:9511/v1 </code></pre>

<pre><code class="language-sql">openstack endpoint create --region RegionOne \
container-infra internal http://controller:9511/v1 </code></pre>

<pre><code class="language-sql">openstack endpoint create --region RegionOne \
container-infra admin http://controller:9511/v1 </code></pre>


6.	Create Magnum domain and domain admin
<pre><code class="language-sql">openstack domain create --description "Owns users and projects created by \ magnum" magnum </code></pre>

<pre><code class="language-sql">openstack user create --domain magnum --password-prompt magnum_domain_admin </code></pre>

Enter password: plungers4900
<pre><code class="language-sql">openstack role add --domain magnum --user-domain magnum –user \  magnum_domain_admin admin</code></pre>

7.	Install package
<pre><code class="language-sql">DEBIAN_FRONTEND=noninteractive apt-get install magnum-api magnum-conductor python3-magnumclient</code></pre>

8.	Apply Configuration
mv /etc/magnum/magnum.conf   /etc/magnum/magnum.conf.orig

<pre><code class="language-sql">mkdir -p ~/os-template-files
nano ~/os-template-files/magnum.conf </code></pre>


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

<pre><code class="language-sql">cp ~/os-template-files/magnum.conf   /etc/magnum/magnum.conf
chown magnum:magnum   /etc/magnum/magnum.conf
chmod 640    /etc/magnum/magnum.conf  </code></pre>




9.	Populate database
<pre><code class="language-sql">su -s /bin/sh -c "magnum-db-manage upgrade" magnum </code></pre>

10.	Finalize installation
<pre><code class="language-sql">service magnum-api restart
service magnum-conductor restart </code></pre>


To enable magnum on the Horizon dashboard
<pre><code class="language-sql">apt install python3-magnum-ui </code></pre>


After installation, restart the Horizon web server
<pre><code class="language-sql">systemctl restart apache2 </code></pre>

Then log back in to Horizon if you get an error, run this command
<pre><code class="language-sql">nano /etc/openstack-dashboard/local_settings.py </code></pre>

Look for this line:
COMPRESS_OFFLINE = True

Change it to:
COMPRESS_OFFLINE = False

This will disable offline compression and let Horizon dynamically compress assets.

Then restart Apache again:
<pre><code class="language-sql">systemctl restart apache2 </code></pre>



Task 3
1.	Setup mysql database
<pre><code class="language-sql">mysql</code></pre>

2.	In the mysql prompt run 
<pre><code class="language-sql">CREATE DATABASE cinder;

GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'plungers4900';

GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'plungers4900';

EXIT;</code></pre>

 
<pre><code class="language-sql">. ~/.bash_aliases</code></pre>  # Loads admin environment for root user

 
<pre><code class="language-sql">openstack user create --domain default --password-prompt cinder</code></pre>

# Enter: plungers4900

<pre><code class="language-sql">openstack role add --project service --user cinder admin</code></pre>

 
<pre><code class="language-sql">openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
 
openstack endpoint create --region RegionOne volumev3 public http://controller:8776/v3/%\(project_id\)s

openstack endpoint create --region RegionOne volumev3 internal http://controller:8776/v3/%\(project_id\)s

openstack endpoint create --region RegionOne volumev3 admin http://controller:8776/v3/%\(project_id\)s</code></pre>

 
<pre><code class="language-sql">apt install cinder-api cinder-scheduler</code></pre>

 
STEP 5: Apply Cinder Configuration  
 
<pre><code class="language-sql">mv /etc/cinder/cinder.conf /etc/cinder/cinder.conf.orig

cp ~/os-template-files/cinder.conf /etc/cinder/cinder.conf

chown root:cinder /etc/cinder/cinder.conf

chmod 640 /etc/cinder/cinder.conf</code></pre>

 
[database]

connection = mysql+pymysql://cinder:plungers4900@controller/cinder
 
[DEFAULT]

transport_url = rabbit://openstack:plungers4900@controller

auth_strategy = keystone

my_ip = 192.168.122.100
 
[keystone_authtoken]

www_authenticate_uri = http://controller:5000

auth_url = http://controller:5000

memcached_servers = controller:11211

auth_type = password

project_domain_name = Default

user_domain_name = Default

project_name = service

username = cinder

password = plungers4900
 
[oslo_concurrency]

lock_path = /var/lib/cinder/tmp

 
<pre><code class="language-sql">su -s /bin/sh -c "cinder-manage db sync" cinder</code></pre>

 
STEP 7: Link Nova to Cinder
Add this block to /etc/nova/nova.conf:
 
[cinder]

os_region_name = RegionOne

 
service nova-api restart

service cinder-scheduler restart

service apache2 restart

 
STEP 9: Verify Cinder
 
openstack volume service list

openstack catalog list | grep volume

 



