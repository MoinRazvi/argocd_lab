Argo CD lab

\-----------------------------------------------------------------------------------

1\) Connect to the eks:  



example: aws eks update-kubeconfig --region <region-name> --name <EKS-CLUSTER-NAME>



Input: aws eks update-kubeconfig --region ap-south-1 --name EKS-Moin-Test1

Output: Updated context arn:aws:eks:ap-south-1:801702847707:cluster/EKS-Moin-Test1 in C:\\Users\\DELL\\.kube\\config



\* If you are not getting the output as mentioned then trouble shoot according the error message and use the chat gpt.



\-----------------------------------------------------------------------------------

2\) Check the cluster details through below commands



Input: kubectx OR kubectl config current-context

Output: arn:aws:eks:ap-south-1:801702847707:cluster/EKS-Moin-Test1



\* To get all the cluster/context: kubectl config get-contexts



\* Below are the important commands for checking the clusters.



&#x09;kubectx                  # List all contexts

&#x09;kubectx <context-name>   # Switch context

&#x09;kubectx -c               # Show current context

&#x09;kubectx -d <context-name> # Delete context



\-----------------------------------------------------------------------------------

3\) Get the end point: 



Input: aws eks describe-cluster \\

&#x20; --region ap-south-1 \\

&#x20; --name EKS-Moin-Test1 \\

&#x20; --query "cluster.endpoint" \\

&#x20; --output text

Output: https://DF9E4FA670AD6BBFDE4F453FFA1806EC.gr7.ap-south-1.eks.amazonaws.com



\* cat \~/.kube/config - we can find all the information about the clusters in this file.

\* \~ - refers to home directory, this the short cut to call the home directory.

\* \~ = /c/Users/DELL/ (in my system) cloud be different in your case

\* check with command "echo \~" to check the home directory.

\* In my case full path is (cat /c/Users/DELL/.kube/config) and short cut is (\~/.kube/config)



\-----------------------------------------------------------------------------------



4\) create the argocd namespace



Input: kubectl create namespace argocd



\-----------------------------------------------------------------------------------



5\) Install Argo CD (you already did / will do) 

Input: kubectl create -n argocd -f  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Output: customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created

customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created



etc....



\-----------------------------------------------------------------------------------



6\) Change service from LoadBalancer to NodePort

Input: kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'



\-----------------------------------------------------------------------------------

7\) Wait for External IP 

Input: kubectl get svc argocd-server -n argocd

output: 



\-----------------------------------------------------------------------------------

8\) Get the public ip of the node

Input: k get nodes -o wide

Output: NAME                                           STATUS   ROLES    AGE   VERSION               INTERNAL-IP     EXTERNAL-IP      OS-IMAGE                        KERNEL-VERSION                    CONTAINER-RUNTIME

ip-172-31-46-225.ap-south-1.compute.internal   Ready    <none>   64m   v1.34.9-eks-254016e   172.31.46.225   13.234.217.132   Amazon Linux 2023.12.20260803   6.12.95-124.187.amzn2023.x86\_64   containerd://2.2.5+unknown



13.234.217.132 - This is the public ip



Check the below ip address, in your case IPs get changed, so take care accordingly. 

**If sites are not working then check the security group and allow the all traffic for your IP. eks will automatically will create the security group like (eks-cluster-sg-Moin-argoCD-535433504) have to modify/allow the all traffic to your node-ip) do not update anything in default security group.**



https://13.234.217.132:80 - This will NOT work (because we updated to Nodeport and ports are different. Range of NodePort is 30000–32767)

https://13.234.217.132:443  - This will NOT work (because we updated to Nodeport and ports are different. Range of NodePort is 30000–32767)

https://13.234.217.132:30900 - This will work

https://13.234.217.132:31921 - This will work



\* Because the node port range is from 30000–32767



\-----------------------------------------------------------------------------------



9\) get the scretes 



kubectl get secret -n argocd (To get the secret information in name space argocd)



kubectl describe secret argocd-initial-admin-secret -n argocd (To check the details of secret information of (argocd-initial-admin-secret)  in name space argocd)



kubectl get secret argocd-initial-admin-secret -n argocd -o yaml (Here you will get the information about the password and need to decode)



echo "ZjR4UTdyaEpVYWx4NnJ3VA==" | base64 -d (This command will give the password which we have to use in the argo CD to access)

userid: admin

password: f4xQ7rhJUalx6rwT (This password is generated after the decoding from the echo command show above)



OR below is command to directly get the password both the methods you can use. 



Direct get the password: kubectl get secret argocd-initial-admin-secret -n argocd \\

&#x20; -o jsonpath="{.data.password}" | base64 -d

&#x20; 

\-----------------------------------------------------------------------------------



10\) grafana-monitoring



create the name space grafana-monitoring



kubectl create namespace grafana-monitoring

kubectl create namespace nginx



\------------------------------------------------------------------------------------



11\)

kubectl get applications -n argocd (to check the applications which are running on argo CD syn status, health status)

kubectl get application grafana -n argocd -o yaml



kubectl apply -f argo-apps/ 



General Kubernetes syntax

Remember this pattern: kubectl describe <resource-type> <resource-name> -n <namespace>







