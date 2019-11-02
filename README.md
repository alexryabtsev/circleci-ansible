# circleci-ansible

Dockerfile to run [Ansible](https://github.com/ansible/ansible) in [CircleCI](https://circleci.com/).

## How to use

1. Place the next `Dockerfile` to your `provisioning` directory:

```dockerfile
FROM doomatel/circleci-ansible:3.8.0-2.9.0
ADD --chown=circleci:circleci . /home/circleci
RUN sudo chmod 700 /home/circleci/.ssh && sudo chmod 600 /home/circleci/.ssh/id_rsa*
```

2. Add job `deploy` to your `circle.yml` with the following config:

```yaml
version: 2
jobs:
  deploy:
    docker:
      - image: docker:19.03.4-git
    steps:
      - checkout
      - setup_remote_docker
      - add_ssh_keys:
          name: DEPLOY - Add SSH keys
          fingerprints:
            - 11:11:11
            - 22:22:22
      - run:
          name: DEPLOY - make directory for SSH keys
          command: mkdir /root/project/provisioning/.ssh
      - run:
          name: DEPLOY - Copy SSH keys
          command: |
            cp /root/.ssh/id_rsa_111111 /root/project/provisioning/.ssh/id_rsa_develop
            cp /root/.ssh/id_rsa_222222 /root/project/provisioning/.ssh/id_rsa_beta
      - deploy:
          name: DEPLOY - Run deployment
          command: |
            docker build /root/project/provisioning -t ansible-deploy
            docker run --rm ansible-deploy \
              ansible-playbook /home/circleci/deploy.yml \
              -i /home/circleci/environments/hosts.ini \
              -e env=${CIRCLE_BRANCH} \
              --key-file=/home/circleci/id_rsa_${CIRCLE_BRANCH}
```

* Change `11:11:11` + `111111` and `22:22:22` + `222222`to your actual SSH key fingerprints.
* Change `id_rsa_develop` & `id_rsa_beta` to your environments. They should be equal to github branches passed in `${CIRCLE_BRANCH}`.

## How it works

1. Checkout code.

```yaml
- checkout
```

2. Setup connection to Docker daemon.

```yaml
- setup_remote_docker
```

3. Load SSH keys from CI.

```yaml
- add_ssh_keys:
    name: DEPLOY - Add SSH keys
    fingerprints:
      - 11:11:11
      - 22:22:22
```

4. Copy SSH keys to directory with Ansible playbook (`provisioning`) to be able to put it into the context of temporary Docker image.

```yaml
- run:
    name: DEPLOY - make directory for SSH keys
    command: mkdir /root/project/provisioning/.ssh
- run:
    name: DEPLOY - Copy SSH keys
    command: |
      cp /root/.ssh/id_rsa_111111 /root/project/provisioning/.ssh/id_rsa_develop
      cp /root/.ssh/id_rsa_222222 /root/project/provisioning/.ssh/id_rsa_beta
```

5. Build temporary Docker image `ansible-deploy` from `doomatel/circleci-ansible` image, including SSH keys.


```bash
docker build /root/project/provisioning -t ansible-deploy
```

6. Run Ansible playbook in one time container based on temporary `ansible-deploy` Docker image.

```bash
docker run --rm ansible-deploy \
  ansible-playbook /home/circleci/deploy.yml \
  -i /home/circleci/environments/hosts.ini \
  -e env=${CIRCLE_BRANCH} \
  --key-file=/home/circleci/id_rsa_${CIRCLE_BRANCH}
```
