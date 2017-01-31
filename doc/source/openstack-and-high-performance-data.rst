OpenStack and High Performance Data
###################################

What can data requirements mean an HPC context?  The range of use cases
is almost boundless.  With considerable generalisation we can consider
some broad criteria for requirements, which expose the inherent tensions
between HPC-centric and cloud-centric storage offerings:

* The **data access** model: data objects could be stored and retrieved
  using file-based, block-based, object-based or stream-based access.
  HPC storage tends to focus on a model of file-based shared data storage
  (with an emerging trend for object-based storage proposed for achieving
  new pinnacles of scalability).  Conversely cloud infrastructure favours
  block-based storage models, often backed with and extended by object-based
  storage.  Support for data storage through shared filesystems is still
  maturing in OpenStack.

* The **data sharing** model: applications may request the same data
  from many clients, or the clients may make data accesses that are
  segregated from one another.  This distinction can have significant
  consequences for storage architecture.  Cloud storage and HPC storage
  are both highly distributed, but often differ in the way in which data
  access is parallelised.  Providing high-performance access for many
  clients to a shared dataset can be a niche requirement specific to HPC.
  Cloud-centric storage architectures typically focus on delivering high
  aggregate throughput on many discrete data accesses.

* The level of **data persistence**.  An HPC-style tiered data storage
  architecture does not need to incorporate data redundancy at every level
  of the hierarchy.  This can improve performance for tiers caching data
  closer to the processor.

The cloud model offers capabilities that enable new possibilities for HPC:

* **Automated provisioning**.  Software-defined infrastructure automates the
  provisioning and configuration of compute resources, including storage.
  Users and group administrators are able to create and configure storage
  resources to their specific requirements at the exact time they are
  needed.
* **Multi-tenancy**.  HPC storage does not offer multi-tenancy with the level
  of segregation that cloud can provide.  A virtualised storage resource
  can be reserved for the private use of a single user, or could be shared
  between a controlled group of collaborating users, or could even be
  accessible by all users.
* **Data isolation**.  Sensitive data requires careful data management.
  Medical informatics workloads may contain patient genomes.  Engineering
  simulations may contain data that is trade secret.  OpenStack’s
  segregation model is stronger than ownership and permissions on a
  POSIX-compliant shared filesystem, and also provides finer-grained
  access control.

There is clear value in increased flexibility - but at what cost in
performance?  In more demanding environments, HPC storage tends to focus
on and be tuned for delivering the requirements of a confined subset
of workloads.  This is the opposite approach to the conventional cloud
model, in which assumptions may not be possible about the storage access
patterns of the supported workloads.

This study will describe some of these divergences in greater detail, and
demonstrate how OpenStack can integrate with HPC storage infrastructure.
Finally some methods of achieving high performance data management on
cloud-native storage infrastructure will be discussed.

File-based Data: HPC Parallel Filesystems in OpenStack
======================================================

Conventionally in HPC, file-based data services are delivered
by parallel filesystems such as Lustre and Spectrum Scale (GPFS).
A parallel filesystem is a shared resource.  Typically it is mounted on
all compute nodes in a system and available to all users of a system.
Parallel filesystems excel at providing low-latency, high-bandwidth
access to data.

Parallel filesystems can be integrated into an OpenStack environment in
a variety of configuration models.

Provisioned Client Model
------------------------

Access to an external parallel filesystem is provided through an OpenStack
provider network.  OpenStack compute instances - virtualised or bare
metal - mount the site filesystem as clients.

This use case is fairly well established.  In the virtualised use case,
performance is achieved through use of SR-IOV (with only a moderate
level of overhead).  In the case of Lustre, with a layer-2 VLAN provider
network the o2ib client drivers can use RoCE to perform Lustre data
transport using RDMA.

Cloud-hosted clients on a parallel filesystem raise issues with root in
a cloud compute context.  On cloud infrastructure, privileged accesses
from a client do not have the same degree of trust as on conventional HPC
infrastructure.  Lustre approaches this issue by introducing Kerberos
authentication for filesystem mounts and subsequent file accesses.
Kerberos credentials for Lustre filesystems can be supplied to OpenStack
instances upon creation as instance metadata.

