# Other Topics Progress: 5 / 5

## Section 14:236

### JSON PATH

<https://kodekloud.com/p/json-path-quiz>

Query language that lets you parse data in a JSON dataset. Top level dictionary is called root and denoted by a `$`

#### Wildcards

`$.*.color`

`$[*].model`

`$.prizes[?(@.year == 2014)].laureates[*].firstname`

#### Lists

From 1st to 4th
`$[0:4]`

Everyother
`$[0:8:2]`

Last item
`$[-1:0]`

#### JSON Path in Kubernetes

add `-o json` to the kubectl command to get the json output.

    kubectl get nodes -o json > /opt/outputs/nodes.jsonpath

then add `-o=jsonpath='{query}'`

Example `kubectl get nodes -o=jsonpath='{$.items[*].metadata.name}'`

    kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
