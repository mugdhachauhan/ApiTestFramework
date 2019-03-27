# Postman exercise
1. Link to install postman

	`https://www.getpostman.com/downloads/`

2. Account subdomain for testing: 

	`https://z3ntestframework.zendesk.com`

3. API under test : `/api/v2/users.json`

4. Request body : 
	```
	{
	"user": 
		{
			"name": "U1", 
			"email": "U1@example.org"
		}
	}
	```
			
5. Code snippet generated by Postman:

	```
	require 'uri'
	require 'net/http'

	url = URI("https://z3ntestframework.zendesk.com/api/v2/users.json")

	http = Net::HTTP.new(url.host, url.port)
	http.use_ssl = true
	http.verify_mode = OpenSSL::SSL::VERIFY_NONE

	request = Net::HTTP::Post.new(url)
	request["content-type"] = 'application/json'
	request["authorization"] = 'Basic bWNoYXVoYW5AemVuZGVzay5jb206YWRtaW4='
	request["cache-control"] = 'no-cache'
	request["postman-token"] = '5508c333-b1eb-db66-da5a-c1511044c28b'
	request.body = "{\n\"user\": \n\t{\n\t\t\"name\": \"U1\", \n\t\t\"email\": \"U1@example.org\"\n\t}\n}\n"

	response = http.request(request)
	puts response.read_body
	
	```
	
# STEP 1 : Set up new project

