# Client SDR service (provision environment for sdr)


This project is a service to provide an environment for client as service based on project id, project name, email address, and zone of SDR, the environment will contain the needed bigdata services in nifi as process group, the environment will have a relationship between each zone of SDR including exec, data, int, and gin

## Usage

### I) Run apps with python script directly from the machine

We choose for provision environment chosen zone (SDR (nifi) ), project id and project name and email then we run this script `provision_env.py` with 4 parameters as follow:

The script should be run from (SDR admin machine prod) because it is the only one that contains the python package `nipyapi` (nipyapi is the api that we use to communicate with nifi)
`python provision_env.py -p project_id -pn project_name -e e-mail -z zone
` <br>

`p`: project id of environment <br>
`pn`: project name or name of the process group that will be created <br>
`e`: email (orange email or any email allowed to the orange network)
`z`: zone of SDR nifi (gin, internet, exec, and data)

- Path of the script (in role client SDR service in GitLab) 

        path of `provision_env.py` = client_sdr_service/files/provisioning_service/provision_env.py
        
**conclusion**:  To run this script in 4 zones as one time, we will use Ansible 
###II) Run apps with ansible directly 

Run ansible-playbook  directly from machine (admin prod machine prsdradmnifi0101-0102-0103)

##### This is the steps :
2.1 - Git clone admin task project <br>
    
```git clone https://url_orange/Ansible_admin_tasks.git```

2.2 - create `extravars.json` that contain this specific Infos  inside admin task project: <br>

        {
            "project_id": "id_project",
            "admin_email": "firstname.lastname@orange.com",
            "pg_name": "name of the project"
        }

2.3 - Run Ansible Playbook<br>

```ansible-playbook -i Inventory/nifi_01.pr.admin.diod.fe --private-key /home/nifi/.ssh/deploy_key  -e "@extravars.json" provision_env.yml```<br>

 #####Command description<br> 
 
```nifi_01.pr.admin.diod.fe ```: inventory that we use to deploy script across admin nodes<br>
```provision_env.yml```: playbook that we used to run all tasks inside this role

###III) Run apps from https endpoint (directly in SDR)

This normally the first original version to activate the environment from SDR admin, first we begin sending a request from the postman to https endpoint in SDR gin after the request will be routed to the SDR admin to find pgprovi route to launch the ansible that I describe above.

1) request to add on postman : <br>

        https://pr-diod-sdrginin01fe.itn.intraorange:10006/v2/service_instances/diod.sdraas.pgprovi/

2) body to be added on postman on mode 'PUT'

         {
           "service_id": "diod.sdraas.pgprovi",
           "context": {
             "project_id": "id_project"
           },
           "parameters": {
             "admin_email": "e-mail",
             "pg_name": "name_project"
           },
           "maintenance_info": {
             "version": "2.1.1+abcdef"
           }
         }
         
     ##### Screenshot to show how it is the request in postman
     N.B : you should configure ssl context and proxy to enable requesting on postamn with this request 
    ![screenshot postman request](/home/hamza/Pictures/Screenshot from 2020-03-16 16-33-27.png)

#### JSON Description

```id_project```(mandatory): the convention always begins with 'p' and put after what you want without any space.<br>

```name_project```(mandatory): write any name but for now without any space

```email```(mandatory): write an orange email or any email that is allowed to our network.

#### Https Response 

JSON that contain  URLs of the created environment directly and also the response's status  (200,201,400)<br>

```200``` : environment already created ==> "ALREADY CREATED OPERATION"<br>
```201``` : environment created ==> "CREATE OPERATION" <br>
```400``` : bad request, it will contain specific message as errors for debugging 

# Delete provisioned environment 

Sometimes customers will want to delete their older environment and create another, or sometimes they will create the wrong one and want to delete it, for that we create a service that deletes this created environment on the fly, we used for that also nipyapi to communicate with nifi, so to Delete environment we use for now directly this python command : 

        python delete_env.py -p project _id -pn project_name -z zone 

- Path of delete_env.py : 

        client_sdr_service/files/provisioning_service/delete_env.py
    
**Important** : 
project id should be unique, if not the delete_env.py will delete just identical connection and also the update attribute and ControlRate that contains the name of project id.
Normally, we use project id and pg name to delete process group from the canvas but for connection and update attribute always still using project_id.
ControlRate processor should always have name as follow : (controlRate_project_id) to be unique
 #### still working adding delete as service and send it from https endpoint
