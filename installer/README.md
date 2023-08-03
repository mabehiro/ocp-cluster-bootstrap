# Agent-Based-Installer setup

Here is the officical product [documentation](https://docs.openshift.com/container-platform/4.13/installing/installing_with_agent_based_installer/installing-with-agent-based-installer.html#installing-ocp-agent-retrieve_installing-with-agent-based-installer).

## install installer
~~~
sudo mv openshift-install /usr/local/bin/
~~~

## Install nmstate dependency 

~~~
sudo dnf install /usr/bin/nmstatectl -y
~~~

## make directory
install-config.yaml and agent-config.yaml need to be placed under working directory. After building the image, those yaml will be deleted and replaced by artifacts. Take backups.

~~~
mkdir manifests
~~~

## Create install-config.yaml

~~~
cat << EOF > ./manifests/install-config.yaml
apiVersion: v1
baseDomain: cotton.blue
compute:
- architecture: amd64 
  hyperthreading: Enabled
  name: worker
  replicas: 3
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: hub
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  - cidr: fd02::/48
    hostPrefix: 64
  machineNetwork:
  - cidr: 192.168.1.0/24
  - cidr: fd00::64
  networkType: OVNKubernetes 
  serviceNetwork:
  - 172.30.0.0/16
  - fd03::/112
platform:
  baremetal:
    apiVIPs:
    - 192.168.1.125
    - fd00::192.168.1.125
    ingressVIPs:
    - 192.168.1.126
    - fd00::192.168.1.126
pullSecret: '{"auths":{"registry1.cotton.blue":{"auth":"aW5pdDpjaGFuZ2VtZQ==","email":"mabe@redhat.com"}}}'
sshKey: |
  'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD2Sh84/z9jj4Neg4xI3jlPO6C8sLdz3ykJjo+vD+WveOSBql2LiJ1k4z/AcBC55/ZPfZt6oK1pqtHYTIcZLobH7ApHvRSdsDmVgh67u/Lx5XMZ5ct6O9jC+lp98vNjGP5c9Zfcn1zIzcPATleOS52ubGQJA58ryuckoEq6TyyfvcHjUMNBv7AULDQ68syv0i9tuS+SH9cUgaEQGHsTyL9wA5fudh2JpLMvZigE0KqAwP/Z91QLyMFZsuxoy7x0CFBFym4miXiF7gVSpYsJxcUbb6qRSeHi4PGhNBm3IytRcbTOrwcdCx2E2uiyuWliC85NtMslQHYy+8eXcUavSW3GZnxgjenUnaNAIhQ1GQdUAdRmsLCzRehzBi/i3mc4ZgkhODQHacCg14RKj7c5GgyAh3b/6M9D43sqrjZa2vH1+RwID9wgWngVjJmy/RZ3tIhT94GKUn4/AWfAUpAtM1c3C1sxqRFTk6Q9otYm1Qq95/V0ewBQnbuzkpJSlCjtNqM= mabe@mabe.remote.csb' 
additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  MIIDgzCCAmugAwIBAgIUD8nDejM6bCeMjKKhfwG+ix+tD/AwDQYJKoZIhvcNAQEL
  BQAwbTELMAkGA1UEBhMCVVMxCzAJBgNVBAgMAlRYMQ8wDQYDVQQHDAZEQUxMQVMx
  DDAKBgNVBAoMA09DUDEQMA4GA1UECwwHSE9NRUxBQjEgMB4GA1UEAwwXQ2VydGlm
  aWNhdGlvbiBBdXRob3JpdHkwHhcNMjMwNzEwMjEyNDAwWhcNMjQwNzA5MjEyNDAw
  WjBhMQswCQYDVQQGEwJVUzELMAkGA1UECAwCVFgxDzANBgNVBAcMBkRBTExBUzEM
  MAoGA1UECgwDT0NQMRAwDgYDVQQLDAdIT01FTEFCMRQwEgYDVQQDDAtjb3R0b24u
  Ymx1ZTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKA30o3sGfmhWiv9
  7IhVN3R5mYsoC+qpmcWx+uNk3XMNEH+02ywSYLlxiJ6rquAmHyTRQtR9mUz2DBN7
  JUKCd1rA+MZOSrerqpKN/XgtW75dDttFm25mgo3hVsBNmIabeE45w9HcEhFgacjp
  djnDJE7RDQkIHHpR3JKUDCFy16507XNVLmDd9gjH3WRej0pbHCOXMzTZCUoKo2iK
  h7U1Fvj/4m1QGmALzWSkPO3WArPjgFVEaAMymysNORsXFGLtfuIseU34+tXmwH8j
  h7pW0C808Tts7mVi4f6H+QLaykiFMlRIXPt6N6cwovc9zBt+wj25Zy/yFn3u7Vz5
  StiOM6ECAwEAAaMnMCUwCQYDVR0TBAIwADAYBgNVHREEETAPgg0qLmNvdHRvbi5i
  bHVlMA0GCSqGSIb3DQEBCwUAA4IBAQAwwWx3zuiK3pNVmdbH9BFusW1dpve1VECh
  bkWa8LuQoHlZeBCPa/gS7wFeZC2rYKPwGWF5e1AgD8Iif+DLH1P/MunX0Bxc7vUc
  9YaczER+xM3w7aYmLFQUW428Ip3TzlBRJdcl7X7+OcJJ+CDFvqmT62O7yKz+hZuc
  ZjSgdP35HtoVcaNhlvUuhjD6DEIRO1v8qWu1UhmnliHYZcLFPKKW9Wyys9nwuLFO
  nQ5JxMVxtMS/LM2izxlMIWV5cU1Fh6bG71X/3yvKdoNbAYcnSeinkQ8U6jT9+qG0
  KruOlTNY6UnkvBF41vlWcAryLbH2I8757aCxrr4rJY/r1G1cJUhN
  -----END CERTIFICATE-----
imageContentSources:
  - mirrors:
    - registry1.cotton.blue:8443/mirror/ubi8
    source: registry.access.redhat.com/ubi8
  - mirrors:
    - registry1.cotton.blue:8443/mirror/ubi8
    source: registry.redhat.io/ubi8
  - mirrors:
    - registry1.cotton.blue:8443/mirror/container-native-virtualization
    source: registry.redhat.io/container-native-virtualization
  - mirrors:
    - registry1.cotton.blue:8443/mirror/openshift-update-service
    source: registry.redhat.io/openshift-update-service
  - mirrors:
    - registry1.cotton.blue:8443/mirror/openshift4
    source: registry.redhat.io/openshift4
  - mirrors:
    - registry1.cotton.blue:8443/mirror/nvidia
    source: nvcr.io/nvidia
  - mirrors:
    - registry1.cotton.blue:8443/mirror/nvidia
    source: registry.connect.redhat.com/nvidia
  - mirrors:
    - registry1.cotton.blue:8443/mirror/odf4
    source: registry.redhat.io/odf4
  - mirrors:
    - registry1.cotton.blue:8443/mirror/oadp
    source: registry.redhat.io/oadp
  - mirrors:
    - registry1.cotton.blue:8443/mirror/lvms4
    source: registry.redhat.io/lvms4
  - mirrors:
    - registry1.cotton.blue:8443/mirror/rhceph
    source: registry.redhat.io/rhceph
  - mirrors:
    - registry1.cotton.blue:8443/mirror/multicluster-engine
    source: registry.redhat.io/multicluster-engine
  - mirrors:
    - registry1.cotton.blue:8443/mirror/rhel8
    source: registry.redhat.io/rhel8
  - mirrors:
    - registry1.cotton.blue:8443/mirror/rhmtc
    source: registry.redhat.io/rhmtc
  - mirrors:
    - registry1.cotton.blue:8443/mirror/rhacm2
    source: registry.redhat.io/rhacm2
  - mirrors:
    - registry1.cotton.blue:8443/mirror/openshift-gitops-1
    source: registry.redhat.io/openshift-gitops-1
  - mirrors:
    - registry1.cotton.blue:8443/mirror/cert-manager
    source: registry.redhat.io/cert-manager
  - mirrors:
    - registry1.cotton.blue:8443/mirror/rh-sso-7
    source: registry.redhat.io/rh-sso-7
  - mirrors:
    - registry1.cotton.blue:8443/mirror/openshift/release
    source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
  - mirrors:
    - registry1.cotton.blue:8443/mirror/openshift/release-images
    source: quay.io/openshift-release-dev/ocp-release

EOF
~~~

## Generate Agent-config.yaml
~~~
cat > agent-config.yaml << EOF
apiVersion: v1alpha1
kind: AgentConfig
metadata:
  name: hub
rendezvousIP: 192.168.1.113
hosts:
  - hostname: master1
    role: master
    rootDeviceHints:
      deviceName: /dev/vda
    interfaces:
      - name: eno1
        macAddress: 52:54:00:2a:45:c6
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          mac-address: 52:54:00:2a:45:c6
          ipv4:
            enabled: true
            dhcp: true
          ipv6:
            enabled: true
            dhcp: true
  - hostname: master2
    role: master
    rootDeviceHints:
      deviceName: /dev/vda
    interfaces:
      - name: eno1
        macAddress: 52:54:00:c3:4b:67
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          mac-address: 52:54:00:c3:4b:67
          ipv4:
            enabled: true
            dhcp: true
          ipv6:
            enabled: true
            dhcp: true
  - hostname: master3
    role: master
    rootDeviceHints:
      deviceName: /dev/vda
    interfaces:
      - name: eno1
        macAddress: 52:54:00:7c:76:94
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          mac-address: 52:54:00:7c:76:94
          ipv4:
            enabled: true
            dhcp: true
          ipv6:
            enabled: true
            dhcp: true
  - hostname: worker1
    role: worker
    rootDeviceHints:
      deviceName: /dev/vda
    interfaces:
      - name: eno1
        macAddress: 52:54:00:f4:f4:d4
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          mac-address: 52:54:00:f4:f4:d4
          ipv4:
            enabled: true
            dhcp: true
          ipv6:
            enabled: true
            dhcp: true
  - hostname: worker2
    role: worker
    rootDeviceHints:
      deviceName: /dev/vda
    interfaces:
      - name: eno1
        macAddress: 52:54:00:5e:b6:e5
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          mac-address: 52:54:00:5e:b6:e5
          ipv4:
            enabled: true
            dhcp: true
          ipv6:
            enabled: true
            dhcp: true
  - hostname: worker3
    role: worker
    rootDeviceHints:
      deviceName: /dev/vda
    interfaces:
      - name: eno1
        macAddress: 52:54:00:34:a6:3d
    networkConfig:
      interfaces:
        - name: eno1
          type: ethernet
          state: up
          mac-address: 52:54:00:34:a6:3d
          ipv4:
            enabled: true
            dhcp: true
          ipv6:
            enabled: true
            dhcp: true
EOF
~~~

## create the agent image

~~
cp install-config.yaml ./manifests
cp agent-config.yaml ./manifests
openshift-install --dir manifests agent create image
~~

## boot VMs and the watch
~~~
openshift-install --dir manifests agent wait-for bootstrap-complete --log-level=info
openshift-install --dir manifests agent wait-for install-complete
~~~
