## It's time to dive into ci/cd ppipelines.
```yaml
image: alpine:latest

# Login to GitLab Docker registry
.gitlab_login: &gitlab_login |
  docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY

stages:
  - build
  - api



build:
  image: alpine:latest
  stage: build
  script:
    - pwd
    - ls
  only:
    - main

deploy:
  variables:
    SSH: 'root@x.x.x.x -p 2222'   
  stage: api
  script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SSH_PRIVATE_KEY")
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    - |
      ssh $SSH "
        ls
        pwd
  only:
    - main
  






 
```
