# AWS Serverless ACH Service

This project connects to the ACH service to enable online payments using serverless through a proxy.

![keystore](./screenshots/ACHArchitecture.png 'Keystore configuration with keypair and ACH certificate')

## Prerrequisites

- VPN Site to Site setup
- VPC with public and private networks with the architecture mentioned in <a href="https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html">here</a>
- Eclipse
  - Springboot tools installed in the marketplace
  - Maven installed
  - Setup Axis2 as instructed <a href="https://www.adictosaltrabajo.com/2008/10/24/web-services-axis-2/">here</a>
- Serverless Framework, follow the intructions <a href="https://www.serverless.com/framework/docs/getting-started/">here</a>.

## Tinyproxy Setup

To bridge requests between your Lambda service and the ACH services, you must setup an EC2 instance as follows:

1. Deploy an Ubuntu 20.x EC2 instance within the network that has the VPN
2. SSH into the created instance
3. Install tinyproxy as follows

```bash
sudo apt-get install tinyproxy
```

4. Copy and paste the **tinyproxy > tinyproxy.conf** file into the **/etc/tinyproxy/tinyproxy.conf** file. Be sure to replace the annotated parameters with your own values.

5. Restart the service

```bash
sudo systemctl restart tinyproxy.service
```

If you need to access the tinyproxy logs in order to ensure that the service is working as expected, you can execute the following command:

```bash
sudo cat /var/log/tinyproxy/tinyproxy.log
```

## Project Setup

First you need to clean the project and install the maven dependencies using the following command:

```bash
mvn clean install
```

### Keystore

After this, you need to generate a keystore in JKS format using any tool. We recommend using Keystore Explorer for this task. The keystore must have the private/public keypair of your certificate and the ACH certificate. It must look like this:

![keystore](./screenshots/keystore.png 'Keystore configuration with keypair and ACH certificate')

<strong>Note:</strong> the private key must have a password or Rampart will fail. If you want to set a password to an existing key, you can execute the following command:

```bash
openssl rsa -aes256 -in your.key -out your.encrypted.key
```

After generating the keystore you need to place it in the <strong>src > main > resources > certificate</strong> folder. Then update the configurations lying in the <strong>src > main > resources > policies.xml</strong> file, which are marked with the <strong>TODO</strong> annotation.

## Running the project

To run the springboot service locally be sure that the <strong>spring-boot-starter-web</strong> dependency has the exclusions commented as follows:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <!-- Comment the exclusions statement to test Spring boot local -->
  <!--
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
  -->
</dependency>
```

Then configure the following environment variables in your Eclipse run configurations:

| Name              | Description                                     |
| ----------------- | ----------------------------------------------- |
| TINYPROXY_HOST    | The url of the tinyproxy reverse proxy endpoint |
| NIT               | The NIT of the organization                     |
| PRIVATE_KEY_PSSWD | The password of the private key of the keystore |

<br/>

Then right click the project folder and choose the <strong>"Run As > Spring Boot App"</strong>. If all configurations where done properly the console must show the running project.

<strong> Be sure that the tinyproxy service running on the EC2 instance allows the connections from your IP.</strong>

## Service routes

You can test that your project is running by using the following command in your machine:

```bash
curl http://localhost/ach/test
```

The allowed routes by the service are the following:

| Route                                    | Method | Description                                                |
| ---------------------------------------- | ------ | ---------------------------------------------------------- |
| /ach/test                                | GET    | Testing route to check running service                     |
| /ach/bank-list                           | GET    | Retrieves the bank list from the ACH servers               |
| /ach/transaction-payment                 | POST   | Creates a new payment transaction                          |
| /ach/transaction-information/{id}        | GET    | Retrieves the information for a given transaction          |
| /ach/transaction-information/detail/{id} | GET    | Retrieves the detailed information for a given transaction |
| /ach/finalize-transaction/{id}           | POST   | Ends a given transaction                                   |
| /ach/transaction-information/detail/{id} | GET    | Retrieves the detailed information for a given transaction |

<br/>

## Build

After the project is configured, you need to install the dependencies and build the project. For this you need to execute the following command:

```bash
mvn clean install package -P assembly-zip
```

This will generate a ZIP file inside the <strong>target/</strong> folder. This zip is ready to be deployed using the Serverless Framework.

## Deploy

To deploy the serverless application to your AWS account you need to first edit the <strong>serverless.yml</strong> file with your AWS parameters. Then you should run the following commands:

```bash
npm i
npx sls deploy
```

After this, the serverless stack should be deployed in your AWS account.

## References

[1] Web Service Security with Axis2, https://www.adictosaltrabajo.com/2009/02/09/wss-usertoken/

[2] Web Services with Axis2, https://www.adictosaltrabajo.com/2008/10/24/web-services-axis-2/

[3] Web Services Security Policy Language, http://specs.xmlsoap.org/ws/2005/07/securitypolicy/ws-securitypolicy.pdf

&#169; Financialo Mejor 2020, all rights reserved
