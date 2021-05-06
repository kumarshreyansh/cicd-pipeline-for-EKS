Step 1:
create a cluster using eksctl:
eksctl create cluster --name cicd-kumar --region=us-west-2 --nodegroup-name ng-1 --node-type
t2.medium --nodes 3


Step 2:
create an ecr and push your image:
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin
<account no>.dkr.ecr.us-west-2.amazonaws.com
docker build -t cicd-demo .
docker tag cicd-demo:latest <account no>.dkr.ecr.us-west-2.amazonaws.com/cicd-demo:latest
docker push <account no>.dkr.ecr.us-west-2.amazonaws.com/cicd-demo:latest


Step 3:
create a codecommit repository and push the codes attached:


Step 4:
Test current version of Application deployment:
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
to update latest image in deployment:
kubectl rollout restart -f deployment.yaml


Step 5:
create a role and map all the required permissions for codebuild to work/communicate with eks cluster,
s3, codecommit, cloudwatch, ecr etc:
run create_iam_role.sh to create role. attach this role with codebuild and make sure to elevate privillege
to support docker runtime environment
grant this role access to eks cluster using below command:
eksctl create iamidentitymapping --cluster cicd-demo --arn arn:aws:iam::<account
no>:role/CodeBuildKubectlRole --group system:masters --username CodeBuildKubectlRole
verify whether role is mapped properly or not:
kubectl get configmaps aws-auth -n kube-system -o yaml > aws-auth.yaml


Step 6:
create a pipeline and add source stage as CodeCommit and add build stage(it should include buildspec
file mentioned in the CodeCommit repository to
perform build function, don't forget to mention above created role to provide access to all the desired
components)


Step 7:
make changes in the CodeCommit repository to see the changes in the pipeline
