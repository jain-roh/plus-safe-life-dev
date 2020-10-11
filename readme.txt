Make sure you in this folder and have Kubectl working
1. run the following command on terminal
    export KUBECONFIG=kubeconfig.yaml   
2. Install istio for bookinfo app and service mesh(for the first time, we already have in our cluster)
    curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.4.2 sh -
    cd istio-1.4.2
    export PATH=$PWD/bin:$PATH
    istioctl install --set profile=demo
    cd ..



3. API Gateway -execute for the first when kubecluster is created (not now, since we have our kubecluster already created)
     kubectl apply -f https://www.getambassador.io/yaml/aes-crds.yaml
     kubectl wait --for condition=established --timeout=90s crd -lproduct=aes
     kubectl apply -f https://www.getambassador.io/yaml/aes.yaml
     kubectl -n ambassador wait --for condition=available --timeout=90s deploy -lproduct=aes

3. Get your IP address for apigateway
     kubectl get -n ambassador service ambassador -o "go-template={{range .status.loadBalancer.ingress}}{{or .ip .hostname}}{{end}}"

4. execute the following commands for deploying service
    kubectl apply -f groom-service-deployment.yml
    kubectl apply -f user-auth-deployment.yml
    kubectl apply -f istio-1.4.2/samples/bookinfo/platform/kube/bookinfo.yaml

5. execute following to apply filter-policy, and filter (used for token validation for provided path in filter-policy)
    kubectl apply -f jwt-filter.yml
    kubectl apply -f filter-policy.yml

6. Configure path as per services in ambassador-config.yaml and apply the following yml filter
    kubectl apply -f ambassador-config.yml

7. To check if ambassador api gateway in working:
     Open following in Browser: <ip_address>/productpage/
     (Error for token)
     with token:
     In postman choose Authorisation as Bearer Token and enter following token in the box : eyJhbGciOiJub25lIiwidHlwIjoiSldUIiwiZXh0cmEiOiJzbyBtdWNoIn0.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
     Also hit the following URL <ip_address>/productpage/
        It should return productpage
8. To check if service mesh is working:
    kubectl exec "$(kubectl get pod -l app=ratings --namespace=ambassador -o jsonpath='{.items[0].metadata.name}')" -c ratings --namespace=ambassador -- curl productpage/productpage | grep -o "<title>.*</title>"

    following things make IPC(inter process communication) between rating and productpage

    

    



