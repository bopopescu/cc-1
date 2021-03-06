..
      Copyright [2010] [Anso Labs, LLC]
 
      Licensed under the Apache License, Version 2.0 (the "License");
      you may not use this file except in compliance with the License.
      You may obtain a copy of the License at
 
          http://www.apache.org/licenses/LICENSE-2.0
 
      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      See the License for the specific language governing permissions and
      limitations under the License.

Auth Documentation
==================    

Nova provides RBAC (Role-based access control) of the AWS-type APIs. We define the following roles:

Roles-Based Access Control of AWS-style APIs using SAML Assertions
“Achieving FIPS 199 Moderate certification of a hybrid cloud environment using CloudAudit and declarative C.I.A. classifications”

Introduction
--------------

We will investigate one method for integrating an AWS-style API with US eAuthentication-compatible federated authentication systems, to achieve access controls and limits based on traditional operational roles.
Additionally, we will look at how combining this approach, with an implementation of the CloudAudit APIs, will allow us to achieve a certification under FIPS 199 Moderate classification for a hybrid cloud environment.

Relationship of US eAuth to RBAC
--------------------------------

Typical implementations of US eAuth authentication systems are structured as follows::

  [ MS Active Directory or other federated LDAP user store ]
  	--> backends to…
  [ SUN Identity Manager or other SAML Policy Controller ]
  	--> maps URLs to groups…
  [ Apache Policy Agent in front of eAuth-secured Web Application ]
       
In more ideal implementations, the remainder of the application-specific account information is stored either in extended schema on the LDAP server itself, via the use of a translucent LDAP proxy, or in an independent datastore keyed off of the UID provided via SAML assertion.

Basic AWS API call structure
----------------------------

AWS API calls are traditionally secured via Access and Secret Keys, which are used to sign API calls, along with traditional timestamps to prevent replay attacks. The APIs can be logically grouped into sets that align with five typical roles:  

*	System User
*	System Administrator
*	Network Administrator
*	Project Manager
*	Cloud Administrator
*	(IT-Sec?)         

There is an additional, conceptual end-user that may or may not have API access:             

*	(EXTERNAL) End-user / Third-party User    

Basic operations are available to any System User:

*	Launch Instance
*	Terminate Instance (their own)
*	Create keypair
*	Delete keypair
*	Create, Upload, Delete: Buckets and Keys (Object Store) – their own
*	Create, Attach, Delete Volume (Block Store) – their own

System Administrators:

*	Register/Unregister Machine Image (project-wide)
*	Change Machine Image properties (public / private)
*	Request / Review CloudAudit Scans

Network Administrator:

*	Change Firewall Rules, define Security Groups
*	Allocate, Associate, Deassociate Public IP addresses

Project Manager:

*	Launch and Terminate Instances (project-wide)
*	CRUD of Object and Block store (project-wide)

Cloud Administrator:

*	Register / Unregister Kernel and Ramdisk Images
*	Register / Unregister Machine Image (any)

Enhancements
------------

*	SAML Token passing 
*	REST interfaces
*	SOAP interfaces

Wrapping the SAML token into the API calls.
Then store the UID (fetched via backchannel) into the instance metadata, providing end-to-end auditability of ownership and responsibility, without PII.

CloudAudit APIs
---------------

*	Request formats
*	Response formats
*	Stateless asynchronous queries

CloudAudit queries may spawn long-running processes (similar to launching instances, etc.) They need to return a ReservationId in the same fashion, which can be returned in further queries for updates.
RBAC of CloudAudit API calls is critical, since detailed system information is a system vulnerability.

Type declarations
---------------------
*	Data declarations – Volumes and Objects
*	System declarations – Instances

Existing API calls to launch instances specific a single, combined “type” flag. We propose to extend this with three additional type declarations, mapping to the “Confidentiality, Integrity, Availability” classifications of FIPS 199. An example API call would look like::

  RunInstances type=m1.large number=1 secgroup=default key=mykey confidentiality=low integrity=low availability=low

These additional parameters would also apply to creation of block storage volumes (along with the existing parameter of ‘size’), and creation of object storage ‘buckets’. (C.I.A. classifications on a bucket would be inherited by the keys within this bucket.)

Request Brokering
-----------------

 *	Cloud Interop
 *	IMF Registration / PubSub
 *	Digital C&A

Establishing declarative semantics for individual API calls will allow the cloud environment to seamlessly proxy these API calls to external, third-party vendors – when the requested CIA levels match.

See related work within the Infrastructure 2.0 working group for more information on how the IMF Metadata specification could be utilized to manage registration of these vendors and their C&A credentials.

Dirty Cloud – Hybrid Data Centers
---------------------------------

*	CloudAudit bridge interfaces
*	Anything in the ARP table

A hybrid cloud environment provides dedicated, potentially co-located physical hardware with a network interconnect to the project or users’ cloud virtual network. 

This interconnect is typically a bridged VPN connection. Any machines that can be bridged into a hybrid environment in this fashion (at Layer 2) must implement a minimum version of the CloudAudit spec, such that they can be queried to provide a complete picture of the IT-sec runtime environment.

Network discovery protocols (ARP, CDP) can be applied in this case, and existing protocols (SNMP location data, DNS LOC records) overloaded to provide CloudAudit information.

The Details
-----------

 *	Preliminary Roles Definitions
 *	Categorization of available API calls
 *	SAML assertion vocabulary

System limits
-------------

The following limits need to be defined and enforced:  

*	Total number of instances allowed (user / project)
*	Total number of instances, per instance type (user / project)
*	Total number of volumes (user / project)
*	Maximum size of volume
*	Cumulative size of all volumes
*	Total use of object storage (GB)
*	Total number of Public IPs


Further Challenges
------------------
 *	Prioritization of users / jobs in shared computing environments
 *	Incident response planning
 *	Limit launch of instances to specific security groups based on AMI
 *	Store AMIs in LDAP for added property control



The :mod:`access` Module
--------------------------

.. automodule:: nova.auth.access
    :members:
    :undoc-members:
    :show-inheritance:

The :mod:`signer` Module
------------------------

.. automodule:: nova.auth.signer
    :members:
    :undoc-members:
    :show-inheritance:

The :mod:`users` Module
-----------------------

.. automodule:: nova.auth.users
    :members:
    :undoc-members:
    :show-inheritance:

The :mod:`users_unittest` Module
--------------------------------

.. automodule:: nova.tests.users_unittest
    :members:
    :undoc-members:
    :show-inheritance:

The :mod:`access_unittest` Module
---------------------------------

.. automodule:: nova.tests.access_unittest
    :members:
    :undoc-members:
    :show-inheritance:


