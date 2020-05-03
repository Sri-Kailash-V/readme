DP-SD3 Docker Setup

PREREQUISITES

1.Install docker -> install

2.send a IT ops request for disabling your system Firewall, this is needed since firewall will not allow port forwarding and mapping which is required for our setup.

3.Make sure you have bitbucket access, The following setup requires authentication with bitbucket to fetch the repos

4.create a directory in C:/ or Documents/ for cloning the sd3_documentation repo
Example:- C:\Users\Ashish\Documents\sd3_documentation

5.once the directory structure is created, go sd3_documentation and clone the below repo
	git clone https://bit_bucket_username@bitbucket.org/pramata/sd3_api_documentation.git

6.cd sd3_documentation

7.Checkout master



Setting up dp and sd3 related services
1.Cd to “sd3_documentation/sd3_api_documentation/Scripts”
	a.cd to “resources/metadata_digitization_api” and add the updated resources form metadata_digitization_api repo with gemfile,gemfile.lock
	b.cd to “resources/metadata_digitization_ui” and add the updated resources form metadata_digitization_ui repo with gemfile,gemfile.lock,package.json,yarn.lock

2.sh script_env_setup

3.this will ask for GIT ID and password, Make sure the credentials are correct.

4.Please provide network name -> here please enter a network name which will be used for connecting all the docker containers/services.
	Note:- If the network with driver as bridge is already present then the it will ignore the new name and use the existing network.

5.enter yes when it asks for ->  Please provide Input as "yes" If you are Sure ?? 
	If you say yes it will delete the existing db container and create a new db container
	Note :- If you already have a db container for please say no.

6.for every service it will for following details
	1.Image name
	2.Container name
	3.port number
	4.base directory of repo for non windows host
Example:-
	1.Please provide sd3_api image name: sd3_api_image
	2.Please provide sd3_api container name : sd3_api_container
`	3.Please provide sd3_api port number : 3300
For Non windows host
	4.Please provide sd3_api base : /Users/workspace/sd3_api

Note1: if container name already exists then the old container will be replaced, else it will create a new container.
Note2: If the default values are present the prompt message will show the last values entered by the user, Please use the default values in this case always.
Note3:- Also you can see the config details for each of the services in the below mentioned file
user_data_file=~/local_profile/.temp_user_data_clone


7.go to each container and start the server (IF script fails o open them automaically)
	docker exec -it container_name bash
	Cd to repo_directory example :- if its dp container then cd delivery_platform
	Note1:- Please enter “Yes” when it ask to run .rvmrc
	Note2:- If the Note1 fails then please manually run bundle install and rake db:migrate
	Start server by rails server -p port_number -b 0.0.0.0


Following Changes are done automatically by he script However you might verify them in case any issue 
8. Changes to the code 
Changes in sd3_ui
1.changes application_controller.rb
	Change line in def set_digitization_host
	@api_http_helper = Pramata::Api::Sd3ApiHelper.new(@tenant,{:data=>{:app_user_id => session[:app_user_id]}},"localhost:3500")

2.changes in authentication_controller.rb
def check_type_and_authenticate
return true
end

 def authenticate 
    if session[:app_user_id].nil?
      session[:tenant_name] = 'earth'
      session[:app_user_id] = 7
    else
      set_digitization_host
    end
  end


Changes in sd3_api

1.changes in application controller
 def security_check
Return true
end


Changes in dp
1.changes in sd3_tenant_identifier.rb
def self.sd3_url
    return “http://earth.lvh.me:place_holder_for_sd3_ui_port_number”
end

Changes in metadata_digitization_ui
1.def get_tenant
		return Tenant.first
	end

2.def authenticate
	if session[:app_user_id].nil?
      cookies[:redirect_url] = request.original_url
      tenant = get_tenant
      tenant_dp_url = tenant.dp_url
      # redirect_to "#{tenant_dp_url}/metadata_auth/token" and return		**comment this line **	
    else
      set_digitization_host
    end
  end
