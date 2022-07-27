# Operator Demo v5.1.1
Preference is to perform all demos using OpenShift unless the client is using a specific environment that you can mirror for the demo (i.e. Rancher)

# Environment Setup:

## [-] Check Loadbalancer for ArgoCD, AcctDev, Keycloak, and Grafana:
```
kubectl get services -n finance
kubectl get services -n keycloak
kubectl get services -n pgmonitor
kubectl get services -n argocd

sudo vi /etc/hosts
```

## Install ArgoCD
```
kc create namespace argocd
kc apply -n argocd -f util/argocd/install.yaml
kc patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kc get secret -n argocd argocd-initial-admin-secret  --template={{.data.password}} | base64 -d
```

If not running on OpenShift or environment that does not have a load balancer:
```
kc patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

## [-] Open in browser:
- OpenShift Console
- ArgoCD Console
- Keycloak
- Monitoring (Grafana)

## [-] Setup Terminal:
```
kc login...

cd pgodemo/openshift
cd rancher/postgres-operator-examples
```

## [-] Clean Up Environment:
- Remove applications from Argo
- Remove all postgres clusters
- Ensure argo is connected to github
- Uninstall pgMonitor

## [-] Delete Operator

1. Make sure there are no postgresclusters still active
```
kubectl get postgresclusters --all-namespaces
```

2. Delete operator
```
kubectl delete -k install/default
```

## [-] Create Namespaces
```
kubectl create namespace pgmonitor
kubectl create namespace finance
kubectl create namespace finance-uat
kubectl create namespace finance-prod
kubectl create namespace keycloak
```

## [-] Update Manifest for OpenShift (if applicable)

- For each postgres manifest, ensure that openshift key is set to
  `true` if running on OpenShift, otherwise set to `false`
- For monitoring, ensure that `fsGroup` is commented out in all `deploy*`
  yaml files if on OpenShift
- Change kustomization manifest for monitoring to use pgmonitor namespace

## [-] Setup Jmeter
- Start Jmeter
- Open Accounting.jmx test
- Clear all (broom icons) previous runs

## [-] Prepare terminal windows:
- 2 terminal windows
- 1 split terminal window (1 with watch command ready)

# Demo

## Slide 1: Title Slide

## Slide 2: About Me
> Answer the question: Why should I listen to you?

My name is Roberto Mello and I’m a Solutions Architect for Crunchy Data.
Over the last 20 years I’ve worked in several enterprise database,
development and management roles. At Crunchy I help customers deploy
Crunchy Data Postgres solutions into their infrastructure and application
stack, including the Postgres Operator for Kubernetes, helping them have
the best experience with Postgres so they can focus on their business needs.

## Slide 3: Crunchy Data
> Having the right partner ensures a higher degree of success...
> Establish credibility of Crunchy Data with Postgres

In the on-premises bare-metal world, DBAs had to worry about setting up,
configuring and baby-sitting everything from the servers, to disks, SANs, 
operating system parameters, etc

The container world brings opportunities and flexibility, but running stateful
applications in containers also requires specific skills. 

Regardless of where you in that journey, the key to success is having a
great partner who understands both Kubernetes and the database inside and out.

### Postgres Background

**Our business is Postgres!**

- We contribute about a significant portion of the ongoing Postgres code
- Several Crunchy Data team members are members of the Postgres Global
  development Group Core Team and/or are Major Contributors
- Many advanced components like pgBackRest, pgMonitor, JDBC Driver, etc.
  that are key components in the overall Postgres ecosystem are authored
  and maintained by Crunchy Data
- For a lot of our team, the experience with Postgres goes back very close
  to its original release to the open source community in 1996

## Slide 4: Crunchy Postgres for Kubernetes
> Establish credibility with Operators and Kubernetes

### Containers Background
- In 2014, when containers were then called Cartridges, Crunchy 
  started asking the question: "Can we and even should we run Postgres
  inside containers?"
- If so, what would be the best practices for doing so?
- Around this same time Google released Kubernetes as an open source
  orchestrator for containers
- Crunchy recognized the value of Kubernetes and invested heavily
  in developing a Kubernetes operator to automate various life cycles
  and apply its expertise in best practices around Postgres in Kubernetes

### Operator Background
- In 2016, the development of the Operator officially began and was released
  for GA in 2017, with continuous improvements ever since
- The Crunchy Operator is one of the first 3 operators ever developed
- The Operator was built from many years of experience with the technology
- In addition we listened closely our clients and how their and other
  development worlds were evolving
- This great history, decades of experience, and observing the evolution
  of development and IT operations all serves as fuel for driving innovation
  in our Operator solution

## Slide 5: Crunchy Postgres for Kubernetes

> Determine exposure to Postgres, Kubernetes

- What is your experience with PostgreSQL today?
- Do you have any stateful applications deployed in Kubernetes?
- Have you used our Operator before?

- Agenda or goals for the meeting

## Slide 6: Easy to Get Started
> Show the GitHub Repository
> Deploy the Operator

```
kc apply -k install/namespace
kc apply --server-side -k install/default
kc get pods -n postgres-operator

