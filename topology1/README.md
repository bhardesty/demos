### How to easily create an Application Router Network across four clusters using the Application Router ?

* topology1.jpg shows the topology of an Application Router Network  that enables seamless cross cluster/cloud communication even if the Application Routers are not directly connected to each other.
* To create the topology shown in topology1.jpg, we will use the skoot tool to create four yaml files each of which can be applied to four different clusters respectively. The topology needs to be expressed in
  a config file (skoot-topology1.conf) that skoot will read and create the yamls.
* Open skoot-topology1.conf and replace ${NAMESPACE_1}, ${NAMESPACE_2} and ${NAMESPACE_3} with your respective Openshift project names. Replace ${OPENSHIFT_CLUSTER_NAME_1} and ${OPENSHIFT_CLUSTER_NAME_2} with your public
  cluster names respectively. This step is very important for the proper functioning of the Application Routers. If there is any error in the values provided, the Application Routers will be unable to talk to each other
  thus being unable to form a high level Application Network.
* To run skoot follow the directions in the skoot README file - https://github.com/skubaproject/skoot/blob/master/README.md
* Currently, skoot produces yaml files that can only be applied on OpenShift clusters.






