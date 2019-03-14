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

`oc get all"`\
`oc describe svc parksmap"`

**Logout from Openshift**

`oc logout`
