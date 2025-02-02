<!--
title: Serverless Framework - AWS Lambda Events - Self-managed Apache Kafka
menuText: Kafka
description:  Setting up AWS self-managed Apache Kafka Events with AWS Lambda via the Serverless Framework
layout: Doc
-->

<!-- DOCS-SITE-LINK:START automatically generated  -->

### [Read this on the main serverless docs site](https://www.serverless.com/framework/docs/providers/aws/events/kafka)

<!-- DOCS-SITE-LINK:END -->

# Kafka

A self-managed Apache Kafka cluster can be used as an event source for AWS Lambda.

In order to configure lambda to trigger via `kafka` events, you must provide three required properties:

- `accessConfigurations` which defines the chosen [authentication](#authentication) method configuration
- `topic` to consume messages from
- `bootstrapServers` - an array of bootstrap server addresses for your Kafka cluster

## Authentication

You must authenticate your Lambda with a self-managed Apache Kafka cluster using one of;

- VPC - subnet(s) and security group
- SASL SCRAM/PLAIN - AWS Secrets Manager secret containing credentials
- Mutual TLS (mTLS) - AWS Secrets Manager secret containing client certificate, private key, and optionally a CA certificate

You can provide this configuration via `accessConfigurations`

You must provide at least one method, but it is possible to use VPC in parallel with other methods. For example, you may choose to authenticate via mTLS or SASL/SCRAM, and also place your Lambda and cluster within a VPC.

Valid options for `accessConfigurations` are:

```yaml
saslPlainAuth: arn:aws:secretsmanager:us-east-1:01234567890:secret:SaslPlain
saslScram256Auth: arn:aws:secretsmanager:us-east-1:01234567890:secret:SaslScram256
saslScram512Auth: arn:aws:secretsmanager:us-east-1:01234567890:secret:SaslScram512
clientCertificateTlsAuth: arn:aws:secretsmanager:us-east-1:01234567890:secret:ClientCertificateTLS
serverRootCaCertificate: arn:aws:secretsmanager:us-east-1:01234567890:secret:ServerRootCaCertificate
vpcSubnet:
  - subnet-0011001100
  - subnet-0022002200
vpcSecurityGroup: sg-0123456789
```

For more information see:

- [AWS Documentation - Using Lambda with self-managed Apache Kafka](https://docs.aws.amazon.com/lambda/latest/dg/with-kafka.html#smaa-authentication)

- [AWS Documentation - AWS::Lambda::EventSourceMapping SourceAccessConfiguration](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-lambda-eventsourcemapping-sourceaccessconfiguration.html)

- [Confluent documentation - Authentication with SASL/PLAIN](https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_plain.html)

- [Confluent documentation - Authentication with SASL/SCRAM](https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_scram.html)

- [Confluent documentation Encryption and Authentication with SSL](https://docs.confluent.io/platform/current/kafka/authentication_ssl.html)

## Basic Example: SASL/SCRAM

In the following example, we specify that the `compute` function should be triggered whenever there are new messages available to consume from Kafka topic `MySelfManagedKafkaTopic` from self-hosted cluster at `xyz.com`. The cluster has been authenticated using SASL/SCRAM, the credentials are stored at secret `MyBrokerSecretName`

```yml
functions:
  compute:
    handler: handler.compute
    events:
      - kafka:
          accessConfigurations:
            saslScram512Auth: arn:aws:secretsmanager:us-east-1:01234567890:secret:MyBrokerSecretName
          topic: MySelfManagedKafkaTopic
          bootstrapServers:
            - abc3.xyz.com:9092
            - abc2.xyz.com:9092
```

## Example: Using mTLS

In this example, the lambda event source is a self-managed Apache kafka cluster authenticated via mTLS. The value of `clientCertificateTlsAuth` is an arn of a secret containing the client certificate and privatekey required for the mTLS handshake. The value of `serverRootCaCertificate` is an arn of a secret containing the Certificate Authority (CA) Certificate. This is optional, you only need to provide if your cluster requires it.

```yml
functions:
  compute:
    handler: handler.compute
    events:
      - kafka:
          accessConfigurations:
            clientCertificateTlsAuth: arn:aws:secretsmanager:us-east-1:01234567890:secret:ClientCertificateTLS
            serverRootCaCertificate: arn:aws:secretsmanager:us-east-1:01234567890:secret:ServerRootCaCertificate
          topic: MySelfManagedMTLSKafkaTopic
          bootstrapServers:
            - abc3.xyz.com:9092
            - abc2.xyz.com:9092
```

## Using VPC configurations

You can also specify VPC configurations for your event source. The values will be automatically transformed into their corresponding URI values, so it not required to specify the URI prefix. For example, `subnet-0011001100` will be automatically mapped to the value `subnet:subnet-0011001100`.

```yml
functions:
  compute:
    handler: handler.compute
    events:
      - kafka:
          accessConfigurations:
            vpcSubnet:
              - subnet-0011001100
              - subnet-0022002200
            vpcSecurityGroup: sg-0123456789
          topic: mytopic
          bootstrapServers:
            - abc3.xyz.com:9092
            - abc2.xyz.com:9092
```

## Enabling and disabling Kafka event trigger

The `kafka` event also supports `enabled` parameter, which is used to control if the event source mapping is active. Setting it to `false` will pause polling for and processing new messages.

In the following example, we specify that the `compute` function's `kafka` event should be disabled.

```yml
functions:
  compute:
    handler: handler.compute
    events:
      - kafka:
          accessConfigurations:
            saslScram512Auth: arn:aws:secretsmanager:us-east-1:01234567890:secret:MyBrokerSecretName
          topic: AWSKafkaTopic
          bootstrapServers:
            - abc3.xyz.com:9092
            - abc2.xyz.com:9092
          enabled: false
```

## IAM Permissions

The Serverless Framework will automatically configure the most minimal set of IAM permissions for you. However you can still add additional permissions if you need to. Read the official [AWS documentation](https://docs.aws.amazon.com/lambda/latest/dg/kafka-smaa.html) for more information about IAM Permissions for Kafka events.
