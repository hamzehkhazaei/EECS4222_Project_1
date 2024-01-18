# Microservice Deployment

In this section, we will deploy a sample microservice application called 
[Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo/)
developed by Google to demonstrate the capabilities of microservice deployment
and their kubernetes engine. To know more about the architecture of Online Boutique,
check out [their documentation](https://github.com/GoogleCloudPlatform/microservices-demo#architecture). You can see a running version of this application [here](https://onlineboutique.dev/).

## Screenshots

| Home Page                                                                                                         | Checkout Screen                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](./img/online-boutique-frontend-1.png)](./img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](./img/online-boutique-frontend-2.png)](./img/online-boutique-frontend-2.png) |

## Deployment

For this project, you will be using a pre-rendered version of this application with
publicly available docker containers. Run the following command on your master node. 

```sh
# deploy the default manifest
kubectl apply -f https://raw.githubusercontent.com/hamzehkhazaei/EECS4222_Project_1/master/files/online-boutique.yaml
```

It may take up to 10 minutes for the deployment to be up and running. You can
wait for all pods to go to the `Running` state using the following command:

```console
$ watch kubectl get pods
NAME                                     READY   STATUS    RESTARTS      AGE
checkoutservice-b888cf998-z8d4q          1/1     Running   0             2m5s
productcatalogservice-79d9bd9666-hpwpm   1/1     Running   0             2m4s
shippingservice-799dd7f6dd-cbh4c         1/1     Running   0             2m1s
redis-cart-69cdbb8ffb-gd7zt              1/1     Running   0             2m
frontend-56d8b4ff94-cxvgm                1/1     Running   0             2m5s
cartservice-5746fbfc7-s9mt4              1/1     Running   0             2m3s
currencyservice-7dd8bb8d8b-xwhwn         1/1     Running   0             2m2s
emailservice-779ff9c7cb-nk6kk            1/1     Running   0             2m5s
paymentservice-8b678b4d8-sgmth           1/1     Running   0             2m4s
recommendationservice-85d4cc98b8-6whhg   1/1     Running   0             2m5s
loadgenerator-6fd975f6b7-f2l4b           1/1     Running   3 (69s ago)   2m3s
adservice-665789b648-hmwtw               1/1     Running   0             119s
```

After the all the services are up and running, you can open the deployed
website by openning the **Microsoft Edge** and going to the address `http://WORKER_IP`. You can also open
the load generator UI by going to the address `http://WORKER_IP:8089`.

Note that you need to replace `WORKER_IP` with the IP of your worker node.

If everything has worked well, you will be able to open the online store
deployed and see the load generator plots and statistics shown. We will later
on use the load generator API to change the number of simulated users and
query the quality of service statistics from the load generator. 

[Next Step](04-loadgenerator.md) -->

## References

- [Online Boutique Development Guide](https://github.com/GoogleCloudPlatform/microservices-demo/blob/master/docs/development-guide.md)
- [Online Boutique Original Deployment](https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml)
