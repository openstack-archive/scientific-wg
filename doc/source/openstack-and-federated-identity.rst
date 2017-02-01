OpenStack and Federated Identity Management
###########################################

Scientific research depends on the free flow of ideas, and the free
flow of ideas depends upon the free flow of people.  Scientific
collaborations and research groups are often composed of users from
different institutions across different countries.

To support convenient and effective collaboration, compute resources
managed by one institution should be seamlessly accessible to
collaborators from other institutions, and vice versa.  This is the
core principle of identity federation.

OpenStack is uniquely positioned.  Universities and research
institutions use OpenStack to deliver research computing services.
Through improved support for identity federation, OpenStack can
support collaboration between institutions.


Federation Terminology
======================

* **National Research and Education Networks (NREN)** is a collective term used
  for research federations.

* **Home Organisation** is the institution to which a user is affiliated, and in
  federated use cases is where a user will be authenticated.

* **Security Assertion Markup Language (SAML)** ...

* **Open ID Connect (OIDC)** ...

* **Identity Provider (IdP)** ...

* ...

The Concepts of Identity Federation
===================================

NRENs use OpenStack to offer cloud compute services to authorized
users.  The users may belong to universities, research institutions,
industrial partners or more generally to any organisation within
an identify federation.

We have three actors: the NREN running the public cloud, the user
and the user's home organisation, acting as an identity provider.

Federated public clouds built SAML-based Identity Federations,
leveraging Universities and Research Institutions as Identity
Providers.  National Identity Federations are currently used to
authenticate and authorize milions of users, and enable single sign
on across thousands of services.  OpenStack-based public cloud
computing is provided to users through federated identity login.

Openstack services and dashboards need to be configured as Service
Providers within identity federations and have to be accessed like
all other services.

The Challenges of Identity Federation
-------------------------------------

Federated research computing infrastructure needs to enable users
from other institutions within an identity federation to authenticate
and get authorized for access to use infrastructure resources.  How is
this process controlled?

* *A user claims to be a user from another institution.  How is it proven that
  they are who they say they are?*

* *The user has successfully authenticated at their home organisation.
  How are they authorized to use compute resources at this institution?
  Is the affiliation to a project (and role within that project)
  recorded at the home organisation or at the institution providing
  the compute resources?*

* *A federated user wants to run periodic background jobs.  How can
  they do that without having to interactively submit their password?*

* *How does an institution monitor and account for the usage of a
  user who has no presence in the institution's user database?*

* *How is a federated user contacted when required by an organisation,
  and how would disciplinary actions be taken against a federated
  user violating the terms of service?*

* *A federated user has already used a lot of compute resources as
  a guest of this institution.  How does the institutin decide and
  enforce the user's limits?*

* *A federated user has just left her home organisation, or left
  the project on which she was collaborating with this institution.
  How are active resources assigned to this user and project dealt with?*

* *A user is part of an industrial research project in which their
  organisation pays for access to the resources.  How is billing applied to the
  resources used across the identity federation?*

* *How does an institution enable its users to be authenticated and
  authorized to use resources at other institutions within an
  identity federation?*


Federated Identity Management in OpenStack
==========================================

Case Study 1
============

Case Study 2
============

Further Reading
===============

* The INDIGO Identity and Access Management (IAM) service: https://github.com/indigo-iam/iam
* INDIGO Keystone OpenID-Connect integration guide: https://www.gitbook.com/book/indigo-dc/openid-keystone
* ...

Acknowledgements
================

* **Person 1** from organisation 1
* **Person 2** from organisation 2

.. figure:: images/cc-by-sa.png
   :width: 100
   :alt: Creative commons licensing

   This document is provided as open source with a Creative Commons license
   with Attribution + Share-Alike (CC-BY-SA)