Provisioned Filesystem Model
----------------------------

There are use cases where the dynamic provisioning of software-defined
parallel filesystems has considerable appeal.  There have been
proof-of-concept demonstrations of provisioning Lustre filesystems from
scratch using OpenStack compute, storage and network resources.

The OpenStack Manila project aims to provision and manage shared
filesystems as an OpenStack service.  IBM’s Spectrum Scale integrates
with Manila to re-export GPFS parallel filesystems using the user-space
Ganesha NFS server.

Currently these projects demonstrate functionality over performance.
In future evolutions the overhead of dynamically provisioned parallel
filesystems on OpenStack infrastructure may improve.

A Parallel Data Substrate for OpenStack Services
------------------------------------------------

IBM positions Spectrum Scale as a distributed data service for
underpinning OpenStack services such as Cinder, Glance, Swift and Manila.
More information about using Spectrum Scale in this manner can be found
in IBM Research’s red paper on the subject (listed in the Further
Reading section).

Applying HPC Technologies to Enhance Data IO
============================================

A recurring theme throughout this study has been the use of remote DMA
for efficient data transfer in HPC environments.  The advantages of this
technology are especially pertinent in data intensive environments.
OpenStack’s flexibility enables the introduction of RDMA protocols
for many cloud infrastructure operations to reduce latency, increase
bandwidth and enhance processor efficiency:

Cinder block data IO can be performed using iSER (iSCSI extensions
for RDMA).  iSER is a drop-in replacement for iSCSI that is easy to
configure and set up.  Through providing tightly-coupled IO resources
using RDMA technologies, the functional equivalent of HPC-style burst
buffers can be added to the storage tiers of cloud infrastructure.

Ceph data transfers can be performed using the Accelio RMDA transport.
This technology was demonstrated some years ago but does not appear
to have achieved production levels of stability or gained significant
mainstream adoption.

The NOWLAB group at Ohio State University have developed extensions to
data analytics platforms such as HBase, Hadoop, Spark and Memcached to
optimise data movements using RDMA.

Optimising Ceph Storage for Data-Intensive Workloads
====================================================

The versatility of Ceph embodies the cloud-native approach to storage,
and consequently Ceph has become a popular choice of storage technology
for OpenStack infrastructure.  A single Ceph deployment can support
various protocols and data access models.

Ceph is capable of delivering strong read bandwidth.  For large reads
from OpenStack block devices, Ceph is able to parallelise the delivery
of the read data across multiple OSDs.

Ceph’s data consistency model commits writes to multiple OSDs before
a write transaction is completed.  By default a write is replicated
three times.  This can result in higher latency and lower performance
on write bandwidth.

Ceph can run on clusters of commodity hardware configurations.  However,
in order to maximise the performance (or price performance) of a Ceph
cluster some design rules of thumb can be applied:

Use separate physical network interfaces for external storage network and
internal storage management.  On the NICs and switches, enable Ethernet
flow control and raise the MTU to support jumbo frames.

Each drive used for Ceph storage is managed by an OSD process.
A Ceph storage node usually contains multiple drives (and multiple
OSD processes).

The best price/performance and highest density is achieved using fat
storage nodes, typically containing 72 HDDs.  These work well for
large scale deployments, but can lead to very costly units of failure
in smaller deployments.  Node configurations of 12-32 HDDs are usually
found in deployments of intermediate scale.

Ceph storage nodes usually contain a higher-speed write journal, which is
dedicated to service of a number of HDDs.  An SSD journal can typically
feed 6 HDDs while an NVMe flash device can typically feed up to 20 HDDs.

About 10G of external storage network bandwidth balances the read
bandwidth of up to 15 HDDs.  The internal storage management network
should be similarly scaled.

A rule of thumb for RAM is to provide 0.5GB-1GB of RAM per TB per
OSD daemon.

On multi-socket storage nodes, close attention should be paid to NUMA
considerations.  The PCI storage devices attached to each socket should
be working together.  Journal devices should be connected with HDDs
attached to HBAs on the same socket.  IRQ affinity should be confined
to cores on the same socket.  Associated OSD processes should be pinned
to the same cores.

