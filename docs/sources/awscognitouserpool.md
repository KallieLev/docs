# Amazon Cognito User Pools source

This event source captures messages from an [Amazon Cognito User Pool][cup-docs] whenever a specific action, such as the
creation of a new user, happens in the user identity pool.

With `tmctl`:

```
tmctl create source awscognitouserpool --arn <arn> --auth.credentials.accessKeyID <keyID> --auth.credentials.secretAccessKey <key>
```

On Kubernetes:

```yaml
apiVersion: sources.triggermesh.io/v1alpha1
kind: AWSCognitoUserPoolSource
metadata:
  name: sample
spec:
  arn: arn:aws:cognito-idp:us-west-2:123456789012:userpool/us-west-2_abcdefghi

  auth:
    credentials:
      accessKeyID:
        valueFromSecret:
          name: awscreds
          key: aws_access_key_id
      secretAccessKey:
        valueFromSecret:
          name: awscreds
          key: aws_secret_access_key

  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: default
```

Alternatively you can use an IAM role for authentication instead of an access key and secret, for Amazon EKS only:

```yaml
auth:
  iamrole: arn:aws:iam::123456789012:role/foo
```

To setup an IAM role for service accounts, please refer to the [official AWS documentation](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html).

Events produced have the following attributes:

* type `com.amazon.cognitouserpool.sync_trigger`
* Schema of the `data` attribute: [com.amazon.cognitouserpool.sync_trigger.json](https://raw.githubusercontent.com/triggermesh/triggermesh/main/schemas/com.amazon.cognitouserpool.sync_trigger.json)

See the [Kubernetes object reference](../../reference/sources/#sources.triggermesh.io/v1alpha1.AWSCognitoUserPoolSource) for more details.

## Prerequisite(s)

- Amazon Cognito User Pool
- Amazon Resource Name (ARN)
- API Credentials

### Amazon Cognito User Pool

If you don't already have an Amazon Cognito User Pool, create one by following the instructions in the [Getting started
with User Pools][cup-getting-started] guide.

### Amazon Resource Name (ARN)

A fully qualified ARN is required to uniquely identify the Amazon Cognito User Pool.

![User Pool ARN](../assets/images/awscognitouserpool-source/userpool-arn.png)

As shown in the above screenshot, you can obtain the ARN of a User Pool from the AWS console. It typically has the
following format:

```
arn:aws:cognito-idp:{awsRegion}:{awsAccountId}:userpool/{poolId}
```

Alternatively you can also use the [AWS CLI][aws-cli]. The following command retrieves the ARN of a User Pool in the
`us-west-2` region which has the pool id `us-west-2_fak3p001B`.

```console
$ aws --region us-west-2 cognito-idp describe-user-pool --user-pool-id us-west-2_fak3p001B
{
    "UserPool": {
        "Id": "us-west-2_fak3p001B",
        ...
        "Arn": "arn:aws:cognito-idp:us-west-2:043455440429:userpool/us-west-2_fak3p001B",
        ...
    }
}
```

### API credentials

The TriggerMesh event source for Amazon Cognito User Pools authenticates calls to the AWS API using AWS Access Keys. The
page [Understanding and getting your AWS credentials][accesskey] contains instructions to create access keys when
signed-in either as the root user or as an IAM user. Take note of the **Access Key ID** and **Secret Access Key**, they
will be used to create an instance of the event source.

It is considered a [good practice][iam-bestpractices] to create dedicated users with restricted privileges in order to
programmatically access AWS services. Permissions can be added or revoked granularly for a given IAM user by attaching
[IAM Policies][iam-policies] to it.

As an example, the following policy contains the permissions required by the TriggerMesh Amazon Cognito User Pool event
source to list users in any user pool associated with the AWS account:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AWSCognitoUserPoolSourceReceiveAdapter",
            "Effect": "Allow",
            "Action": [
                "cognito-idp:DescribeUserPool",
                "cognito-idp:ListUsers"
            ],
            "Resource": "arn:aws:cognito-idp:*:*:userpool/*"
        }
    ]
}
```

[cup-docs]: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html
[cup-getting-started]: https://docs.aws.amazon.com/cognito/latest/developerguide/getting-started-with-cognito-user-pools.html
[accesskey]: https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys
[iam-bestpractices]: https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html#iam-user-access-keys
[iam-policies]: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html
[arn]: https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazoncognitouserpools.html
[tm-secret]: ../secrets.md
