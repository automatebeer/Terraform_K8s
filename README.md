# Terraform_K8s
Terraform to Scale pods in K8s

For this setup you need to understand basics of Kubernetes and Terraform

We will be using for setting up K8s cluster

1.  Install kind on the machine 
    brew install kind
  
2.  Download the kind configuration file for port mappings
    curl https://raw.githubusercontent.com/hashicorp/learn-terraform-deploy-nginx-kubernetes-provider/master/kind-config.yaml --output kind-config.yaml

3.  Create the kind K8s cluster
    kind create cluster --name terraform-learn --config kind-config.yaml
   
4.  Verify that kind cluster is created
    kind get clusters
    
5.  We need to point kubectl to point to the kind cluster
    kubectl cluster-info --context kind-terraform-learn
    
6.  Now we need to point the Terraform to our k8s cluster, in our case as we have only one k8s running on our local machine it won't be much of a task

    - mkdir learn-terraform-deploy-nginx-kubernetes
    - cd learn-terraform-deploy-nginx-kubernetes
    - create a file with name kubernetes.tf and add the below line
       provider "kubernetes" {}
       
 7.  You can switch the context to the context we created using the below line
```      kubectl config use-context kind-terraform-learn```
     
 8.   After we have configured our provider, we can initialize the terraform
      ```terraform init```
      
 9.   Now we will create Nginx deployments with 2 replicas and internally exposing on port 80. Add the below code to kubernetes.tf
 
 ```
 resource "kubernetes_deployment" "nginx" {
  metadata {
    name = "scalable-nginx-example"
    labels = {
      App = "ScalableNginxExample"
    }
  }

  spec {
    replicas = 2
    selector {
      match_labels = {
        App = "ScalableNginxExample"
      }
    }
    template {
      metadata {
        labels = {
          App = "ScalableNginxExample"
        }
      }
      spec {
        container {
          image = "nginx:1.7.8"
          name  = "example"

          port {
            container_port = 80
          }

          resources {
            limits {
              cpu    = "0.5"
              memory = "512Mi"
            }
            requests {
              cpu    = "250m"
              memory = "50Mi"
            }
          }
        }
      }
    }
  }
}

 ```
 
 10.  Apply the above configuration to run the nginx deployments
      `terraform apply`
 
 11. To verify the deployments we run successfully run the below command
     `kubectl get deployements`

 12. Now we will expose the nginx service using NodePort, Add the below configs in kubernetes.tf
  ``` 
  resource "kubernetes_service" "nginx" {
  metadata {
    name = "nginx-example"
  }
  spec {
    selector = {
      App = kubernetes_deployment.nginx.spec.0.template.0.metadata[0].labels.App
    }
    port {
      node_port   = 30201
      port        = 80
      target_port = 80
    }

    type = "NodePort"
  }
}
```

 13. `terraform apply`
 
 14. To check if the services are running
      `kubectl get services`
 
 15. You can also change the number of replicas created by changing the count in the kubernetes.tf file where replicas are mentioned. 
 
 16. If once installed and tested you want to destroy the service and setup, use the line
    `terraform destroy`