kubectl apply -k kustomize/install
kubectl get pods -n postgres-operator
```

> Deploy pgMonitor
```
kubectl apply -k kustomize/monitoring
# kubectl get pods -n pgmonitor
```

**Now that was Easy**

## Slide 7: Fully Declarative
> Introduce concept of Fully Declarative PostgreSQL (manifest)

There are now two categories of methods for building out a database
environment: **Imperative** and **Declarative**
		
### Imperative vs Declarative
- Imperative
  - Step by step process on how to get your infrastructure configured to a desired state.
  - `yum install...`, config wizard, allocate storage, etc.
  - Can be complex
  - Often relies on one, or maybe a few people, running commands or scripts that
    go undocumented, or kept out of version control
  - Configuration creep: How do you know that all your databases are all
    the same?
  - Process does not scale very well
- Declarative
  - The process is made simple
  - Define the final state of the infrastructure and let the provider do the rest
				
With Declarative Postgres, you tell the operator in a documented, easy to understand
manifest about your desired end state Postgres configuration, and the Operator
will perform the necessary steps to make it happen

>   Show the ACCTDEV Postgres Manifest, Deploy Cluster

```
cd ../pgodemo
kubectl apply -k acctdev
```

>   Scale Out the Cluster:  Change replicas to 3
```
kubectl apply -k acctdev
```

> Show PODs in OpenShift or Rancher Console or k9s

> Patch the monitoring spec to include the `LoadBalancer` service for Grafana,
so it can be accessed externally

```bash
kubectl patch service crunchy-grafana -p '{"spec":{"type": "LoadBalancer"}}'
```

> Show monitoring pods in k9s

Get the external IP address of the Grafana service either by looking for the
`crunchy-granafa` pod in the output of `kubectl get services` or run this to
get the IP address directly:

```bash
kubectl get services -l name="crunchy-grafana" \
  -o jsonpath="{.items[0].status.loadBalancer.ingress[0].ip}"
```

Head to `<GRAFANA IP>:3000`, login as admin/admin, set a new password, click on the 
magnifying glass (top left), and `PostgreSQLDetails`

> Exec into Primary Pod, load data, and start benchmark

```
MASTER=$(kubectl get pod -l postgres-operator.crunchydata.com/role=master -o name -n finance)
echo $MASTER
kubectl exec -c database -n finance -it $MASTER -- bash

psql -c "alter user postgres with password 'Welcome1'"
$PGROOT/bin/pgbench --initialize --scale=10 acctdev; $PGROOT/bin/pgbench --time=300 acctdev
```

> PAUSE FOR QUESTIONS

>   Start Jmeter

- Navigate to `View Results in Table` and ensure `Scroll automatically` is checked
- Wait until transactions start to flow through

## Slide 8:  Monitoring

- Production grade starts with good monitoring
- Talk through slide and show various monitoring pages
-	Things to Highlight
	- POD Details: resource utilization from within the cgroup
	- PostgreSQL Details
	- pgBackRest details (note, pgbackrest data only collected every 10 minutes or so)

## Slide 9:  Auto Management and Healing
- Production grade continues with High Availability
- 3 Levels of High Availability: Kubernetes, Operator, Postgres
- These levels are **independent** of each other

HA is occurring at three levels:

- Level 1: **Kubernetes**
	- Typical Kubernetes management.  If I deleted a pod that was generated from a
    deployment/stateful set, Kube will automatically respawn that pod.
		
- Level 2:** Operator**
	- At this level the operator is ensuring the environment always looks like our
    manifest/description
	- Using OpenShift console/k9, demonstrate HA by deleting the acctdev-primary service

> Show how quickly the missing service was detected and corrected

- Level 3: **Postgres**
	- While the operator is ensuring that everything is available at the Kubernetes
    level, a consensus based leader election process is watching the Postgres
    cluster itself
			
It is important to note that the HA handled by this process is independent of the Operator.

So if something happens to Postgres and the Operator is down, the HA failover will still happen.

> Demonstrate HA at 3 Levels

- Kubernetes:  Delete pgBackRest pod and let it restart.
- Operator: Delete acctdev-primary service and watch it get recreated.
- Postgres: Exec into primary and remove data files (show application impact by also showing JMeter)

```
watch kubectl get pod -n finance -L postgres-operator.crunchydata.com/role -l postgres-operator.crunchydata.com/instance