For tiered storage applications in which data can be regenerated from
other storage, the replication count can safely be reduced from 3 to
2 copies.

The Cancer Genome Collaboratory: Large-scale Genomics on OpenStack
==================================================================

Genome datasets can be hundreds of terabytes in size, sometimes requiring
weeks or months to download and significant resources to store and
process.

.. image:: images/high_performance_data-oicr_logo.jpg
   :width: 300
   :align: right
   :alt: OICR logo

The Ontario Institute for Cancer Research built the Cancer Genome
Collaboratory (or simply The Collaboratory) as a biomedical research
resource built upon OpenStack infrastructure.  The Collaboratory aims
to facilitate research on the world’s largest and most comprehensive
cancer genome dataset, currently produced by the International Cancer
Genome Consortium (ICGC).

By making the ICGC data available in cloud compute form in the
Collaboratory, researchers can bring their analysis methods to the cloud,
yielding benefits from the high availability, scalability and economy
offered by OpenStack, and avoiding the large investment in compute
resources and the time needed to download the data.

An OpenStack Architecture for Genomics
--------------------------------------

The Collaboratory’s requirements for the project were to build a cloud
computing environment providing 3000 compute cores and 10-15 PB of raw
data stored in a scalable and highly-available storage.  The project
has also met constraints of budget, data security, confined data centre
space, power and connectivity.  In selecting the storage architecture,
capacity was considered to be more important than latency and performance.

Each rack hosts 16 compute nodes using 2U high-density chassis, and
between 6 and 8 Ceph storage nodes.  Hosting a mix of compute and storage
nodes in each rack keeps some of the Nova-Ceph traffic in the same rack,
while also lowering the power requirement for these high density racks
(2 x 60A circuits are provided to each rack).

As of September 2016, Collaboratory has 72 compute nodes (2600 CPU
cores, Hyper-Threaded) with a physical configuration optimized for large
data-intensive workflows: 32 or 40 CPU cores and a large amount of RAM
(256 GB per node).  The workloads make extensive use of high performance
local disk, incorporating hardware RAID10 across 6 x 2TB SAS drives.

The networking is provided by Brocade ICX 7750-48C top-of-rack switches
that use 6x40Gb cables to interconnect the racks in a ring stack topology,
providing 240 Gbps non-blocking redundant inter-rack connectivity,
at a 2:1 oversubscription ratio.

The Collaboratory is deployed using entirely community-supported free
software.  The OpenStack control plane is Ubuntu 14.04 and deployment
configuration is based on Ansible.  The Collaboratory was initially
deployed using OpenStack Juno and a year later upgraded to Kilo and
then Liberty.

Collaboratory deploys a standard HA stack based on Haproxy/Keepalived and
Mariadb-Galera using three controller nodes.  The controller nodes also
perform the role of Ceph-mon and Neutron L3-agents, using three separate
RAID1 sets of SSD drives for MySQL, Ceph-mon and Mongodb processes.

The compute nodes have 10G Ethernet with GRE and SDN capabilities
for virtualized networking.  The Ceph nodes use 2x10G NICs bonded for
client traffic and 2x10G NICs bonded for storage replication traffic.
The Controller nodes have 4x10G NICs in an active-active bond (802.3ad)
using layer3+4 hashing for better link utilisation.  The Openstack tenant
routers are highly-available with two routers distributed across the three
controllers.  The configuration does not use Neutron DVR out of concern
for limiting the number of servers directly attached to the Internet.
The public VLAN is carried only on the trunk ports facing the controllers
and the monitoring server.

Optimising Ceph for Genomics Workloads
--------------------------------------

Upon workload start, the instances usually download data stored in Ceph's
object storage.  OICR developed a download client that controls access
to sensitive ICGC protected data through managed tokens.  Downloading a
100GB file stored in Ceph takes around 18 minutes, with another 10-12
minutes used to automatically check its integrity (md5sum), and is mostly
limited by the instance’s local disk.

The ICGC storage system adds a layer of control on top of Ceph’s
object storage.  Currently this is a 2-node cluster behind an Haproxy
instance serving the ICGC storage client.  The server component uses
OICR’s authorization and metadata systems to provide secure access to
related objects stored in Ceph.  By using OAuth-based access tokens,
researchers can be given access to the Ceph data without having to
configure Ceph permissions.  Access to individual project groups can
also be implemented in this layer.

