# eksfpdemo
EKS Flashpaper Demo

    1  pip3 install --user --upgrade boto3
    2  export instance_id=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
    3  python -c "import boto3
    4  import os
    5  from botocore.exceptions import ClientError 
    6  ec2 = boto3.client('ec2')
    7  volume_info = ec2.describe_volumes(
    8      Filters=[
    9          {
   10              'Name': 'attachment.instance-id',
   11              'Values': [
   12                  os.getenv('instance_id')
   13              ]
   14          }
   15      ]
   16  )
   17  volume_id = volume_info['Volumes'][0]['VolumeId']
   18  try:
   19      resize = ec2.modify_volume(    
   20              VolumeId=volume_id,    
   21              Size=30
   22      )
   23      print(resize)
   24  except ClientError as e:
   25      if e.response['Error']['Code'] == 'InvalidParameterValue':
   26          print('ERROR MESSAGE: {}'.format(e))"
   27  sudo curl --silent --location -o /usr/local/bin/kubectl    https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.11/2020-09-18/bin/linux/amd64/kubectl
   28  sudo chmod +x /usr/local/bin/kubectl
   29  sudo pip install --upgrade awscli && hash -r
   30  sudo yum -y install jq gettext bash-completion moreutils
   31  echo 'yq() {
  docker run --rm -i -v "${PWD}":/workdir mikefarah/yq "$@"
}' | tee -a ~/.bashrc && source ~/.bashrc
   32  for command in kubectl jq envsubst aws;   do     which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND";   done
   33  kubectl completion bash >>  ~/.bash_completion
   34  . /etc/profile.d/bash_completion.sh
   35  . ~/.bash_completion
   36  echo 'export LBC_VERSION="v2.0.0"' >>  ~/.bash_profile
   37  .  ~/.bash_profile
   38  rm -vf ${HOME}/.aws/credentials
   39  export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
   40  export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
   41  export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))
   42  test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
   43  echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
   44  echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
   45  echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
   46  aws configure set default.region ${AWS_REGION}
   47  aws configure get default.region
   48  aws sts get-caller-identity --query Arn | grep eksworkshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
   49  cd ~/environment
   50  git clone https://github.com/brentley/ecsdemo-frontend.git
   51  git clone https://github.com/brentley/ecsdemo-nodejs.git
   52  git clone https://github.com/brentley/ecsdemo-crystal.git
   53  ssh-keygen
   54  aws ec2 import-key-pair --key-name "eksworkshop" --public-key-material file://~/.ssh/id_rsa.pub
   55  aws kms create-alias --alias-name alias/eksworkshop --target-key-id $(aws kms create-key --query KeyMetadata.Arn --output text)
   56  export MASTER_ARN=$(aws kms describe-key --key-id alias/eksworkshop --query KeyMetadata.Arn --output text)
   57  echo "export MASTER_ARN=${MASTER_ARN}" | tee -a ~/.bash_profile
   58  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   59  sudo mv -v /tmp/eksctl /usr/local/bin
   60  eksctl version
   61  eksctl completion bash >> ~/.bash_completion
   62  . /etc/profile.d/bash_completion.sh
   63  . ~/.bash_completion
   64  cat << EOF > eksworkshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eksworkshop-eksctl
  region: ${AWS_REGION}
  version: "1.17"

availabilityZones: ["${AZS[0]}", "${AZS[1]}", "${AZS[2]}"]

managedNodeGroups:
- name: nodegroup
  desiredCapacity: 3
  instanceType: t3.small
  ssh:
    allow: true
    publicKeyName: eksworkshop


secretsEncryption:
  keyARN: ${MASTER_ARN}
