# mySecureMqttBroker

### Setup MQTT Broker:
Installation:

    sudo apt-get install mosquitto mosquitto-clients
    
    mosquitto_sub -h localhost -t test
    mosquitto_pub -h localhost -t test -m "hello world"
	
Configuring MQTT Passwords
		
    sudo mosquitto_passwd -c /etc/mosquitto/passwd sammy	
			
    sudo nano /etc/mosquitto/conf.d/default.conf
			
      allow_anonymous false
      password_file /etc/mosquitto/passwd

Check connection

	telnet ip port
	
restart Mosquitto
	
    sudo systemctl restart mosquitto

    mosquitto_sub -h localhost -t test -u "sammy" -P "password"
    mosquitto_pub -h localhost -t "test" -m "hello world" -u "sammy" -P "password"
    mosquitto_pub -h ip -p 1883 -t "test" -m "hello" -u "user" -P "pwd"

Remove the mosquitto

	sudo apt-get remove  mosquitto mosquitto-clients
		
Completely removing mosquitto with all configuration files

	sudo apt-get purge mosquitto mosquitto-clients

### Manually generate certs:

1. Create CA key pair, CA certificate

        openssl genrsa -des3 -out ca.key 2048
        openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
      
    or

        openssl req -newkey rsa:2048 -x509 -nodes -sha512 -days 3650 -extensions v3_ca -keyout ca.key -out ca.crt -subj "/CN=A MQTT broker/O=OwnTracks.org/OU=generate-CA/emailAddress=nobody@example.net"
	
2. Create Broker key pair, CA certificate

        openssl genrsa -out server.key 2048
        openssl req -new -out server.csr -key server.key
        openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 3650   

    or
    
        openssl genrsa -out server.key 2048
        openssl req -new -sha512 -out server.csr -key server.key -subj "/CN=server/O=OwnTracks.org/OU=generate-CA/emailAddress=nobody@example.net"
        openssl x509 -req -sha512 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -CAserial ca.srl -out server.crt -days 3650 -extensions JPMextensions -extfile /tmp/cacnf.nHbN4nAJ

### Automatic generate certs:

Run auto script
	
			bash ./generate-CA.sh

### Config:

1. This has created the following files:

			ca.crt  : CA Certificate
			ca.key  : CA key pair (private, public)
			ca.srl  : CA serial number file
			server.crt : server certificate
			server.csr : certificate sign request, not needed any more
			server.key : server key pair
	
2. Inside the Mosquitto installation, create a folder (e.g. ‘certs’ if it does not already exist) and copy the following files we have created in the previous steps:
The ca.crt belongs to the client (needs to be copied there).
	
			/etc/mosquitto/certs/ca.crt
			/etc/mosquitto/certs/server.crt
			/etc/mosquitto/certs/server.key
	
	
3. Open and edit <mosquitto>/mosquitto.conf

			# Port to use for the default listener.
			port 8883
			
			#capath
			cafile /etc/mosquitto/certs/ca.crt

			# Path to the PEM encoded server certificate.
			certfile /etc/mosquitto/certs/server.crt

			# Path to the PEM encoded keyfile.
			keyfile /etc/mosquitto/certs/server.key
			
			tls_version tlsv1
			#require_certificate, which may be set to true or false. If false, the SSL/TLS component of the client will 	verify the server but there is no requirement for the client to provide anything for the server
			require_certificate false
	
4. Restart Mosquitto service

			sudo service mosquitto restart
5. Test connection:

		
			mosquitto_sub -h ip -p 8883 -t "test" -u "user" -P "pwd" --cafile ca.crt
			mosquitto_pub -h ip -p 8883 -t "test" -m "hello world" -u "user" -P "pwd" --cafile ca.crt --tls-version tlsv1.2
	
5. Firewall allow 8883
		
			sudo ufw allow 8883

### References:

script to gen Cert-CA

      https://github.com/owntracks/tools/tree/master/TLS

MQTT with TLS/SSL

      https://mcuoneclipse.com/2017/04/14/enable-secure-communication-with-tls-and-the-mosquitto-broker/
	  
	  http://rockingdlabs.dunmire.org/exercises-experiments/ssl-client-certs-to-secure-mqtt

MQTT on Digital Ocean

		https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-the-mosquitto-mqtt-messaging-broker-on-ubuntu-16-04