kubectl exec -c database -n finance -it $(kubectl get pod -l postgres-operator.crunchydata.com/role=master -o name -n finance) -- bash


cd /pgdata/
ls pg14
rm -rf pg14*
ls pg14
<will take a bit for restore to occur>
ls
```

## Slide 10:  Update Without Interruption
> Demonstrate rolling minor version upgrade
> Talk about ease of upgrading Operator without impacting Postgres

- Now that we have seen the HA in a disaster scenario, let's look at it
  in more of a planned scenario
	
- In this example we are going to perform a Postgres upgrade due to a
  recent security alert
- This could have been a change like adding a replica, changing resources or
  Postgres parameters
		
> In Jmeter, look in 'View Results Tree' and catch one of the 99sPostgresVersion outputs to see version.

>   Edit the postgres.yaml for acctdev and modify version/image from 14.3 to 14.4

>   Setup Jmeter and split screen with watch command to watch the rolling upgrade.

```
watch kubectl get pod -n finance -L postgres-operator.crunchydata.com/role \
  -l postgres-operator.crunchydata.com/instance

kubectl apply -k acctdev -n finance
```

> In Jmeter, look in 'View Results Tree' and catch one of the 99sPostgresVersion
  outputs to see version.
		
> Stop Jmeter

## Slide 11:   Built for GitOps
> Use ArgoCD to deploy Keycloak
> Use ArgoCD to deploy payments database with overlays

- As you can see, with this Declarative Postgres approach and the powerful automation,
  the Operator Built for GitOps.
		
- Let's see the power of being built for GitOps in action as we deploy an entire
  application stack.
		
- In this example, we are going to deploy Keycloak which requires a Postgres database
  in the backend.

> Show the manifest for the keycloak and postgres.

> Highlight how the Keycloak application is getting database information
  (userid/password) from the secret.

- This is huge as it avoids passing around a password that someone could use
  to bypass the application and perform actions in the database.
		
> Using Argo, create an application called keycloak and deploy the stack (sync)

- Navigate to Application page (top icon on navigation bar)
  - Click on New App button
	- Provide the following:
    ```
		Application Name: keycloak
		Project: default
		Repository URL:  <your github repo>
		Path: keycloak
		Destination:  <only option available>
		Namespace: keycloak
    ```
	- Click `Create`
	- Click `Sync` and then `Synchronize`

> Show the manifest and overlays for payments
> - Base configuration is used to ensure a baseline for all database builds
> - Overlays provide customization to baseline that are specific to each environment
> - Easily answer the question about configuration creep and audit changes through git history
		
> From Argo, delete keycloak application
	
> From Argo, create application for payments-dev, payments-uat and payments-prod and deploy

```
Use payments/overlays/development for Path
```

> After both dev/prod are deployed, make a postgres parameter change to the base
> postgres manifest

- Change `work_mem` from 5MB to 10MB
- Commit change and push to repo
- Click refresh in ArgoCD to show that now dev and prod are out of sync
- Simulate slow rollout by applying to dev first and then prod

## Slide 12: Infrastructure Agnostic
- Deploy anywhere... on-prem/cloud/hybrid
- Many Kubernetes variants: Kubernetes, OpenShift, EKS (AWS), AKS (Azure), GKE (Google), Rancher

## Slide 13: Batteries Included
- Highlight the Postgres versions, many extensions and options
- End with a brief note about UBI and CentOS images

# References

## Useful Links

- [JMeter](https://jmeter.apache.org/download_jmeter.cgi)
- [Crunchy Operator 5.0 Preview Doc](https://pgo5-docs-preview.crunchydata.com/quickstart/)

Delete Argo after demo:
```
kc delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## GitOps
https://www.gitops.tech/
