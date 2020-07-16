# Application Lifecycle Management Progress: 6 / 23

## Section 4:85

### Rolling Updates and Rollbacks

Show the status of the rollout

    kubectl rollout status deployment/myapp-deployment

View rollout history

    kubectl rollout history deployment/myapp-deployment

#### Deployment strategy

-   Recreate - destroy all and redeploy with new version
-   Rolling Update (default) - destroy and redeploy one pod at a time

New rollout is kicked off once the `kubectl apply -f deployment-definition.yaml` command is ran against an updated deployment definition file.

Another way is the use `kubectl set image deployment/myapp-deployment nginx=nginx:1.91`

Possible to roll back the update

    kubectl rollout undo deployment/myapp-deployment

Creates a deployment and replicaset behind the covers.

    kubectl run nginx --image=nginx
