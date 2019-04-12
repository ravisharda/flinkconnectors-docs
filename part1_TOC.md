# Table of Contents
  * [Setting Up Security for a Distributed Mode Cluster](#setting-up-security-for-a-distributed-mode-cluster)
       * [Setting up SSL/TLS](#setting-up-ssltls)
       * [Stage 1: Setting up a Certificate Authority (CA)](#stage-1-setting-up-a-certificate-authority-ca)
       * [Stage 2: Obtaining Server Certificates and keys](#stage-2-obtaining-server-certificates-and-keys)       
              - [Deploying certificates and enabling TLS in Pravega](#deploying-certificates-and-enabling-tls-in-pravega)      
  * [Enabling TLS and Auth in Pravega](#enabling-tls-and-auth-in-pravega)      
       - [Configuring TLS and Auth Parameters for the Services](#configuring-tls-and-auth-parameters-for-the-services)
       - [Configuring TLS and Credentials on the Client Side](#configuring-tls-and-credentials-on-the-client-side)
       - [Server Hostname Verification](#server-hostname-verification)
       - [Having the TLS and Auth parameters take effect](#having-the-tls-and-auth-parameters-take-effect)
   * [Conclusion](#conclusion)
       
       
 # Setting Up Security for a Distributed Mode Cluster

 In the [distributed mode](../deployment/deployment.md#pravega-modes) of running a Pravega cluster, each service runs
separately on one or more processes, usually spread across multiple machines. The deployment options of this mode
include:

 1. A manual deployment in hardware or virtual machines
2. Containerized deployments of these types:
    *   A Kubernetes native application deployed using the Pravega Operator
    * A Docker Compose application deployment
    * A Docker Swarm based distributed deployment

 Regardless of the deployment option used, setting up Transport Layer Security (SSL/TLS) and client auth (short for
authentication and authorization) are important steps towards a secure Pravega deployment.

 TLS encrypts client-server and internal communication. It also enables clients to authenticate the services running on the server nodes. Client auth enables the services to authenticate and authorize the clients. Pravega strongly recommends 
 
 
 
 both TLS and auth, for production clusters.

 Setting up security - especially TLS - in a large cluster can be daunting at first. To make it easier, this document
provides step-by-step instructions on how to enable and configure security manually.

 Depending on the deployment option used and your environment, you might need to modify the steps and commands to
suit your specific needs and policies.

 ## Setting up SSL/TLS

 There are broadly two ways of using TLS for client-server communications:

 1. Setup Pravega to handle TLS directly. In this case, end-to-end traffic is encrypted.
2. Terminate TLS outside of Pravega, in an infrastructure component such as a reverse proxy or a load balancer. Traffic is encrypted until the terminating point and is in plaintext from there to Pravega.

 Depending on the deployment option used, it might be easier to use one or the other approach. For example, if you
are deploying a Pravega cluster in Kubernetes, you might find approach 2 simpler and easier to manage. In a cluster deployed manually on hardware machines, it might be more convenient to use approach 1 in many cases.
The specifics of enabling TLS will also differ depending on the deployment option used.

 Here, we describe the steps applicable for approach 1 in manual deployments. For approach 2, refer to the platform vendor's documentation.

 At a high level, setting up TLS can be divided into three distinct stages:

This conversation was marked as resolved by ravisharda
1. Setting up a Certificate Authority (CA)
2. Obtaining server certificates and keys
3. Deploying certificates and enabling TLS in Pravega

 They are described in detail in the following sub-sections.

 **Before you Begin**

 As the steps in this section use either OpenSSL or Java Keytool, install
OpenSSL and Java Development Kit (JDK) on the hosts that will be used to  generate
TLS certificates, keys, keystores are truststores.

 NOTE:

 * The examples shown in this section use command line arguments to pass all inputs to the command. To pass
sensitive command arguments via prompts instead, just exclude the corresponding option. For example,

 ```
  # Inputs passed as command line arguments
  $ keytool -keystore server01.keystore.jks -alias server01 -validity <validity> -genkey \
              -storepass <keystore-password> -keypass <key-password> \
              -dname <distinguished-name> -ext SAN=DNS:<hostname>,
   # Passwords and other arguments entered interactively on the prompt
  $ keytool -keystore server01.keystore.jks -alias server01 -genkey
  ```
* A weak password `changeit` is used everywhere, for easier reading. Be sure to replace it with a strong and separate
password for each file.

### Stage 1: Setting up a Certificate Authority (CA)
This conversation was marked as resolved by ravisharda

 If you are going to use an existing public or internal CA service or certificate and key bundle, you may skip this part altogether, and go to [Obtaining Server Certificates and keys](#stage-2-obtaining-server-certificates-and-keys).

 Here, we'll generate a CA in the form of a public/private key pair and a self-signed certificate.
Later, we'll use the CA certificate/key bundle to sign server certificates used in the cluster.

 1. Generate a certificate and public/private key pair, for use as a CA.

    ```
       # All inputs provided using command line arguments
         $ openssl req -new -x509 -keyout ca-key -out ca-cert -days <validity> \
            -subj "<distinguished_name>" \
            -passout pass:<strong_password>
       #  Sample command
      $ openssl req -new -x509 -keyout ca-key -out ca-cert -days 365 \
            -subj "/C=US/ST=Washington/L=Seattle/O=Pravega/OU=CA/CN=Pravega-CA" \
            -passout pass:changeit
   
   ```

 2. Create a truststore containing the CA's certificate.

    This truststore will be used by external and internal clients. External clients are client applications connected
   to a Pravega cluster. Services running on server nodes play the role of internal clients when accessing other services.

    ```
      $ keytool -keystore client.truststore.jks -noprompt -alias CARoot -import -file ca-cert \
        -storepass changeit
     # Optionally, list the truststore's contents to verify everything is in order. The output should show
     # a single entry with alias name `caroot` and entry type `trustedCertEntry`.
     $ keytool -list -v -keystore client.truststore.jks -storepass changeit
   ```

 At this point, the following CA and client truststore artifacts shall be  available:

 | File | Description |
|:-----:|:--------|
| `ca-cert` | PEM-encoded X.509 certificate of the CA |
| `ca-key` | PEM-encoded file containing the CA's encrypted private key  |
| `client.truststore.jks` | A password-protected truststore file containing the CA's certificate |

### Stage 2: Obtaining Server Certificates and keys

 This stage is about performing the following steps for each service.


### Configuring TLS and Auth Parameters for the Services

 This step is about using the certificates, keys, keystores and truststores generated earlier to configure
TLS and Auth for the services on the server side.

 Note that if you enable TLS and Auth on one service, you must enable them for all the other services too. Pravega
does not support using TLS and Auth for only some instances of the services.

### Configuring TLS and Credentials on the Client Side

 After enabling and 
 
### Server Hostname Verification

 Hostname verification during TLS communications verifies that the DNS name to which the client connects matches the hostname specified in either of the following fields in the server's certificate:

### Having the TLS and Auth parameters take effect

 To ensure TLS and auth parameters take effect, all the services on the server-side need to be restarted.
Existing client applications will need to be restarted as well, after they are reconfigured for TLS and auth.

 For fresh deployments, starting the cluster and the clients after configuring TLS and auth, will automatically ensure they take effect.

## Conclusion

 This document explained about how to enable security in a Pravega cluster running in distributed mode. Specifically, how to perform the following actions were discussed:

