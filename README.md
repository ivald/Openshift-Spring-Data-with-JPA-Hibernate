<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/OpenShift-LogoType.svg/1200px-OpenShift-LogoType.svg.png" width="80">

**Openshift Tutorial link:**

`https://learn.openshift.com/middleware/courses/middleware-spring-boot/spring-db-access`

**Run virtual box app**

`.\minishift.exe start --vm-driver virtualbox`

**How to Set up an environment property on linux machine**

**oc (openshift client):** `.\minishift oc-env`

**`$Env:PATH = "C:\Users\ilya.valdman\.minishift\cache\oc\v3.11.0\windows;$Env:PATH"`**

**Login in Openshift on local machine**

`oc login https://192.168.99.100:8443 --insecure-skip-tls-verify=true -u ilyav -p admin`

**Create New Project**

`oc new-project dev --display-name="Dev - Spring Boot App"`

**Inside New Project we can add new apps, such as:**

    1. oc new-app -e POSTGRESQL_USER=dev \
          -e POSTGRESQL_PASSWORD=secret -e POSTGRESQL_DATABASE=my_data \
          openshift/postgresql-92-centos7 --name=my-database
     
    2. oc new-app --docker-image jenkins/jenkins:lts --name=my-jenkins
       oc expose svc/my-jenkins
       oc get pods
       oc exec my-jenkins-1-7m5ft cat /var/jenkins_home/secrets/initialAdminPassword

**How Build and Deploy existing project on Openshift**

`mvn package fabric8:deploy -Popenshift -DskipTests`

**Info command**

`oc get all`

**Examining the Pod**

`oc get pods`\
`oc get pod parksmap-1-6xgcn -o yaml`

**Services**

`oc get services`\
`oc get service parksmap -o yaml`\
`oc describe service parksmap`\
`oc describe svc parksmap`\
`oc get svc parksmap -o yaml`

**Exploring Deployment-related Objects**

`oc get dc`\
`oc get rc`

**Scaling the Application**

`oc scale --replicas=2 dc/parksmap`\
`oc get endpoints parksmap`\
`oc delete pod parksmap-1-hx0kv && oc get pods`

**Creating a Route**

`route "parksmap" exposed`\
`oc get route`

**Logs**

`oc get pods`\
`oc logs parksmap-1-4mkdk`

**Grant User View Permissions**

`oc policy add-role-to-user view userXY`

**Remote Shell Session to a Container Using the CLI**

`oc get pods`\
`oc rsh parksmap-2-4mkdk`\
 sh-4.2$ ls /

**Execute a Command in a Container**

`oc exec parksmap-1-4mkdk -- ls -l /parksmap.jar`\
-rw-r--r--. 1 root root 21753930 Feb 20  2017 /parksmap.jar

**Builds**

`oc get builds`\
`oc logs -f builds/nationalparks-1`

**Import image from redhat repository**

1. `oc login https://192.168.99.100:8443 --insecure-skip-tls-verify=true -u ilyav -p admin`
2. `oc import-image openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm`

**Import templates from another account**

1. `oc login https://master.karyyz-d0ea.openshiftworkshop.com --insecure-skip-tls-verify=true -u user7 -p openshift`
2. `oc get templates -n openshift`\
    -n is a namespace in kubernetes or project in openshift
3. `oc get template mongodb-ephemeral -n openshift -o yaml`
4. `oc get template mongodb-ephemeral -n openshift -o yaml > mongodb-ephemeral.yml`
5. `ls`
6. `oc login https://192.168.99.100:8443 --insecure-skip-tls-verify=true -u ilyav -p admin`
7. `oc create -f mongodb-ephemeral.yml -n openshift`
8. `oc login https://master.karyyz-d0ea.openshiftworkshop.com --insecure-skip-tls-verify=true -u user7 -p openshift`
9. `oc get templates -n openshift`
10. `oc get template openjdk18-web-basic-s2i -n openshift -o yaml > openjdk.yml`
11. `cat openjdk.yml`
12. `oc edit template openjdk18-web-basic-s2i -n openshift`

**Logout from Openshift**

`oc logout`