Each Ceph storage node consists of 36 OSD drives (4, 6 or 8 TB) in
a large Ceph cluster currently providing 4 PB of raw storage, using
three replica pools.  The radosgw pool has 90% of the Ceph space being
reserved for storing protected ICGC datasets, including the very large
whole genome aligned reads for almost 2000 donors.  The remaining 10% of
Ceph space is used as a scalable and highly-available backend for Glance
and Cinder.  Ceph radosgw was tuned for the specific genomic workloads,
mostly by increasing read-ahead on the OSD nodes, 65 MB as rados object
stripe for Radosgw and 8 MB for RBD.

Further Considerations and Future Directions
--------------------------------------------

In the course of the development of the OpenStack infrastructure at the
Collaboratory, several issues have been encountered and addressed:

The instances used in cancer research are usually short lived
(hours/days/weeks), but with high resource requirements in terms of CPU
cores, memory and disk allocation.  As a consequence of this pattern of
usage the Collaboratory OpenStack infrastructure does not support live
migration as a standard operating procedure.

The Collaboratory have encountered a few problems caused by Radosgw bugs
involving overlapping multipart uploads.  However, these were detected by
the Collaboratory’s monitoring system, and did not result in data loss.
The Collaboratory created a monitoring system that uses automated Rally
tests to monitor end-to-end functionality, and also download a random
large S3 object (around 100 GB) to confirm data integrity and monitor
object storage performance.

Because of the mix of very large (BAM), medium (VCF) and very small
(XML, JSON) files, the Ceph OSD nodes have imbalanced load and we have
to regularly monitor and rebalance data.

Currently, the Collaboratory is hosting 500TB of data from 2,000 donors.
Over the next 2 years, OICR will increase the number of ICGC genomes
available in the Collaboratory, with the goal of having the entire ICGC
data set of 25,000 donors estimated to be 5PB when the project completes
in 2018.

Although in a closed beta phase with only a few research labs having
accounts, there were more than 19,000 instances started in the last 18
months, with almost 7,000 in the last three months.  One project that
uses the Collaboratory heavily is the PanCancer Analysis of Whole Genomes
(PCAWG), which characterizes the somatic, and germline variants from
over 2,800 ICGC cancer whole genomes in 20 primary tumour sites.

In conclusion, the Collaboratory environment has been running well for
OICR and its partners.  George Mihaiescu, senior cloud architect at OICR,
has many future plans for OpenStack and the Collaboratory:

   “We hope to add new Openstack projects to the Collaboratory’s offering
   of services, with Ironic and Heat being the first candidates.  We would
   also like to provide new compute node configurations with RAID0 instead
   of RAID10, or even SSD based local storage for improved IO performance.”

CLIMB: OpenStack, Parallel Filesystems and Microbial Bioinformatics
===================================================================

The Cloud Infrastructure for Microbial Bioinformatics (CLIMB) is a
collaboration between four UK universities (Swansea, Warwick, Cardiff
and Birmingham) and funded by the UK’s Medical Research Council.
CLIMB provides compute and storage as a free service to academic
microbiologists in the UK.  After an extended period of testing, the
CLIMB service was formally launched in July 2016.

.. image:: images/high_performance_data-climb.jpg
   :width: 400
   :align: right
   :alt: CLIMB hardware

CLIMB is a federation of 4 sites, configured as OpenStack regions.
Each site has an approximately equivalent configuration of compute nodes,
network and storage.

The compute node hardware configuration is tailored to support the
memory-intensive demands of bioinformatics workloads.  The system as
a whole comprises 7680 CPU cores, in fat 4-socket compute nodes with
512GB RAM.  Each site also has three large memory nodes with 3TB of RAM
and 192 hyper-threaded cores.

The infrastructure is managed and deployed using xCAT cluster management
software.  The system runs the Kilo release of OpenStack, with packages
from the RDO distribution.  Configuration management is automated
using Salt.

Each site has 500 TB of GPFS storage.  Every hypervisor is a GPFS client,
and uses an infiniband fabric to access the GPFS filesystem.  GPFS is
used for scratch storage space in the hypervisors.

