# AWS Container Day, 2019: Build on AWS with Kubernetes

## Machine Learning on Amazon EKS using Kubeflow Workshop

Go to [AWS Console](https://signin.aws.amazon.com/federation?Action=login&Issuer=localhost&Destination=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fregion%3Dus-west-2&SigninToken=fIiuJSX056jmkJyC8eWNRU4OxgXP5IEiJxoiQ3TDOH2YkFHiviEohSN3uu6pNMhHJXk6hlDu8X4w7lwinVcIwg-CSayv_iFr5vDl-D0W_ffCuUUOhyZ-IkzgbWu-vrnKfMHnHf5Tk5WZ2skcik_SVioo3GCtDxpOgttD1pms41GAez5ntw1nqMoXoxWnt5ZypIxlqrZNPR6H-IMs8OQZtYoxiLkZ1kDnhMeqA5Uy4pTWJ1bxvEqrmbKOfvWOaoU7mpsXape35dQisgFZExr6Hmt7tL1q9C-Cra8sJutn_CZE8R7gPPBSDNtWMS9rwKFccRJz6qu6wQKJnbs3OdNx9ENQy1zK5BKSjOBUBI5ebfPP8bSfId6mNTtfkoJsDQZdgdtumnXsARzsR-_o9yHRYPa9BIEeibNt2Sw4Yh8gW_lRBhhI1ocUTherjX2yWpAw9I1g8pltODRIczTP5p3c_nCfFliH71J0bcbM2w5WjHFhUilssuzSwtpA5kIbZbLuQKBavPAd6Oh-aG82uGsaiQOl-uE49cmGC8ROUwX1oggFJMRWh14iRtARINw-Ouiyg0dPJY8Vxe92vkwfitJHkVtwGpemMf9jr-um6hkw0TgQHxCxC0fv1x2lPzWC89pBrfQZVSBroEo7qRuhAHjMEIduCjLe6M5FVwVowepcNHtBNxsjNa_fEq4UN1EgLQIlO7HhI4O0O1Ock_si380oXo2zeCeTpELaW82WfGhs24MoWrvqsXGEaYoKdtoHeVBzwuluwbAwIOPfsQNBMYFAetyTWeiFiUvjZd12uMhXvfGPXKHkCYx7cnFI0HmMLdi8rxlKUiyj61BmSas_wY7hsI4F9_mU32nKx4y_QixBmsdOmyQoI8AQ4XtRQGMgLQFniOnXOJg8l4A0AFvCOiK-4-Zcd93VxEU2U7DHwE6xOfEqjDdij5l62-xDQnKC9HGp3fe920NqAq6v5JFJF-XoXiDBA7qZIVg7t07KclYAARpn7snNeh5Jq6qT7tuibnhwgsTva9DXLdO2MDJEL_gi9Oi1kguw9OIUiMgssp4KUSqAra1RefYYGYFiooieeOTwQwyFvnuO4I9w832M9bLsB2NilMKTnmcTwNVaA9OBPGgFxTrYYXNRXiWCcAbPxG_aV0bSGKpzo0xWzQ1ZPixl2ztcpZECXzi3cbPhT2QPMzazh3gaAOCLAWnOCBWHIWOupF6XkTikrgKJwnSViOkHKdDR6OaUk8Qkr61445KnmcWeQlrSAsmoIhKdNs-nx23N0auMUvVkuqZT1ccpvf-OiOQSTLnvdeSRB0O-DjUYK8eprFCrSLeoYJGRG97tIm3RNPFqz0EpllWsljNBQagbQ-Nt3cJVvDQ5jaD8WMJMndNBWUTlukfNTKPt0V3bEc_s2Z4E)

### Setup the environment

Install `kubectl`:

```bash
sudo curl --silent --location -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.13.7/bin/linux/amd64/kubectl


sudo chmod +x /usr/local/bin/kubectl

sudo yum -y install jq gettext bash-completion

for command in kubectl jq envsubst
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done

kubectl completion bash >>  ~/.bash_completion
```

Create the IAM role for the workspace ...

Attach the IAM role to the workspace ...

Update the IAM role and clone the demo repositories:

```bash
rm -vf ${HOME}/.aws/credentials

export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region

aws sts get-caller-identity

cd ~/environment
git clone https://github.com/brentley/ecsdemo-frontend.git
git clone https://github.com/brentley/ecsdemo-nodejs.git
git clone https://github.com/brentley/ecsdemo-crystal.git

ssh-keygen

aws ec2 import-key-pair --key-name "eksworkshop" --public-key-material file://~/.ssh/id_rsa.pub
```

Install `eksctl`:

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin

eksctl version

eksctl completion bash >> ~/.bash_completion
```

Create the cluster:

```bash
eksctl create cluster --name=eksworkshop-eksctl --nodes=3 --alb-ingress-access --region=${AWS_REGION}

kubectl get nodes

STACK_NAME=$(eksctl get nodegroup --cluster eksworkshop-eksctl -o json | jq -r '.[].StackName')
INSTANCE_PROFILE_ARN=$(aws cloudformation describe-stacks --stack-name $STACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="InstanceProfileARN") | .OutputValue')
ROLE_NAME=$(aws cloudformation describe-stacks --stack-name $STACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="InstanceRoleARN") | .OutputValue' | cut -f2 -d/)
echo "export ROLE_NAME=${ROLE_NAME}" | tee -a ~/.bash_profile
echo "export INSTANCE_PROFILE_ARN=${INSTANCE_PROFILE_ARN}" | tee -a ~/.bash_profile
```

Deploy Kubernetes Dashboard:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

kubectl proxy --port=8080 --address='0.0.0.0' --disable-filter=true &
```

Open the [dashboard](https://4b1e4141dc5a42d59e9c442700408d43.vfs.cloud9.us-west-2.amazonaws.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/) and use the following token:

```bash
aws eks get-token --cluster-name eksworkshop-eksctl | jq -r '.status.token'
```

### Install Kubeflow

```bash
export NODEGROUP_NAME=$(eksctl get nodegroups --cluster eksworkshop-eksctl -o json | jq -r '.[0].Name')
eksctl scale nodegroup --cluster eksworkshop-eksctl --name $NODEGROUP_NAME --nodes 6

curl --silent --location "https://github.com/kubeflow/kubeflow/releases/download/v0.7.0/kfctl_v0.7.0_linux.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/kfctl /usr/local/bin

export CONFIG_URI=https://raw.githubusercontent.com/kubeflow/manifests/v0.7-branch/kfdef/kfctl_aws.0.7.0.yaml

export AWS_CLUSTER_NAME=eksworkshop-eksctl
export KF_NAME=${AWS_CLUSTER_NAME}

export BASE_DIR=~/environment
export KF_DIR=${BASE_DIR}/${KF_NAME}

curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/aws-iam-authenticator
chmod +x aws-iam-authenticator
sudo mv aws-iam-authenticator /usr/local/bin

mkdir -p ${KF_DIR}
cd ${KF_DIR}
kfctl build -V -f ${CONFIG_URI}

export CONFIG_FILE=${KF_DIR}/kfctl_aws.0.7.0.yaml

sed -i "s@eksctl-eksworkshop-eksctl-nodegroup-ng-a2-NodeInstanceRole-xxxxxxx@$ROLE_NAME@" ${CONFIG_FILE}
sed -i -e 's/kubeflow-aws/'"$AWS_CLUSTER_NAME"'/' ${CONFIG_FILE}
sed -i "s@us-west-2@$AWS_REGION@" ${CONFIG_FILE}

rm -rf kustomize
kfctl apply -V -f ${CONFIG_FILE}

kubectl get pods -n kubeflow

watch kubectl get pods -n kubeflow

kubectl get ingress -n istio-system -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}'

```

Open the Kubeflow Dashboard, create a Notebook server, connect to it, create a new Python 3 notebook and paste the code:

```python
from __future__ import print_function

import tensorflow as tf
from tensorflow import keras

# Helper libraries
import numpy as np
import os
import subprocess
import argparse

# Reduce spam logs from s3 client
os.environ['TF_CPP_MIN_LOG_LEVEL']='3'

def preprocessing():
  fashion_mnist = keras.datasets.fashion_mnist
  (train_images, train_labels), (test_images, test_labels) = fashion_mnist.load_data()

  # scale the values to 0.0 to 1.0
  train_images = train_images / 255.0
  test_images = test_images / 255.0

  # reshape for feeding into the model
  train_images = train_images.reshape(train_images.shape[0], 28, 28, 1)
  test_images = test_images.reshape(test_images.shape[0], 28, 28, 1)

  class_names = ['T-shirt/top', 'Trouser', 'Pullover', 'Dress', 'Coat',
                'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']

  print('\ntrain_images.shape: {}, of {}'.format(train_images.shape, train_images.dtype))
  print('test_images.shape: {}, of {}'.format(test_images.shape, test_images.dtype))

  return train_images, train_labels, test_images, test_labels

def train(train_images, train_labels, epochs, model_summary_path):
  if model_summary_path:
    logdir=model_summary_path # + datetime.now().strftime("%Y%m%d-%H%M%S")
    tensorboard_callback = keras.callbacks.TensorBoard(log_dir=logdir)

  model = keras.Sequential([
    keras.layers.Conv2D(input_shape=(28,28,1), filters=8, kernel_size=3,
                        strides=2, activation='relu', name='Conv1'),
    keras.layers.Flatten(),
    keras.layers.Dense(10, activation=tf.nn.softmax, name='Softmax')
  ])
  model.summary()

  model.compile(optimizer=tf.train.AdamOptimizer(),
                loss='sparse_categorical_crossentropy',
                metrics=['accuracy']
                )
  if model_summary_path:
    model.fit(train_images, train_labels, epochs=epochs, callbacks=[tensorboard_callback])
  else:
    model.fit(train_images, train_labels, epochs=epochs)

  return model

def eval(model, test_images, test_labels):
  test_loss, test_acc = model.evaluate(test_images, test_labels)
  print('\nTest accuracy: {}'.format(test_acc))

def export_model(model, model_export_path):
  version = 1
  export_path = os.path.join(model_export_path, str(version))

  tf.saved_model.simple_save(
    keras.backend.get_session(),
    export_path,
    inputs={'input_image': model.input},
    outputs={t.name:t for t in model.outputs})

  print('\nSaved model: {}'.format(export_path))


def main(argv=None):
  parser = argparse.ArgumentParser(description='Fashion MNIST Tensorflow Example')
  parser.add_argument('--model_export_path', type=str, help='Model export path')
  parser.add_argument('--model_summary_path', type=str,  help='Model summry files for Tensorboard visualization')
  parser.add_argument('--epochs', type=int, default=5, help='Training epochs')
  args = parser.parse_args(args=[])

  train_images, train_labels, test_images, test_labels = preprocessing()
  model = train(train_images, train_labels, args.epochs, args.model_summary_path)
  eval(model, test_images, test_labels)

  if args.model_export_path:
    export_model(model, args.model_export_path)
```

Click on `Run`, add `main()` in the second bock and run it.

### Model training

Build the following Dockerfile

```dockerfile
FROM tensorflow/tensorflow:1.13.1

COPY mnist-tensorflow-docker.py /mnist-tensorflow-docker.py

CMD [ "python", "mnist-tensorflow-docker.py" ]
```

Or use `seedjeffwan/mnist_tensorflow_keras:1.13.1`

Create S3 bucket:

```bash
export HASH=$(< /dev/urandom tr -dc a-z-0-9 | head -c6)
export S3_BUCKET=$HASH-eks-ml-data
aws s3 mb s3://$S3_BUCKET --region $AWS_REGION
```

Setup AWS credentials and apply to EKS:

```bash
aws iam create-user --user-name s3user
aws iam attach-user-policy --user-name s3user --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam create-access-key --user-name s3user > /tmp/create_output.json

export AWS_ACCESS_KEY_ID_VALUE=$(jq -r .AccessKey.AccessKeyId /tmp/create_output.json | base64)
export AWS_SECRET_ACCESS_KEY_VALUE=$(jq -r .AccessKey.SecretAccessKey /tmp/create_output.json | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: aws-secret
type: Opaque
data:
  AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID_VALUE
  AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY_VALUE
EOF
```

Run the training

```bash
curl -LO https://eksworkshop.com/kubeflow/kubeflow.files/mnist-training.yaml
envsubst < mnist-training.yaml | kubectl create -f -

kubectl get pods

kubectl logs mnist-training -f
```



## Sources

- https://eksworkshop.com
- https://www.kubeflow.org
