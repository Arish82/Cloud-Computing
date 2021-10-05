# Deployment with kubernetes Engine
  <ul>
    <b>Create a cluster with five n1-standard-1 nodes</b>
    <p>gcloud container clusters create <cluster-name> --num-nodes <number of nodes> --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"</p>
    <b>The explain command in kubectl can tell us about the Deployment object.</b>
    <p>kubectl explan deployment</p>
    <b>We can also see all of the fields using the --recursive option.</b>
    <p>kubectl explain deployment --recursive</p>
    <b>You can use the explain command as you go through the lab to help you understand the structure of a Deployment object and understand what the individual fields do.</b>
    <p>kubectl explain deployment.metadata.name</p>
    <b>create your deployment object using kubectl create</b>
    <p>kubectl create -f <config file></p>
    <b>Verify</b>
    <p>kubectl get replicasets</p>
    <b>View Pods</b>
    <p>kubectl get pods</p>
    <b>create service file for above config file</b>
    <p>kubectl create -f services/auth.yaml</p>
    <b>create and expose fontend deployment</b>
    <p>kubectl create secret generic tls-certs --from-file tls/<br>
    kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf<br>
    <br>
    kubectl create -f deployments/frontend.yaml<br>
    kubectl create -f services/frontend.yaml
    </p>
    <b>Interact with frontend by external IP adn curling to it</b>
    <p>curl -ks https://<EXTERNAL-IP></p>
    <b>You can also use the output templating feature of kubectl to use curl as a one-liner:</b>
    <p>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`</p>

  </ul>

# Scale Deployment
  <ul>
  <b>Now that we have a Deployment created, we can scale it. Do this by updating the spec.replicas field. You can look at an explanation of this field using the kubectl explain command again.</b>
  <p>kubectl explain deployment.spec.replicas</p>
  <b>The replicas field can be most easily updated using the kubectl scale command</b>
  <p>kubectl scale deployment hello --replicas=5</p>
  <b>Verify that there are now 5 hello Pods running:</b>
  <p>kubectl get pods | grep hello- | wc -l</p>
  <b>Now scale back the application:</b>
  <p>kubectl scale deployment hello --replicas=3</p>
  </ul>
  
# Rolling Updates
  <ul>
  <b>To update your Deployment, run the following command</b>
  <p>kubectl edit deployment hello</p>
  <b>Change the image in the containers section of the Deployment to the following</b>
  <b>See the new ReplicaSet that Kubernetes creates.</b>
  <p>kubectl get replicaset</p>
  <b>You can also see a new entry in the rollout history</b>
  <p>kubectl rollout history deployment/hello</p>
  </ul>
## Pause a rolling update
  <ul>
  <b>If you detect problems with a running rollout, pause it to stop the update.</b>
  <p>kubectl rollout pause deployment/hello</p>
  <b>Verify the current state of the rollout</b>
  <p>kubectl rollout status deployment/hello</p>
  <b>You can also verify this on the Pods directly:</b>
  <p>kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'</p>
  </ul>
  
# Resume Rolling Update
  <ul>
  <b>The rollout is paused which means that some pods are at the new version and some pods are at the older version. We can continue the rollout using the resume command.</b>
  <p>kubectl rollout resume deployment/hello</p>
  
  </ul>
  
# RollBack an update
  <ul>
  <p>Assume that a bug was detected in your new version. Since the new version is presumed to have problems, any users connected to the new Pods will experience those issues.
  <br><br>
  You will want to roll back to the previous version so you can investigate and then release a version that is fixed properly.
  <br><br>
  Use the rollout command to roll back to the previous version:</p>
  <b>kubectl rollout undo deployment/hello</b>
  <br><br>
  <b>Verify the roll back in the history:</b>
  <p>kubectl rollout history deployment/hello</p>
  <b>Finally, verify that all the Pods have rolled back to their previous versions:</b>
  <p>kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'</p>
  </ul>
  
# Canary deployments
<i>When you want to test a new deployment in production with a subset of your users, use a canary deployment. Canary deployments allow you to release a change to a small subset of your users to mitigate risk associated with new releases.</i>
  <ul>
  <h2>Create a canary deployment</h2>
  <b>create a new canary deployment for the new version:</b>
  <p>kubectl create -f deployments/hello-canary.yaml</p>
  <h3>Verify the canary deployment</h3>
  <b>You can verify the hello version being served by the request:</b>
  <p>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version</p>
  <b>Canary deployments in production - session affinity</b>
  <i>In this lab, each request sent to the Nginx service had a chance to be served by the canary deployment. But what if you wanted to ensure that a user didn't get served by the Canary deployment? A use case could be that the UI for an application changed, and you don't want to confuse the user. In a case like this, you want the user to "stick" to one deployment or the other.
  <br>
  You can do this by creating a service with session affinity. This way the same user will always be served from the same version. In the example below the service is the same as before, but a new sessionAffinity field has been added, and set to ClientIP. All clients with the same IP address will have their requests sent to the same version of the hello application</i>
  <p>kind: Service<br>
  apiVersion: v1<br>
  metadata:<br>
    name: "hello"<br>
  spec:<br>
    sessionAffinity: ClientIP<br>
    selector:<br>
      app: "hello"<br>
    ports:<br>
      - protocol: "TCP"<br>
        port: 80<br>
        targetPort: 80</p>
  <i>Due to it being difficult to set up an environment to test this, you don't need to here, but you may want to use sessionAffinity for canary deployments in production.</i>
  </ul>
# Blue-green deployment
  <i>Rolling updates are ideal because they allow you to deploy an application slowly with minimal overhead, minimal performance impact, and minimal downtime. There are instances where it is beneficial to modify the load balancers to point to that new version only after it has been fully deployed. In this case, blue-green deployments are the way to go.<br><br>
  Kubernetes achieves this by creating two separate deployments; one for the old "blue" version and one for the new "green" version. Use your existing hello deployment for the "blue" version. The deployments will be accessed via a Service which will act as the router. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service.</i>
  <ul>
  <h2>The Service</h2>
  <i>Use the existing hello service, but update it so that it has a selector app:hello, version: 1.0.0. The selector will match the existing "blue" deployment. But it will not match the "green" deployment because it will use a different version.</i>
  <br>
  <b>Update the service</b>
  <p>kubectl apply -f services/hello-blue.yaml</p>
  <b>Updating using Blue-Green Deployment</b><br>
  <i>In order to support a blue-green deployment style, we will create a new "green" deployment for our new version. The green deployment updates the version label and the image path.</i>
  <p>apiVersion: apps/v1
  kind: Deployment<br>
  metadata:<br>
    name: hello-green<br>
  spec:<br>
    replicas: 3<br>
    selector:<br>
      matchLabels:<br>
        app: hello<br>
    template:<br>
      metadata:<br>
        labels:<br>
          app: hello<br>
          track: stable<br>
          version: 2.0.0<br>
      spec:<br>
        containers:<br>
          - name: hello<br>
            image: kelseyhightower/hello:2.0.0<br>
            ports:<br>
              - name: http<br>
                containerPort: 80<br>
              - name: health<br>
                containerPort: 81<br>
            resources:<br>
              limits:<br>
                cpu: 0.2<br>
                memory: 10Mi<br>
            livenessProbe:<br>
              httpGet:<br>
                path: /healthz<br>
                port: 81<br>
                scheme: HTTP<br>
              initialDelaySeconds: 5<br>
              periodSeconds: 15<br>
              timeoutSeconds: 5<br>
            readinessProbe:<br>
              httpGet:<br>
                path: /readiness<br>
                port: 81<br>
                scheme: HTTP<br>
              initialDelaySeconds: 5<br>
              timeoutSeconds: 1</p>
  <b>Create the green deployment:</b>
  <p>kubectl create -f deployments/hello-green.yaml</p>
  <b>Once you have a green deployment and it has started up properly, verify that the current version of 1.0.0 is still being used:</b>
  <p>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version</p>
  <b>Update the service to point to the new version:</b>
  <p>kubectl apply -f services/hello-green.yaml</p>
  <b>With the service is updated, the "green" deployment will be used immediately. You can now verify that the new version is always being used.</b>
  <p>curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version</p>
  </ul>
