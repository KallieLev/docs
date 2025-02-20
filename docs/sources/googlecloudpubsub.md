# Google Cloud Pub/Sub source

This event source subscribes to messages sent to a [Google Cloud Pub/Sub][gc-pubsub] topic.

With `tmctl`:

```
tmctl create source googlecloudpubsub --topic <topic> --serviceAccountKey $(cat ./key.txt)
```

On Kubernetes:

```yaml
apiVersion: sources.triggermesh.io/v1alpha1
kind: GoogleCloudPubSubSource
metadata:
  name: sample
spec:
  topic: projects/my-project/topics/my-topic

  serviceAccountKey:
    value: >-
      {
        "type": "service_account",
        "project_id": "my-project",
        "private_key_id": "0000000000000000000000000000000000000000",
        "private_key": "-----BEGIN PRIVATE KEY-----\nMIIE...\n-----END PRIVATE KEY-----\n",
        "client_email": "triggermesh-pubsub-source@my-project.iam.gserviceaccount.com",
        "client_id": "000000000000000000000",
        "auth_uri": "https://accounts.google.com/o/oauth2/auth",
        "token_uri": "https://oauth2.googleapis.com/token",
        "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
        "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/triggermesh-pubsub-source%40my-project.iam.gserviceaccount.com"
      }
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: default
```

Events produced have the following attributes:

* type `com.google.cloud.pubsub.message`
* Schema of the `data` attribute: [com.google.cloud.pubsub.message.json](https://raw.githubusercontent.com/triggermesh/triggermesh/main/schemas/com.google.cloud.pubsub.message.json)

See the [Kubernetes object reference](../../reference/sources/#sources.triggermesh.io/v1alpha1.GoogleCloudPubSubSource) for more details.

## Prerequisite(s)

- Service Account
- Pub/Sub Topic
- Pub/Sub Subscription _(optional)_

### Service Account

A [Service Account][gc-pubsub-svcacc] is required to authenticate the event source and allow it to interact with Google
Cloud Pub/Sub. You can create a service account by following the instructions at [Creating and managing service
accounts][gc-iam-svcacc].

The service account must be granted an [IAM Role][gc-iam-roles] with at least the following permissions:

- `pubsub.subscriptions.consume`
- `pubsub.subscriptions.get`

The following set of permissions is also required if you delegate the management of the Pub/Sub subscription to the
event source. In case you prefer to manage the subscription yourself, these can be safely be omitted. More details on
that topic are provided in the [Pub/Sub Subscription](#pubsub-subscription-optional) section below.

- `pubsub.subscriptions.create`
- `pubsub.subscriptions.delete`
- `pubsub.topics.attachSubscription`

The predefined `roles/pubsub.editor` role is one example of role that is suitable for use with the TriggerMesh event
source for Google Cloud Pub/Sub.

![Service account](../assets/images/googlecloudpubsub-source/iam-1.png)

Create a [key][gc-iam-key] for this service account and save it. This key must be in JSON format. It is required to be
able to run an instance of the Google Cloud Pub/Sub event source.

### Pub/Sub Topic

If you don't already have a Pub/Sub topic to subscribe to, create one by following the instructions at [Managing topics
and subscriptions][gc-pubsub-adm].

Take note of the full [topic name][gc-pubsub-resname], it is a required input to be able to run an instance of the
Google Cloud Pub/Sub event source.

![Topic](../assets/images/googlecloudpubsub-source/topic-1.png)

### Pub/Sub Subscription _(optional)_

A subscription is required in order to allow the TriggerMesh event source for Google Cloud Pub/Sub to pull messages
from a Pub/Sub topic.

This section can be skipped if you would like to let the event source manage its own subscription, which is the default
behaviour. In this case, please simply ensure you granted all necessary permissions to the service account in the
previous section.

If, however, you prefer messages to be pulled using a subscription which you manage yourself, please ensure that
subscription is a "pull" subscription as described in the documentation page [Managing topics and
subscriptions][gc-pubsub-adm].

![Subscription](../assets/images/googlecloudpubsub-source/subscription-1.png)

[gc-pubsub]: https://cloud.google.com/pubsub
[gc-pubsub-svcacc]: https://cloud.google.com/pubsub/docs/authentication#service-accounts
[gc-pubsub-adm]: https://cloud.google.com/pubsub/docs/admin
[gc-pubsub-resname]: https://cloud.google.com/pubsub/docs/admin#resource_names
[gc-iam-svcacc]: https://cloud.google.com/iam/docs/creating-managing-service-accounts
[gc-iam-key]: https://cloud.google.com/iam/docs/creating-managing-service-account-keys
[gc-iam-roles]: https://cloud.google.com/iam/docs/understanding-roles
