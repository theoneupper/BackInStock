# Back in Stock - Database Server REST API

This Java application implements a CRUD (Create, Read, Update, Delete) REST API for a MongoDB database server that contains all back in stock notification information. This database is the primary data store and source of truth for the state of all notifications.

## Prerequisites

1. Virtual private server or dedicated server with:
    * Java
    * MongoDB
    * Static IP/domain name
2. Gradle
3. SSL certificate (optional, but highly recommended)
    * See **Enabling HTTPS** below for information on creating/obtaining an SSL certificate

## Installing

1. Open your server network ports for inbound connections (defaults: 8080 for http, and 8090 for https)
2. Rename `application.properties.blank` to `application.properties` and update values
3. Build this application's JAR file on a system that has both Java and Gradle installed using the command `./gradlew build`

## Deployment

Copy the executable `RestApi-0.0.1-SNAPSHOT.jar` file to your server and run it. If the JAR does not execute on your system, execute the application with the command `java -jar RestApi-0.0.1-SNAPSHOT.jar`. You may need to remove the `executable = true` line from `build.gradle` and recompile the application first.

For instructions on running this application as a service, or if you are having trouble executing the JAR file, visit: https://docs.spring.io/spring-boot/docs/current/reference/html/deployment-install.html

## Enabling HTTPS

In order to enable HTTPS:
1. Copy your SSL certificate to the same directory has the application JAR file
2. Update SSL settings in `application.properties`

Here are two ways to create a SSL certificate:

### Method 1: Generate a self-signed certificate

Disadvantages: Unless the website visitor manually/explicitly trusts the certificate, it will cause calls to this REST API via HTTPS to fail and/or cause browser security warnings to appear. This method can be useful, however, if you want to test the service in a controlled environment and have not yet procured an SSL certificate.

1. Run the keytool command `keytool -genkey -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650` and ensure:
    * You correctly enter the hostname of the server that you will be running this application on. The keytool may ask "What is your first and last name?"--the hostname should go here.
    * Remember the password and update the `application.properties` file `server.ssl.key-store-password` with this password
    * The generated `keystore.p12` is in the same directory as this application's JAR file
2. Generate a .crt file which will need to be added to the Java CA store on the system running the [Notification Service](../NotificationService). Do this with command `openssl pkcs12 -in keystore.p12 -clcerts -nokeys -out keystore.crt`. Then add the file to your JVM's cacerts file using the commend: `sudo keytool -importcert -file keystore.crt -alias stocknotificationsrestapi -keystore $(/usr/libexec/java_home)/jre/lib/security/cacerts -storepass changeit`. Note that the alias in the Java CA store has been set to `stocknotificationsrestapi`
    * If you make a mistake in the step above, remove the certificate from the Java CA store using the command `sudo keytool -delete -alias stocknotificationsrestapi -keystore $(/usr/libexec/java_home)/jre/lib/security/cacerts -storepass changeit`

### Method 2: Obtain a certificate from a certificate authority.

Disadvantage: May take quite a bit of setup and/or cost money.

For a free CA-signed SSL Certificate visit [Let's Encrypt](https://letsencrypt.org). Check out [this good guide](https://coderwall.com/p/e7gzbq/https-with-certbot-for-nginx-on-amazon-linux) on setting up a a Let's Encrypt SSL certificate with an Amazon Linux server.

## Developer Reference

This project can be easily edited in [Eclipse for Java](http://www.eclipse.org/downloads/eclipse-packages/):
1. Ensure both Gradle and Eclipse are installed
2. Download this repository to your computer
3. In Eclipse, open **File** then **Import...**
4. Under **Gradle**, select **Existing Gradle Project** and click **Next** 
5. On the **Import Gradle Project** click **Browse** and open the root directory of the project
6. Click **Finish**

### Notification Object

The Notification JSON object, Java class and MongoDB document all have the same structure:

* `_id` (String): The unique ID for the object - Generated by MongoDB when the notification is created
* `_class` (String): The equivalent Java class for the object - Added by the Java Spring Data framework when the notification is created 
* `email` (String): Email address of the customer that requested the notification - Submitted via [Frontend](../FrontEnd)
* `variantId` (Integer): The product variant ID that is out stock/this notification is for - Submitted via [Frontend](../FrontEnd)
* `createdDate` (Unix time): The date this notification was created - Generated by `NotificationController` when the notification is created
* `sent` (boolean): Whether an email notification has been sent to the customer - Updated by the [Notification Service](../NotificationService)
* `sentDate` (Unix time): The date the notification was sent to the customer - Updated by the [Notification Service](../NotificationService)

### REST API Endpoints

| Method | EndPoint            | Parameters              | Auth | Description                         |
|--------|---------------------|-------------------------|------|-------------------------------------|
| POST   | /notifications      | none                    | No   | Submit new notification             |
| GET    | /notifications      | none                    | Yes  | Get all notifications               |
|        |                     | sent=[boolean]          |      | Query on sent status == [boolean]   |
|        |                     | createdDate=[unix date] |      | Query on createdDate >= [unix date] |
| GET    | /notifications/{id} | none                    | Yes  | Get a single notification           |
| PUT    | /notifications/{id} | none                    | Yes  | Update a notification               |
| DELETE | /notifications/{id} | none                    | Yes  | Delete a notification               |

**Auth = Yes** means that HTTP basic authentication is required to access these endpoints.

For the `POST` and `PUT` methods: The HTTP body content most contain a Notification object in JSON.

## License

Copyright (c) 2018 [Chris Leung](https://github.com/chrislzm)

Licensed under the MIT License. You may obtain a copy of the License in the [`LICENSE`](LICENSE) file included with this project.
