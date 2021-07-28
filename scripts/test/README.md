## Integration Script
The Integration test script creates an eksctl cluster and runs the Ginkgo Integration tests on the current build from the repository.
### Usage
The Integration test script will **fail to run** on accounts that is not allowlisted for ENI Trunking feature. The test script is currently used in **CI Setup** for the repository `amazon-vpc-resource-controller-k8s`.

Users can still consume this script indirectly by opening a Pull Request on this repository. Once the PR is marked as okay to test, the users can check the status of the test under GitHub Actions.

### Script Execution Order
```
CLUSTER_NAME=<cluster-name>
K8S_VERSION=<k8s-Version>
```

- Create the EKS Cluster.
  ```
  ./create-cluster.sh -n $CLUSTER_NAME -v $K8S_VERSION
  ```

- Create the necessary IAM Policies and Roles
  ```
  ./iam-resources.sh -o create -n $CLUSTER_NAME
  ```
  
- Start test Execution
  ```
  ./test-on-dataplane.sh -n $CLUSTER_NAME
  ```
- Delete the IAM Role and Policies
  ```
  ./iam-resources.sh -o delete -n $CLUSTER_NAME
  ```

- Delete the EKS Cluster
  ```
  ./delete-cluster.sh
  ```

### Design

#### IAM Roles

The controller uses two different IAM Roles.

- The VPCResourceControllerRole on Account A for managing (create, delete, modify..) Trunk and Branch ENI for the Node and Pods on a EKS Cluster on Account A. This account doesn't have to be allowlisted for ENI Trunking.
- The Instance/Node Role on Account B where the controller runs. This account must be allowlisted for ENI Trunking. The controller uses this role for Associating Trunk and Branch Interfaces owned by account A.

The VPCResourceControllerRole on Account A is passed to the controller running on Account B as a runtime argument. For simplicity, the integration test uses the same account for Account A and Account B.

### Future Enhancement
The test script can be deployed using Kops Kubernetes Cluster on AWS with VPC CNI Plugin. This would eliminate the workarounds we have to temporarily disable the controller on EKS for testing purposes.