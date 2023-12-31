# OC mirror setup
## Create image set template
~~~
$ oc mirror init --registry registry1.cotton.blue/mirror/oc-mirror-metadata > imageset-config.yaml 
~~~

## List available catalogs
~~~
oc-mirror list operators
oc-mirror list operators --catalogs --version=4.13
~~~

Findout required operators and channels. Channel is required to put in the imageset yaml.
~~~
oc-mirror list operators --catalog=registry.redhat.io/redhat/redhat-operator-index:v4.13
~~~

## After update imaegset-config, upload the images
~~~
oc-mirror --config=./imageset-config.yaml docker://registry1.cotton.blue:8443/mirror --continue-on-error
~~~

## Update imagecontentsourcepolicies.operator.openshift.io to add new image
You will need to set ICMP to add new image source into the cluster later.
~~~
(base) [mabe@mabe results-1690507677]$ pwd
/home/mabe/Projects/HomeLab/WIP/baremetalsetup/mirror/oc-mirror-workspace/results-1690507677
(base) [mabe@mabe results-1690507677]$ oc apply -f imageContentSourcePolicy.yaml
~~~