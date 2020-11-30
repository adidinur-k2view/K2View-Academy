# Setup Fabric with Elastic Search and SSL support

Assumptions:
- An ElasticSearch instance is running with TLS support
- Password set for elastic built-in user as Elastic system authentication is required for every HTTPS call/request

Step 1 - Make sure Fabric is stopped

```~/fabric/scripts/stop.sh```

- Make sure elasticsearch instance is stopped (kill the elasticsearch PID if required)

Step 2 - ES Soft Link

- Create a soft link named ```elasticsearch``` which will point to the elasticsearch root directory:

```ln -s elasticsearch-* elasticsearch```

Step 3 - ES_HOME configuration

- Make sure the bash_profile for the elasticsearch user is defined with ES_HOME variable (point to the user’s home directory or the elasticsearch root path

NOTE: 
- In case a password is set to the elastic built-in user 
and set password require temporary elastic certificates and key generation to take place together with commands to be used for the process please hold in continue to step #4 and proceeding to Appendix – How to setup elastic built-in user with password, once done return back and continue forward


Step 4 - Copy the fabric keys tar file from previous steps to elasticsearch user/system (modify path as required):

```scp keyz.tar.gz k2view@10.10.10.10:/opt/apps/k2view/```

Step 5 - Create temporary directory and untar the keys into it:

```mkdir -p $ES_HOME/.cassandra_ssl && tar -zxvf keyz.tar.gz -C $ES_HOME/.cassandra_ssl```

Step 6 - Copy the following two keys from the extracted directory to elasticsearch configuration directory (modify path as required):

```
cp $ES_HOME/.cassandra_ssl/cassandra.keystore /home/elasticsearch/elastic/config/
cp $ES_HOME/.cassandra_ssl/cassandra.truststore /home/elasticsearch/elastic/config/
```

The directory can be removed after copy process done:

```rm -rf $ES_HOME/.cassandra_ssl```

Step 7 -	Acript download 

- [Download](https://owncloud_bkp.s3.amazonaws.com/adminoc/Utils/Hardening/secure_ES.sh) the script into $ES_HOME to set ElasticSearch instance in secure mode

```
cd $ES_HOME
chmod +x secure_ES.sh
```

Note that in order to change the password, edit the secure_ES.sh or execute with password parameter
e.g.: ./secure_ES.sh {Password}

```./secure_ES.sh Q1w2e3r4t5```

Note that the script will set two set of parameters for TLS (transport) and HTTPS, it might require other ELK component associated with Elastic Search Engine to be defined with TLS/HTTPS too (i.e. Kibana connection for ElasticSearch web GUI).

Step 8 - Once the script is executed, re-run the elasticsearch instance (which now includes TLS/SSL support)

Step 9 -	Fabric shall be configured with search engine support but running the following command under each fabric node (in case it was configured already):

```sed -i 's/#PROVIDER=ElasticSearchProvider/PROVIDER=ElasticSearchProvider/' $K2_HOME/config/config.ini```

Step 10 - Start the Fabric service:

```K2fabric start``` 

Step 11 - Fabric Studio 

In Fabric Studio, define interface for SearchEngine type:
 
Define the connection details to Elastic Search system with port 9200, credentials to authentication for HTTPS requests
The SSL properties shall be set as True, the keystore path shall point to path exists location with the .keystore file extension on Fabric server system, Password is the same one as took place in generation of the JKS keystore files (in current example: Q1w2e3r4t5), keystore type shall be set as JKS  Save the changes

Step 12 - LU Validation
- Before deploying the LU ensure the LU contains valid search fields so elastic will be part of deployment process
 

Step 13 - Run URL
When deployment is successfully done – the data shall be also visible in Elastic side by running the following url in browser (when prompted, insert the authentication details):
```https://(ES_ip_address_or_hostname):9200/(LU_name)/_search```

 

#	Appendix – How to setup elastic built-in user with password
As setup password for any built-in user in ElasticSearch system require validation of SSL certificates/keys generated by elastic mechanism.
Run the following commands: (the password which will be set in the coming example will be Q1w2e3r4t5 and should be set to any other value)

Step 1 - Generate CA and server certificates 
- Update values as required
- In case the IP address is not required but the hostname is, use the ```key --dns``` command and set the DNS name
- output file can be named too as well as the path:

```~/elasticsearch/bin/elasticsearch-certutil cert ca --pass Q1w2e3r4t5 --ip 10.21.1.109 --pem --out ~/certs.zip```

Step 2 - Create dirctory called ESCerts to extract the certs files into it and unzip certs.zip:

```mkdir ESCerts && unzip certs.zip -d ~/ESCerts```

Step 3 - Copy certs files and key to config folder in elasticsearch:

```
cp ~/ESCerts/ca/ca.crt ~/elasticsearch/config/
cp ~/ESCerts/instance/instance.crt ~/elasticsearch/config/ && \ 
cp ~/ESCerts/instance/instance.key ~/elasticsearch/config/
```

Step 4 - Create copy of the current elasticsearch.yml file:

```
cp ~/elasticsearch/config/elasticsearch.yml ~/elasticsearch/config/elasticsearch.yml.backup
```

Step 5 - Download the script to set ElasticSearch instance in secure mode 
[Secure_ElasticSearch_temporary_download_link](https://owncloud_bkp.s3.amazonaws.com/adminoc/Utils/Hardening/secure_ES_temp.sh) into $ES_HOME

```
cd $ES_HOME
chmod +x secure_ES_temp.sh
```

* in order to change the password, edit the secure_ES_temp.sh or execute with password parameter

```
e.g.: ./secure_ES_temp.sh {Password}
./secure_ES_temp.sh Q1w2e3r4t5
```

Step 6 - Set secure password (https & transport) for instance.key in elastic-keystore. When prompted to do so, insert the password that was set when temporary certificates and key was generated in step #1:

```
~/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.secure_key_passphrase
~/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.secure_key_passphrase
```

Step 7 - Run the ElasticSearch service

Step 8 - Set built-in users with password (proceed with confirmation and upon each user prompt set the required password for authentication)

```
~/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

- Upon confirmation of the passwords updates the following lines will be presented in the terminal:
```
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

Step 9 - Turnoff the elasticsearch service (kill -9 PID)

Step 10 - Cleanup the system from redundant files and setup:

```
rm -rf ~/certs.zip ~/ESCerts ~/secure_ES_temp.sh ~/elasticsearch/config/ca.crt 
~/elasticsearch/config/instance.crt ~/elasticsearch/config/instance.key ~/elasticsearch/config/elasticsearch.yml
mv ~/elasticsearch/config/elasticsearch.yml.backup ~/elasticsearch/config/elasticsearch.yml
```

[![Previous](/articles/images/Previous.png)](/articles/99_fabric_infras/devops/06_fabric_kafkaSSL_support.md)