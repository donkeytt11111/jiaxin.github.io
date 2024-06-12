```yaml
version: '3.6'
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: '172.16.8.3'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://172.16.8.3:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2424
    ports:
      - '8929:8929'
      - '4443:443'
      - '2424:22'
    volumes:
      - '/srv/config:/etc/gitlab'
      - '/srv/logs:/var/log/gitlab'
      - '/srv/data:/var/opt/gitlab'
    shm_size: '256m'