For longer term data storage, to share datasets and VMs, and to provide
block storage for running VMs, CLIMB deploys a storage solution based
on Ceph.  The Ceph storage is replicated between sites.  Each site has 27
Dell R730XD nodes for Ceph storage servers.  Each storage server contains
16x 4TB HDDs for Ceph OSDs, giving a total raw storage capacity of 6912TB.
After 3-way replication this yields a usable capacity of 2304TB.

On two sites Ceph is used as the storage back end for Swift, Cinder
and Glance.  At Birmingham GPFS is used for Cinder and Glance, with
plans to migrate to Ceph.

In addition to the infiniband network, a Brocade 10G Ethernet fabric is
used, in conjunction with dual-redundant Brocade Vyatta virtual routers
to manage cross-site connectivity.

In the course of deploying and trialling the CLIMB system, a number of
issues have been encountered and overcome.

* The Vyatta software routers were initially underperforming with
  consequential impact on inter-site bandwidth.
* Some performance issues have been encountered due to NUMA topology
  awareness not being passed through to VMs.
* Stability problems with Broadcom 10GBaseT drivers in the controllers
  led to reliability issues.  (Thankfully the HA failover mechanisms were
  found to work as required).
* Problems with interactions between Ceph and Dell hardware RAID cards.
* Issues with Infiniband and GPFS configuration.

CLIMB has future plans for developing their OpenStack infrastructure,
including:

* Migrating from regions to Nova cells as the federation model between
  sites.
* Integrating OpenStack Manila for exporting shared filesystems from
  GPFS to guest VMs.

Further Reading
===============

An IBM research study on integrating GPFS
(Spectrum Scale) within OpenStack environments:
http://www.redbooks.ibm.com/redpapers/pdfs/redp5331.pdf

A 2015 presentation from ATOS on using Kerberos authentication in Lustre:
http://cdn.opensfs.org/wp-content/uploads/2015/04/Lustre-and-Kerberos_Buisson.pdf

Glyn Bowden of HPE and Alex Macdonald from SNIA discuss OpenStack
storage (including the Provisioned Filesystem Model using Lustre):
https://www.brighttalk.com/webcast/663/168821

The High-Performance Big Data team at Ohio State University:
http://hibd.cse.ohio-state.edu

A useful talk from the 2016 Austin OpenStack Summit on Ceph design:
https://www.openstack.org/videos/video/designing-for-high-performance-ceph-at-scale

The Ontario Institute for Cancer Research Collaboratory:
http://www.cancercollaboratory.org

Further details on the International Cancer Genome Consortium:
http://icgc.org/

Dr Tom Connor presented CLIMB at the 2016 Austin OpenStack summit:
https://www.openstack.org/videos/video/the-cloud-infrastructure-for-microbial-bioinformatics-breaking-biological-silos-using-openstack

Acknowledgements
================

This document was written by Stig Telfer of `StackHPC Ltd <https://www.stackhpc.com>`_ with the support
of Cambridge University, with contributions, guidance and feedback from
subject matter experts:

* **George Mihaiescu**, **Bob Tiernay**, **Andy Yang**, **Junjun Zhang**,
  **Francois Gerthoffert**, **Christina Yung**, **Vincent Ferretti**
  from the Ontario Institute for Cancer Research.  The authors wish
  to acknowledge the funding support from the Discovery Frontiers:
  Advancing Big Data Science in Genomics Research program (grant
  no. RGPGR/448167-2013, ‘The Cancer Genome Collaboratory’), which
  is jointly funded by the Natural Sciences and Engineering Research
  Council (NSERC) of Canada, the Canadian Institutes of Health Research
  (CIHR), Genome Canada, and the Canada Foundation for Innovation (CFI),
  and with in-kind support from the Ontario Research Fund of the Ministry
  of Research, Innovation and Science.
* **Dr Tom Connor** from Cardiff University and the CLIMB collaboration.

.. figure:: images/cc-by-sa.png
   :width: 100
   :alt: Creative commons licensing

   This document is provided as open source with a Creative Commons license
   with Attribution + Share-Alike (CC-BY-SA)
