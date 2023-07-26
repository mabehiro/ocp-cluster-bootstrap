# Create image set template
$ oc mirror init --registry registry1.cotton.blue/mirror/oc-mirror-metadata > imageset-config.yaml 

# List available catalogs
oc-mirror list operators
oc-mirror list operators --catalogs --version=4.13

# Findout required operators and channels. Channel is required to put in the imageset yaml.

oc-mirror list operators --catalog=registry.redhat.io/redhat/redhat-operator-index:v4.13


#  After update imaegset-config, upload the images
oc-mirror --config=./imageset-config.yaml docker://registry1.cotton.blue:8443/mirror
