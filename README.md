

## Quick Start

Builds ansible in docker image
```
docker -f Dockerfile.ansible -t ansible:6.4.0 .
```

Playbooks are in <cwd>
```
docker run -v <cwd>:/workspace -it ansible:6.4.0 bash
cd /workspace
ansible-playbook -k -i inventory playbook.yaml 
```