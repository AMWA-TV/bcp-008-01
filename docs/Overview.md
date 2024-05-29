# AMWA BCP-00X-0X: Receiver status monitoring
{:.no_toc}

* A markdown unordered list which will be replaced with the ToC, excluding the "Contents header" from above
{:toc}

_(c) AMWA 2021, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

![NMOS logo](images/NMOS-logo.png)

## Introduction

The aim of this BCP document is to describe the expectations, behaviour and conformance requirements for Devices with stream Receivers in terms of status monitoring.

This document relies on previous familiarity with the following existing documents:

* [NMOS Control Framework](https://specs.amwa.tv/ms-05-02/)
* [NMOS Control Protocol](https://specs.amwa.tv/is-12/)
* [NMOS Discovery and Registration](https://specs.amwa.tv/is-04/)
* [NMOS Device Connection Management](https://specs.amwa.tv/is-05/)

The technical models referenced in this document are fully published in the [Monitoring NMOS Control Feature Set](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-status-reporting/monitoring/).

The following domains are covered in terms of status monitoring with specific sections for each:

* [Receiver connectivity](#receiver-connectivity)
* [Receiver synchronization](#receiver-synchronization)
* [Receiver stream validation](#receiver-stream-validation)

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Definitions

The NMOS terms 'Controller', 'Node', 'Source', 'Flow', 'Sender', 'Receiver' are used as defined in the [NMOS Glossary](https://specs.amwa.tv/nmos/main/docs/Glossary.html).

## Prerequisites

Devices in conformance to this BCP MUST use [NMOS Control Framework](https://specs.amwa.tv/ms-05-02/) for generating device models.  
Devices in conformance to this BCP MUST use [NMOS Control Protocol](https://specs.amwa.tv/is-12/) to expose device models via a standard API with full support for notifications.  
Devices in conformance to this BCP MUST use [NMOS Discovery and Registration](https://specs.amwa.tv/is-04/) to create and register Nodes, Devices and Receiver resources.  
Devices in conformance to this BCP MUST use [NMOS Device Connection Management](https://specs.amwa.tv/is-05/) to perform connection management actions against Receiver resources.  

## Receiver overall status

The technical model describing the monitoring requirements for a receiver is [NcReceiverMonitor](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-status-reporting/monitoring/#ncreceivermonitor).

This model MUST inherit from the baseline status monitoring model [NcStatusMonitor](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-status-reporting/monitoring/#ncstatusmonitor)

The purpose of the overall status is to abstract and combine the specific domain statuses of a monitor into a single status which can be more easily observed and displayed by a simple client. The overall status might remain the same even when specific domain statuses change but the overall status message might change because a different combination of internal states is causing the current overall status value. A good practice is to populate the status message property with details of the worst status causing the current value of the overall status.

The following recommendations are in place when mapping specific domain statuses in the combined overall status:

* Inactive/Not used states are neutral and do no affect the overall status
* The overall status takes the worst state across the different domains (if one status is PartiallyHealthy (or equivalent) and another is Unhealthy (or equivalent) then the overall status would be Unhealthy)
* The overall status is Healthy only when all domain statuses are either Healthy or a neutral state (Inactive/Not used)

The proposed models are minimal and they can be implemented as is or derived in [vendor specific variants](https://specs.amwa.tv/ms-05-02/latest/docs/Introduction.html) which can add more statuses, properties and methods.

| ![Receiver monitoring model](images/receiver-model-minimal.png) |
|:--:|
| _**Receiver monitoring model**_ |

## Receiver connectivity

The technical model describing the monitoring requirements for a receiver is [NcReceiverMonitor](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-status-reporting/monitoring/#ncreceivermonitor).  
This includes the following specific items which cover the connectivity domain:

* Properties
  * linkStatus
  * linkStatusMessage
  * connectionStatus
  * connectionStatusMessage
* Methods
  * GetLostPackets
  * GetLatePackets
  * ResetPacketCounters

| ![Receiver connectivity](images/receiver-model-connectivity.png) |
|:--:|
| _**Receiver connectivity**_ |

### Link status monitoring

Link status monitoring allows devices to expose the health of all the links associated with the receiver.

Devices specify if:

* All interfaces are Down (equivalent to an Unhealthy state)
* Some of the interfaces are Down (equivalent to a PartiallyHealthy state)
* All of the interfaces are Up (equivalent to a Healthy state)

The link status message is an optional nullable property where devices can offer the reason and further details as to why the current status value was chosen.

### Connection status monitoring

Connection status monitoring allows devices to expose the health of the receiver with regards to receiving stream packets successfully.

Devices specify:

* When the receiver is Inactive (is a neutral state)
* Healthy when the receiver is Active and receiving packets without using any form of loss recovery
* PartiallyHealthy when the receiver is Active and is receiving packets but some form of loss recovery is being used (redundant leg recovery or some form of FEC)
* Unhealthy when the receiver is active and is receiving packets but has unrecoverable errors

The connection status message is an optional nullable property where devices can offer the reason and further details as to why the current status value was chosen.

### Late and lost packets

The receiver monitoring model provides means of gathering metrics around late and lost packets. These are not statuses but instead enable further analysis when [link status](#link-status-monitoring) or [connection status](#connection-status-monitoring) indicate problems.

The feature is expressed with the following methods:

* GetLostPackets - returns a numeric value of the lost packets
* GetLatePackets - returns a numeric value of the late packets
* ResetPacketCounters - allows a client application to reset both the Lost and Late packet counters to 0.

## Receiver synchronization

The technical model describing the monitoring requirements for a receiver is [NcReceiverMonitor](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-status-reporting/monitoring/#ncreceivermonitor).  
This includes the following specific items which cover the synchronization domain:

* Properties
  * synchronizationStatus
  * synchronizationStatusMessage
  * grandMasterClockId

| ![Receiver synchronization](images/receiver-model-synchronization.png) |
|:--:|
| _**Receiver synchronization**_ |

### Synchronization status monitoring

Synchronization status monitoring allows devices to expose the health of the receiver with regards to its time synchronization mechanisms.

Devices specify:

* When the receiver is not using external synchronization (is a neutral state)
* When the receiver is baseband locked (is equivalent to a Healthy state)
* When the receiver is partially baseband locked (is equivalent to a PartiallyHealthy state)
* When the receiver is network locked (is equivalent to a Healthy state)
* When the receiver is partially network locked (is equivalent to a PartiallyHealthy state)
* When the receiver is not locked (is equivalent to an Unhealthy state)

The synchronization status message is an optional nullable property where devices can offer the reason and further details as to why the current status value was chosen.

### Grandmaster change

When devices are configured to use network synchronization they MUST publish the grandmaster clock id currently being used and update the property whenever it changes. For devices which are not using network synchronization this property MUST be set to `null`.

## Receiver stream validation

The technical model describing the monitoring requirements for a receiver is [NcReceiverMonitor](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-status-reporting/monitoring/#ncreceivermonitor).  
This includes the following specific items which cover the stream validation domain:

* Properties
  * streamStatus
  * streamStatusMessage

| ![Receiver stream validation](images/receiver-model-stream-validation.png) |
|:--:|
| _**Receiver stream validation**_ |

### Stream status monitoring

Stream status monitoring allows devices to expose the health of the receiver with regards to the validity of the stream being received.

Devices specify:

* When the receiver is Inactive (is a neutral state)
* Healthy when the receiver is Active and can decode the incoming stream without any errors
* PartiallyHealthy when the receiver is Active and can decode the incoming stream but there are inconsistencies in the stream with what the device is expecting
* Unhealthy when the receiver is active and cannot decode the incoming stream

The stream status message is an optional nullable property where devices can offer the reason and further details as to why the current status value was chosen.
