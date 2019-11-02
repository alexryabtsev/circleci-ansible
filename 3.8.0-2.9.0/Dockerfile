FROM circleci/python:3.8

ENV ANSIBLE_HOST_KEY_CHECKING=False

RUN sudo apt-get update && \
	sudo apt-get -y install rsync && \
	sudo apt-get clean && \
	sudo rm -rf /var/lib/apt/lists/* && \
	sudo pip install --no-cache Ansible==2.9.0

WORKDIR /home/circleci
