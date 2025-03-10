# Endpointslice

This repository is fork of https://github.com/kubernetes/endpointslice (https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/endpointslice).

This EndpointSlice reconciler library should be sufficiently generic to be used by:
* The Kubernetes EndpointSlice controller
* The Kubernetes EndpointSlice Mirroring controller
* A Gateway API EndpointSelector (See [GEP-3539](https://github.com/kubernetes-sigs/gateway-api/pull/3608)) EndpointSlice Controller
* A Multi-Cluster Service (MCS, See [KEP-1645](https://github.com/kubernetes/enhancements/blob/master/keps/sig-multicluster/1645-multi-cluster-services-api/README.md#using-endpointslice-objects-to-track-endpoints)) EndpointSlice Controller
* An EndpointSlice Controller Selecting IPs in the annotations (e.g. [Multus](https://github.com/k8snetworkplumbingwg/multus-cni))
* An EndpointSlice Controller Selecting IPs in the ResourceClaims of a Pod (e.g. IPs stored in the Device Status of the ResourceClaim, See [KEP-4817](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/4817-resource-claim-device-status))

Orginally, this work comes from the [KEP-4770](https://github.com/kubernetes/enhancements/pull/4771). This part has been removed since but is still accessible from a [previous commit here (77ffcef2914d903b3b443feeaf502888d6f706c5)](https://github.com/kubernetes/enhancements/blob/77ffcef2914d903b3b443feeaf502888d6f706c5/keps/sig-network/4770-endpointslice-controller-flexibility/README.md#flexibility-on-the-endpointslice-reconciler-module-1).

Example Usage (as if it would be used directly in Kubernetes):
```go
// Creates the reconciler.
reconciler = NewReconciler(
    client,
    endpointSliceTracker,
    eventRecorder,
    controllerName,
    maxEndpointsPerSlice,
    endpointSliceMetrics,
    WithTopologyCache(topologyCache),
    WithTrafficDistribution(utilfeature.DefaultFeatureGate.Enabled(features.ServiceTrafficDistribution)),
    WithPlaceholder(true),
    WithOwnershipEnforced(true),
)

// DesiredEndpointSlicesFromServicePods replicates the behavior of the Kubernetes EndpointSlice Controller, so it is used for Services and PodIPs.
desiredEndpointsByAddrTypePort, supportedAddressesTypes, err := DesiredEndpointSlicesFromServicePods(logger, pods, service, nodeLister)
if err != nil {
    return err
}

// The Version and Kind must be set manually.
runtimeObject := service.DeepCopyObject()
runtimeObject.GetObjectKind().SetGroupVersionKind(schema.GroupVersionKind{Version: "v1", Kind: "Service"})

// Reconcile the EndpointSlices.
err = reconciler.Reconcile(
    logger,
    runtimeObject,
    service,
    desiredEndpointsByAddrTypePort,
    supportedAddressesTypes,
    existingSlices,
    service.Spec.TrafficDistribution,
    LabelsFromService{Service: service},
    triggerTime,
)
if err != nil {
    return err
}
```