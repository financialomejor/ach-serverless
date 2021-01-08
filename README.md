# AWS Serverless ACH Service

This project connects to the ACH service in development enviroment, to enable online payments.

## Prerrequisites

- VPN Site to Site setup
- EC2 instance
  - Deployed within the VPC attached to the VPN
  - Reverse proxy configured with tinyproxy, pointing to the service IP
  - IP Whitelist configured in /etc/tinyproxy/tinyproxy.conf
- Eclipse
  - Springboot tools installed in the marketplace
  - Maven installed
  - Setup Axis2 as instructed <a href="https://www.adictosaltrabajo.com/2008/10/24/web-services-axis-2/">here</a>
- Serverless Framework, follow the intructions <a href="https://www.serverless.com/framework/docs/getting-started/">here</a>.

## Installation

First you need to clean the project using these commands:

```bash
mvn clean
```

After this you need to replace the WSDL ip target under the name of <strong>"ACH_SERVICE_URL"</strong> in the <strong>"src > main> resources > services.wsdl"</strong> file with the tinyproxy IP and port. Then, you can generate the Java classes from the WSDL definition using the following command:

<!-- $ sed -i -e 's/ACH_SERVICE_URL/xxx.xxx.xxx.xxx/g' /tmp/file.txt -->

```bash
mvn org.apache.axis2:axis2-wsdl2code-maven-plugin:1.7.9:wsdl2code
```

<!-- The first command intends to replace the "ACH_SERVICE_URL" placeholder for the ip that hosts the ACH service.

<h3>Note: <strong>If you are using Windows you can do this change manually at the end of the "src/main/resources/service.wsdl" file.</strong></h3>

<br> -->

Then you have to reference the generated code directory as a source code directory, if the sources don't show in the project explorer. An example is shown in the following image.

![generated-code-config](https://www.adictosaltrabajo.com/wp-content/uploads/tutorial-data/wss_usertoken/sourceFolder.jpg 'Generated code configuration to access it in project')

### Keystore

After this, you need to generate a keystore in JKS format using any tool. We recommend using Keystore Explorer for this task. The keystore must have the private/public keypair of your certificate and the ACH certificate. It must look like this:

![keystore](./screenshots/keystore.png 'Keystore configuration with keypair and ACH certificate')

<strong>Note:</strong> the private key must have a password or Rampart will fail. If you want to set a password to an existing key, you can execute the following command:

```bash
openssl rsa -aes256 -in your.key -out your.encrypted.key
```

After generating the keystore you need to place it in the <strong>src > main > resources > certificate</strong> folder. Then update the configurations lying in the <strong>src > main > resources > policies.xml</strong> and <strong>com.financialomejor.ach.ACHMainController</strong> files, which are marked with the <strong>TODO</strong> comment. You also have to specify the private key password in the <strong>com.financialomejor.ach.PWCBHandler</strong> class.

## Running the code

To run the springboot service be sure that the <strong>spring-boot-starter-web</strong> dependency has the exclusions commented as follows:

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

Then right click the project folder and choose the <strong>"Run As > Spring Boot App"</strong>. If all configurations where done properly the console must show the running project.

## Build

After the project is configured, you need to install the dependencies and build the project. For this you need to execute the following command:

```bash
mvn clean install package -P assembly-zip
```

This will generate a ZIP file inside the <strong>target/</strong> folder. This zip is ready to be uploaded to a AWS Lambda Function.

## Deploy

To deploy the serverless application to your AWS account you should run the following commands:

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
