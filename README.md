# MySQL operator
MySQL operator creates/configures/manages MySQL databases in containers, in AWS, GKE and Azure.

## How does it work?

First you run the `mysql-operator` which consists in 2 containers:

* KWatch: that watches the API Server for changes in the MySQL `thirdpartyobject`
* Klunch: that interfaces with the cloud provider specified in the 3rd party object.

First, you create a MySQL 3rd party object (`mysql-3rdpart.yaml):

        apiVersion: extensions/v1beta1
        kind: ThirdPartyResource
        description:  "A specification of a MySQL Database."
        metadata:
          name: "mysql.infra.k8s.uk"
          labels:
              resource: database
              object: mysql
        versions:
          - name: v1

Once we have the third party object created we can use the API to create objects of that type. If we list the available APIs we can see that there's a new entry


        curl -k -l -E ~/.minikube/certificate.pfx:test https://192.168.99.100:8443
        {
          "paths": [
            "/api",
            "/api/v1",
            "/apis",
            "/apis/apps",
            "/apis/apps/v1alpha1",
            "/apis/authentication.k8s.io",
            "/apis/authentication.k8s.io/v1beta1",
            "/apis/authorization.k8s.io",
            "/apis/authorization.k8s.io/v1beta1",
            "/apis/autoscaling",
            "/apis/autoscaling/v1",
            "/apis/batch",
            "/apis/batch/v1",
            "/apis/batch/v2alpha1",
            "/apis/certificates.k8s.io",
            "/apis/certificates.k8s.io/v1alpha1",
            "/apis/extensions",
            "/apis/extensions/v1beta1",
            "/apis/infra.k8s.uk",
            "/apis/infra.k8s.uk/v1",
            "/apis/infra.sohohouse.com",
            "/apis/infra.sohohouse.com/v1",
            "/apis/policy",
            "/apis/policy/v1alpha1",
            "/apis/rbac.authorization.k8s.io",
            "/apis/rbac.authorization.k8s.io/v1alpha1",
            "/apis/storage.k8s.io",
            "/apis/storage.k8s.io/v1beta1",
            "/healthz",
            "/healthz/ping",
            "/logs",
            "/metrics",
            "/swaggerapi/",
            "/ui/",
            "/version"
          ]
        }

we can see that we have 2 new APIs:

            /apis/infra.k8s.uk
            /apis/infra.k8s.uk/v1

Now, if we create a `Mysql` object we will be able to query the API. There are a few examples in the `kubernetes` folder. This is the simpler case:

        apiVersion: "infra.k8s.uk/v1"
        kind: "Mysql"
        description: "Object to define a database"
        metadata:
          name: "wordpress"
        spec:
          name: "wordpress"
          provider: "kubernetes"
          username: "wpsecretadmin"
          password: "S3cr3t_P4ssw0rd"
          service: "mywordpress"

Note that the `apiVersion` is now `infra.k8s.uk/v1` that matches our 3rd party object `name` and `version` attribute.

### What happens once I create the 3rd party object?

Nothing, once you create the 3rd party object what you're doing behind the scenes is to define the new API endpoints. Nothing happens until you create a `Mysql` object.

### How do I create a `Mysql` object?

The same way that you create all the other kubernetes resources, using the API and sending the manifest with your desired state.

        kubectl create -f kubernetes/wordpress-db.yaml

This will create a `mysql` object in the cluster:

        -> % kubectl get mysql
        NAME           LABELS    DATA
        wordpres-gke   <none>    {"apiVersion":"infra.k8s.uk/v1","description":"Obj...
        wordpres-rds   <none>    {"apiVersion":"infra.k8s.uk/v1","description":"Obj...
        wordpress      <none>    {"apiVersion":"infra.k8s.uk/v1","description":"Obj...

That's it, from the user point of view, the only thing you have to do is to define a yaml file with a minimal configuration and submit it to the cluster, once the object is created, the heavy lifting happens behind the scenes and it's completely transparent to the user.

### Great, but how is my database created?

The Operator has 2 processes: one that watches the api for changes and one that submits kubernetes jobs to the api server.

We split functionality in 2 services because it's more extesible this way. From the conceptual point of view there are two functions also: watching and sending work loads.