1. Create a new project in your IDE or use any online ruby compliler like Repl (https://repl.it/languages/ruby)

2. Create folders as follows
	- config
	- spec
	   - features
	   - fixtures
	   - support	
	   
3. Create new file
 	- Features -> original_test.rb
	
4. Copy code snippet from postman to original_test.rb

5. Following lines of code are required to run postman test in ruby. Please add them if its missing.

	copy `require 'openssl'` at the top of the file
	
	copy `http.use_ssl = true` after `Net::HTTP.new`
	
6. To remove postman specific settings and tokens delete follwoing code from your test as they are NO longer required

	`request["postman-token"] = xxxxxxxxxxx`
	
	`request["cache-control"] = 'no-cache'	`

7. Change user name and email for now

	` name: yourname_anynumber` example: `mugdhac577`
	
	`email: yourname_anynumber@example.org` example: `mugdhac666@example.org`

8. Run test from the project path

	If using terminal to run test 

	```
	cd spec/features
	ruby original_test.rb
	```
	
	If using Repl.it online compiler
	` require_relative 'spec/features/original_test.rb`
	
	
	` !!! Test Should pass !!! `
	
Note : if test fails with error `email is already being used by another user` please change user email address and rerun the test case	


# STEP 2 : Add description and import files

1. Create a duplicate copy of  `original_test.rb` and name it as `refactor_test.rb`
	
	`refactor_test.rb` will be now used to refactor the test code and can be later compared with original_test.rb

2. Follow presenter instructions to add explanation/documentation to your test 

3. Create new files under follwing folders

	config -> 
		config.yml
	
	fixtures -> 
		endpoints.rb
	
	support ->
		configClient.rb,	 
		helper.rb, 	
		loadConfig.rb
		
4. Modify `refactor_test.rb` to import all required files

	```
	require 'json'
	require '../../spec/support/loadConfig'
	require '../../spec/fixtures/endpoints'
	require '../../spec/support/helper'
	require '../../spec/support/configClient'
	```
	
	Note : if require doesn’t work try require_relative

4. Modify `refactor_test.rb` to add assertions as follows

	```
	# assert if response code is not 201
	if response.code != '201'
	  raise "Invalid response code"
	end
	
	```
5. Run `refactor_test.rb` 

If using repl.it make sure to change file name in `main.rb`

	`require_relative 'spec/features/original_test.rb`

	`!!! Test will fail with error : email is already being used by another user !!!`

So now its time to create some test data


# STEP 3 : Create Test data

1. To create test data copy following methods in `support->helper.rb`

	```
	require 'securerandom'
	require 'time'

	def current_time
	 Time.now.to_i
	end

	def create_user_name
	 "user#{current_time}#{rand(100)}"
	end

	def create_user_email
	 "user#{current_time}#{rand(100)}@example.com"
	end

	```

2. Modify `refactor_test.rb` to use the test data by modifying the request body as follows

	```
	# set the request body
	request.body = "{\t\n\"user\": \n\t{\n\t\t\"name\": \"#{create_user_name}\", \n\t\t\"email\": \"#
	{create_user_email}\"\n\t}\n}"
	
	```
3. Run `refactor_test.rb`

	` !!! Test Should be successful !!!`
	
# STEP 4 : Define endpoints

1. To set up account configuration copy the following code into `config -> config.yml`

	```
	account:
	 url: 'subdomain url'
	 env: 'Test'	
	```

2. For simplicity define domain url and endpoints separately in `refactor_test.rb`
	
	```
	domain_url = "https://z3n-wwt.zendesk.com"
	endpoint = "/api/v2/users.json"
	
	url = URI(domain_url + endpoint)
	
	```
3. Add subdomain information in `config.yml`

	``` 
	account:
	   url: 'https://z3n-wwt.zendesk.com'
	   
	 ```
4. Define API endpoint in `endpoints.rb`

	` CREATE_USER = "/api/v2/users.json" `
	
	
5. Run `refactor_test.rb`

	` !!! Test Should be successful !!!`	
	
	
# STEP 5 : Load account information

1. To load the account information add following methods to `support->loadConfig.rb`

	```
	require 'yaml'
	
	def load_config
	 path = '../../config/config.yml'
	 YAML::load(File.open(path))
	end

	def account_url
	 yml = load_config
	 yml['account']['url']
	end
	
	def envirionment
	 yml = load_config
	 yml['account']['env']
	end 
	```

2. use above methods in `refactor_test.rb`to make sure correct values of subdomain and endpoints are loaded

	```
	puts account_url
	puts CREATE_USER

	```

3. Run `refactor_test.rb`

	`!!! test should print values present in config.yml !!!`
	
	
	
# STEP 6 : API Client configuration

1. For client configuration copy following code into `configClient.rb`

	```
	require 'net/http'

	# This method configures and returns api client 
	def configure_client
	 url = URI(account_url)
	 # api client
	 api_client = Net::HTTP.new(url.host, url.port)
	 api_client.use_ssl = true
	 api_client
	end

	```
2. modify `refactor_test.rb` to use  `configure_client `

	Replace following code

	```	
	url = URI(domain_url + endpoint)
	http = Net::HTTP.new(url.host, url.port)
	http.use_ssl = true
	```
	
	 with

	`configure_client`

2. Make changes in the as follows 

	`response = http.request(request)`  ---- change to ----> `response = api_client.request(request)


3. Run `refactor_test.rb`

	`!!! Test should be successful !!!`
	
	
# STEP 7 : Build Post request 

1. Add following method to `configClient.rb` 

	```
	def build_post_request(endpoint)
	  url = account_url + endpoint

	  # configure an API client
	  request = Net::HTTP::Post.new(URI(url))
	  request["content-type"] = 'application/x-www-form-urlencoded'
	  request["authorization"] = 'Basic bWNoYXVoYW4rd3d0QHplbmRlc2suY29tOk11Z2RoYSoxMjM='
	  request["Content-Type"] = "application/json"
	  request
	end
	```


2. In `refactor_test.rb` 

1. Replace following code

	```
	request = Net::HTTP::Post.new(url)
	request["content-type"] = 'application/x-www-form-urlencoded'
	request["content-type"] = 'application/json'
	request["authorization"] = 'Basic bWNoYXVoYW4rd3d0QHplbmRlc2suY29tOk11Z2RoYSoxMjM='
	```
	
	With

	`request = build_post_request(CREATE_USER)`
	
	
	
2. Run `refactor_test.rb`

	`!!! Test should be successful !!!`	
	
	
# STEP 8:

1. Remove print statements


# Final spec

```

code 


```