EOF

   65  eksctl create cluster -f eksworkshop.yaml
   66  kubectl get nodes # if we see our 3 nodes, we know we have authenticated correctly
   67  STACK_NAME=$(eksctl get nodegroup --cluster eksworkshop-eksctl -o json | jq -r '.[].StackName')
   68  ROLE_NAME=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
   69  echo "export ROLE_NAME=${ROLE_NAME}" | tee -a ~/.bash_profile
   70  c9builder=$(aws cloud9 describe-environment-memberships --environment-id=$C9_PID | jq -r '.memberships[].userArn')
   71  if echo ${c9builder} | grep -q user; then rolearn=${c9builder};         echo Role ARN: ${rolearn}; elif echo ${c9builder} | grep -q assumed-role; then         assumedrolename=$(echo ${c9builder} | awk -F/ '{print $(NF-1)}');         rolearn=$(aws iam get-role --role-name ${assumedrolename} --query Role.Arn --output text) ;         echo Role ARN: ${rolearn}; fi
   72  eksctl create iamidentitymapping --cluster eksworkshop-eksctl --arn ${rolearn} --group system:masters --username admin
   73  kubectl describe configmap -n kube-system aws-auth
   74  curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
   75  helm version --short
   76  helm repo add stable https://charts.helm.sh/stable
   77  helm search repo stable
   78  helm completion bash >> ~/.bash_completion
   79  . /etc/profile.d/bash_completion.sh
   80  . ~/.bash_completion
   81  source <(helm completion bash)
   82  cd ~/environment
   83  helm create eksdemo
   84  rm -rf ~/environment/eksdemo/templates/
   85  rm ~/environment/eksdemo/Chart.yaml
   86  rm ~/environment/eksdemo/values.yaml
   87  cat <<EoF > ~/environment/eksdemo/Chart.yaml
apiVersion: v2
name: eksdemo
description: A Helm chart for EKS Workshop Microservices application
version: 0.1.0
appVersion: 1.0
EoF

   88  #create subfolders for each template type
   89  mkdir -p ~/environment/eksdemo/templates/deployment
   90  mkdir -p ~/environment/eksdemo/templates/service
   91  # Copy and rename frontend manifests
   92  cp ~/environment/ecsdemo-frontend/kubernetes/deployment.yaml ~/environment/eksdemo/templates/deployment/frontend.yaml
   93  cp ~/environment/ecsdemo-frontend/kubernetes/service.yaml ~/environment/eksdemo/templates/service/frontend.yaml
   94  # Copy and rename crystal manifests
   95  cp ~/environment/ecsdemo-crystal/kubernetes/deployment.yaml ~/environment/eksdemo/templates/deployment/crystal.yaml
   96  cp ~/environment/ecsdemo-crystal/kubernetes/service.yaml ~/environment/eksdemo/templates/service/crystal.yaml
   97  # Copy and rename nodejs manifests
   98  cp ~/environment/ecsdemo-nodejs/kubernetes/deployment.yaml ~/environment/eksdemo/templates/deployment/nodejs.yaml
   99  cp ~/environment/ecsdemo-nodejs/kubernetes/service.yaml ~/environment/eksdemo/templates/service/nodejs.yaml
  100  cat <<EoF > ~/environment/eksdemo/values.yaml
replicas: 3
version: 'latest'

nodejs:
  image: brentley/ecsdemo-nodejs
crystal:
  image: brentley/ecsdemo-crystal
frontend:
  image: brentley/ecsdemo-frontend
