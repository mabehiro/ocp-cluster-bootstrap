# ocp-cluster-bootstrap
### Dry run, -K asks password for sudo
ansible-playbook -i inventory baremetal_setup_playbook.yaml --check -K

### Jinja test
ansible all -i "localhost," -c local -m template -a "src=./templates/vm_master.xml.j2 dest=./test.txt" --extra-vars='{"masters": ["master1", "master2", "master3"]}'


### ZTP-pipeline setup
oc patch argocd openshift-gitops -n openshift-gitops --type=merge --patch-file ${ZTP-PIPELINE}/deployment/argocd-openshift-gitops-patch.json