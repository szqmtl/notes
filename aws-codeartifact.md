Because of growth of my projects, there are some demands for an independent package management system. This doc is to explore the functionalities of AWS CodeArtifact, and summarize the steps.

# Environment preparation
there are three setups involved in the research: AWS user, AWS CodeArtifact, and sample projects with AWS CLI installed

## AWS user
Setup user Access Key in IAM

Setup AWS CLI profile in local machine. we will use LocalProfile as an example later

## AWS CodeArtifact
After preparing the aws local profile

1. Create domain: 
```
# TestDomain is the domain name to be created
aws --profile LocalProfile codeartifact create-domain --domain TestDomain
```
2. Create repository:
```
# 123456789012 is the user ID which can get from the domain page
# TestRepo is the repository to be created
aws --profile LocalProfile codeartifact create-repository --domain TestDomain --domain-owner 123456789012 --repository TestRepo
```
3. Get authorization token to be used later:
```
aws --profile LocalProfile codeartifact get-authorization-token --domain TestDomain --query authorizationToken --output text
```
## Sample projects
library project: jarLib

main project: jarMain

# Project setup
In library project, putting below code snippets in the file build.gradle

```
plugins {
    id 'java-library'
    id 'maven-publish'
}

java {
    sourceCompatibility = 11
    targetCompatibility = 11
}

tasks.named('jar') {
    manifest {
        attributes('Implementation-Title': project.name,
                'Implementation-Version': project.version)
    }
}


publishing {
    publications {
        mavenJava(MavenPublication) {
            groupId = project.group
            artifactId = project.name
            version = project.version
            from components.java
        }
    }
    repositories {
        maven {
            url 'https://TestDomain-123456789012.d.codeartifact.us-east-1.amazonaws.com/maven/TestRepo/'
            credentials {
                username "aws"
                password "Authorization Token"
            }
        }
    }
}

```

Running 
```
gradle publish
```
to upload the jar file to the repository


In the main project build.gradle file, putting this snippet:

```
repositories {
    maven {
        url "https://TestDomain-123456789012.d.codeartifact.us-east-1.amazonaws.com/maven/TestRepo/"
        credentials {
            username "aws"
            password "Authorization Token"
        }
    }
}

dependencies {
    implementation 'code.example:jarlib:1.0-SNAPSHOT'
}
```

# Conclusion
After the POC, CodeArtifact would be one of the options when we need to use library management system. However, there will be more detail needed to be filled in when we actually set up the building pipeline.

 

# Reference
[AWS CodeArtifact User Guide](https://docs.aws.amazon.com/codeartifact/latest/ug/welcome.html)
