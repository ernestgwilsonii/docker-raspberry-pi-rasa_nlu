#######################################
# RASA NLU on Docker for Raspberry Pi #
#              REF: https://rasa.com/ # 
#######################################


###############################################################################
# Docker build
time docker build --no-cache -t ernestgwilsonii/docker-raspberry-pi-rasa_nlu:15 -f Dockerfile.armhf .
time docker build -t ernestgwilsonii/docker-raspberry-pi-rasa_nlu:15 -f Dockerfile.armhf .
docker images

# Verify 
docker run -it -p 5000:5000 ernestgwilsonii/docker-raspberry-pi-rasa_nlu:15
# From another ssh session:
#docker ps

# Upload to Docker Hub
docker login
docker push ernestgwilsonii/docker-raspberry-pi-rasa_nlu:15
###############################################################################


###############################################################################
# First time setup #
####################
# Create bind mounted directies
sudo mkdir -p /opt/rasa
sudo chmod -R a+rw /opt/rasa

##########
# Deploy #
##########
# Deploy the stack into a Docker Swarm
docker stack deploy -c docker-compose.yml rasa
# docker stack rm rasa

# Verify
docker service ls | grep rasa
docker service logs -f rasa_rasa


#######################
# RASA NLU Basic Test #
#######################
# REF: https://github.com/RasaHQ/rasa
# REF: https://pypi.org/project/rasa-nlu/

# Verify RASA NLU is running
curl 'http://localhost:5000/status'

# Determine RASA NLU version
curl 'http://localhost:5000/version'


# Train the "default" model
curl --request POST --header 'content-type: application/x-yml' --data-binary @config_train_server_md.yml --url 'localhost:5000/train?project=default'

# Query the default model
curl 'http://localhost:5000/parse?q=amazing'


# Train a model named: test_model
curl --request POST --header 'content-type: application/x-yml' --data-binary @config_train_server_md.yml --url 'localhost:5000/train?project=test_model'

# Query a model named: test_model
curl 'http://localhost:5000/parse?q=sad&project=test_model'

###############################################################################

