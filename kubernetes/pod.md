# Kubernetes Pod Guide

## Overview

- Pod: A group of one or more containers, with shared storage and network resources
- Pod ID: A unique identifier for a pod
- Pod IP: An IP address assigned to a pod

## Usage

Create and run a particular image in a pod.

### Examples

#### Start a basic nginx pod

```bash
kubectl run nginx --image=nginx

#### Start a hazelcast pod with exposed port

kubectl run hazelcast --image=hazelcast/hazelcast --port=5701

#### Start a hazelcast pod with environment variables

kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

#### Start a hazelcast pod with labels

kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"

#### Dry run to preview pod configuration

kubectl run nginx --image=nginx --dry-run=client

#### Start nginx pod with custom JSON overrides

kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

#### Start an interactive busybox pod

kubectl run -i -t busybox --image=busybox --restart=Never

#### Start nginx pod with custom arguments

kubectl run nginx --image=nginx -- arg1 arg2 ... argN

#### Start nginx pod with custom command and arguments

kubectl run nginx --image=nginx --command -- custom-command arg1 ... argN

``` 