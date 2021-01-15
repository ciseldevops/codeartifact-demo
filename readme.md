<a href="https://www.cisel.ch/"><img src="https://www.cisel.ch/wp-content/uploads/2019/05/cisel_80.png" title="CISEL" alt="CISEL"></a>

<!-- [![CISEL](https://www.cisel.ch/)](https://www.cisel.ch/) -->


If you are looking to store your Artifcats online in a secure way, take a try with Amazon CodeArtifact. This can be an alternative to JFrog Artifactory.

In this article, we describe the way to push a maven artifact from the gitlab CI to AWS CodeArtifact. 

First of all, you need to create a repository on AWS CodeArtifact. You can try the "Get started" button on the aws cloud console.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1610549117896/YmhPWXKrR.png)

In the repository creation, you have to enter the name of your repository and choose your domain. After that, you are redirected to your repository and can find all the connection instructions.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1610617842885/k-3yKUxij.png)

In the connection instructions, select mvn and copy the CodeArtifact authorization token line.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1610620155939/QZ-S6L93o.png)

The main objective of this article is to push an artifact during the maven deployment process. In our case, we use Gitlab for CI/CD and we can configure the deployment in the gitlab-ci.yaml described bellow.

In the step of the gitlab-ci.yaml, we use a docker image, Maven with AWS and ECS CLI tools  https://hub.docker.com/r/softinstigate/maven-aws/ 

As the token of aws codeartifact change every 3hours, we create a "before_script" to get back the token and export it for the settings.xml file that will be described in a second step. For this export, paste the line that was copied in the connection instructions of AWS.

After that, we use the command "mvn deploy" in the script section to deploy the artifact on AWS.

```
variables:
  MAVEN_CLI_OPTS_DEV: "-s .m2/settings.xml --batch-mode"
  DOMAIN_OWNER: xxxxxx
  APP_DOMAIN: myappdomain
  APP_PACKAGE: myapppackage

deploy-maven-artifact-aws:
  image: 
    name: softinstigate/maven-aws
  stage: deploymavenaws
  before_script:
    - export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token --domain $APP_DOMAIN --domain-owner $DOMAIN_OWNER --query authorizationToken --output text)
  script:
    - mvn $MAVEN_CLI_OPTS_DEV deploy

``` 

As you can see in the gitlab-ci.yaml, the mvn deploy command had a parameter with a settings file. If you dont already have this settings.xml in your project, you can create it.
To fill this file with the correct content, you should go back to your AWS CodeArtifact connections instructions. This time, you have to copy the step 2 and 3.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1610620119765/ibvx-u0dV.png)

Here is what your file should look like 
```
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd"
    xmlns="http://maven.apache.org/SETTINGS/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <servers>
    <server>
      <id>${env.AWSID}</id>
      <username>${env.USERNAME}</username>
      <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
    </server>
  </servers>
  <profiles>
    <profile>
      <id>${env.AWSID}</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>${env.AWSID}</id>
            <url>${env.URL_ENV_AWS}/maven/${env.APP_PACKAGE}/</url>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
``` 
A good practise would be to save your credentials informations as variables, for example in a secure vault to avoid having this value hardcoded in your project.

Additionally, for Maven, you need to add the following lines at the end of your pom.xml file. The fields <id> and <url> are the same values that you copy in the settings.xml file. The name represent the repository name.
```
...
    <distributionManagement>
         <repository>
             <id>cisel--${env.APP_PACKAGE}</id>
             <name>${env.APP_NAME}</name>
             <url>${env.URL_ENV_AWS}/maven/${env.APP_PACKAGE}/</url>
         </repository>
    </distributionManagement>

</project>
``` 

With this method, you should have a new artifact on AWS CodeArtifact every time you launch your gitlab CI/CD !

Tips : if you launch your CI/CD with the same version in your pom.xml, there will be a 409 conflict issue on aws because you artifact already exist.
You can delete artifact with the same id (for example on a dev CI/CD branch) by adding the following lines in your before_script : 

```
- apt-get install -y libxml2 libxml2-utils
- RELEASE_VERSION=$(xmllint --xpath '/*[local-name()="project"]/*[local-name()="version"]/text()' pom.xml)
- aws codeartifact delete-package-versions  --domain $APP_DOMAIN --repository $APP_REPO --format maven --package $APP_PACKAGE --namespace $APP_NS --versions $RELEASE_VERSION || true
``` 



Feel free to contact us directly if you have any question at cloud@cisel.ch
[https://www.cisel.ch](Link) 

## Team

| <a href="https://www.cisel.ch" target="_blank">**CISEL**</a> | <a href="https://www.cisel.ch" target="_blank">**CISEL**</a> | <a href="https://www.cisel.ch" target="_blank">**CISEL**</a> |
| :---: |:---:| :---:|
| FCA | SME | QBA |

---

## Contact

Reach us at one of the following places!

- Website at <a href="https://www.cisel.ch" target="_blank">`cisel.ch`</a>
- LinkedIn at <a href="https://www.linkedin.com/company/cisel-informatique-sa/" target="_blank">`CISEL Informatique SA`</a>

---

## License
- We used the readme.md example from  <a href="https://gist.github.com/fvcproductions/1bfc2d4aecb01a834b46" target="_blank">fvcproductions</a> pour créer ce template.
- Copyright 2020 © <a href="https://www.cisel.ch" target="_blank">CISEL</a>.
