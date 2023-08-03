# Fakefish

In order to use HPE DL160 G9, I need to use fakefish which will proxy Redfish.
Please refer to the below.

https://github.com/openshift-metal3/fakefish


https://github.com/openshift-metal3/fakefish/blob/main/user-docs/running-fakefish-on-ocp.md

For my lab, 

```
FAKEFISH_IMAGE=quay.io/lhalleb/fakefish:hpe-dl360-gen9
for bmc in 192.168.1.62 
do
    BMC_NAME=$(echo ${bmc} | tr "." "-")
    cat <<EOF | oc apply -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: fakefish-${BMC_NAME}
      name: fakefish-${BMC_NAME}
      namespace: fakefish
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: fakefish-${BMC_NAME}
      strategy: {}
      template:
        metadata:
          labels:
            app: fakefish-${BMC_NAME}
        spec:
          containers:
          - image: ${FAKEFISH_IMAGE}
            name: fakefish
            resources: {}
            args:
            - "--remote-bmc"
            - "${bmc}"
            - "--tls-mode"
            - "disabled"
EOF
cat <<EOF | oc apply -f -
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: fakefish-${BMC_NAME}
      name: fakefish-${BMC_NAME}
      namespace: fakefish
    spec:
      ports:
      - name: http
        port: 9000
        protocol: TCP
        targetPort: 9000
      selector:
        app: fakefish-${BMC_NAME}
      type: ClusterIP
EOF
cat <<EOF | oc apply -f -
    apiVersion: route.openshift.io/v1
    kind: Route
    metadata:
      name: fakefish-${BMC_NAME}
      namespace: fakefish
    spec:
      port:
        targetPort: http
      tls:
        termination: edge
      to:
        kind: "Service"
        name: fakefish-${BMC_NAME}
        weight: null
EOF
done
```