EoF

  101  helm install --debug --dry-run workshop ~/environment/eksdemo
  102  helm install workshop ~/environment/eksdemo
  103  kubectl get svc,po,deploy
  104  kubectl get svc ecsdemo-frontend -o jsonpath="{.status.loadBalancer.ingress[*].hostname}"; echo
  105  helm upgrade workshop ~/environment/eksdemo
  106  kubectl get pods
  107  helm status workshop
  108  helm history workshop
  109  # rollback to the 1st revision
  110  helm rollback workshop 1
  111  helm status workshop
  112  kubectl get pods
  113  helm uninstall workshop
  114  kubectl create namespace argocd
  115  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  116  VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
  117  sudo curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
  118  sudo chmod +x /usr/local/bin/argocd
  119  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
  120  export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname`
  121  ARGO_PWD=`kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2`
  122   argocd login $ARGOCD_SERVER --username admin --password $ARGO_PWD --insecure
  123  env
  124  env |grep ARGO
  125  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
  126  export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname`
  127  kubectl get svc argocd-server -n argocd -o
  128  kubectl get svc argocd-server -n argocd
  129  env |grep ARGO
  130  ARGO_PWD=`kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2`
  131   argocd login $ARGOCD_SERVER --username admin --password $ARGO_PWD --insecure
  132  echo $ARGOCD_SERVER
  133  echo $ARGOCD_PWD
  134  env |grep ARGO
  135  echo $ARGO_PWD
  136  argocd-server-5d7b59fcd-mqb9k
  137  argocd cluster list
  138  ping www.yahoo.com
  139  curl https://github.com/chenian1/demo_eks_argo_helm.git
  140  curl https://github.com/chenian1/demo_eks_argo_helm
  147  mkdir temp
  148  cd temp
  149  helm create fp1
  150  ls
  151  cd fp1
  160  git clone git@github.com:chenian1/demo_eks_argo_helm.git

  194  kubectl get deploy -n argocd -o=yaml

  203  CONTEXT_NAME=`kubectl config view -o jsonpath='{.current-context}'`
  204  argocd cluster add $CONTEXT_NAME
  205  kubectl create namespace ecsdemo-nodejs
  206  argocd app create ecsdemo-nodejs --repo https://github.com/chenian1/ecsdemo-nodejs.git --path kubernetes --dest-server https://kubernetes.default.svc --dest-namespace ecsdemo-nodejs
  207  argocd app get ecsdemo-nodejs
  208  argocd app sync ecsdemo-nodejs

  214  export HELM_EXPERIMENTAL_OCI=1
  215  aws ecr create-repository      --repository-name artifact-test      --region us-west-1
  216  aws ecr get-login-password --region us-west-1  | helm registry login --username AWS --password-stdin 538865708116.dkr.ecr.us-west-1.amazonaws.com
  221  mkdir helm-tutorial
  222  cd helm-tutorial
  223  helm create mychart
  224  rm -rf ./mychart/templates/*
  225  cd mychart/templates
  226  cat <<EOF > configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
EOF

  228  helm chart save . mychart

  233  helm chart save . 538865708116.dkr.ecr.us-west-1.amazonaws.com/artifact-test:mychart
  234  helm chart list
  235  helm chart push 538865708116.dkr.ecr.us-west-1.amazonaws.com/artifact-test:mychart
  236  aws ecr describe-images      --repository-name artifact-test      --region us-west-1
  237  kubectl get svc
  238  export HELM_EXPERIMENTAL_OCI=1

  250  helm install ecr-chart-demo ./mychart
  251  helm get manifest ecr-chart-demo
  252  kubectl get pods --all-namespaces
  253  helm uninstall ecr-chart-demo
  254  kubectl get pods --all-namespaces


  275  cd /home/ec2-user/temp/fp_direct_deploy/
  276  ls
  277  cp /home/ec2-user/environment/ecsdemo-nodejs/kubernetes/deployment.yaml .
  278  cp /home/ec2-user/environment/ecsdemo-nodejs/kubernetes/service.yaml .

  305  aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
  307  kubectl apply -f ./deployment.yaml
  310  more /home/ec2-user/environment/ecsdemo-frontend/kubernetes/deployment.yaml 

  314  kubectl apply -f service.yaml
  315  kubectl get deployment ecsdemo-frontend
  316  kubectl get service ecsdemo-frontend
  317  kubectl get service ecsdemo-frontend -o wide
  318  ELB=$(kubectl get service ecsdemo-frontend -o json | jq -r '.status.loadBalancer.ingress[].hostname')
  319  curl -m3 -v $ELB
  320  kubectl get deployments
  321  kubectl get ns

  413  kubectl delete -f service.yaml 
  414  kubectl delete -f deployment.yaml 
  415  kubectl get deployment ecsdemo-frontend
  416  kubectl get deployment

  430  helm install --debug --dry-run workshop ./fp_helm_deploy
  431  helm install workshop ./fp_helm_deploy
  432  kubectl get svc,po,deploy

  435  ELB=$(kubectl get service ecsdemo-frontend -o json | jq -r '.status.loadBalancer.ingress[].hostname')
  437  curl -m3 -v $ELB
  438  kubectl get svc,po,deploy
  439  kubectl get svc ecsdemo-frontend -o jsonpath="{.status.loadBalancer.ingress[*].hostname}"; echo
  440  kubectl get pods
  441  helm status workshop
  442  helm uninstall workshop
  443  kubectl get pods

  457  helm install --debug --dry-run workshop ./fp_helm_deploy
  
  459  helm install workshop ./fp_helm_deploy

  461  kubectl get svc,po,deploy

  463  ELB=$(kubectl get service ecsdemo-frontend -o json | jq -r '.status.loadBalancer.ingress[].hostname')
  464  curl -m3 -v $ELB

  466  helm uninstall workshop
  
  467  kubectl get svc,po,deploy
  468  kubectl get po
  469  kubectl get deploy
  470  kubectl get svc,po,deploy

  472  ELB=$(kubectl get service ecsdemo-frontend -o json | jq -r '.status.loadBalancer.ingress[].hostname')
  473  curl -m3 -v $ELB
  474  kubectl get svc,po,deploy
  475  kubectl get po
