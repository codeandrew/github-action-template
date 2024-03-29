# github-action-template

## Github Action Templates

For Merge Request
```yaml
name: Merge Request Complete

on:
  push:
    branches:
      - main

jobs:
  detect_mr_complete:
    runs-on: ubuntu-latest
    if: "github.event_name == 'push' && github.event.pull_request.merged == true && github.event.pull_request.base.ref == 'main'"
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Run a script when merge request is completed
        run: |
          echo "Merge request is completed on main branch!"
```


Automated Release Tag
```yaml
name: Create Release Tag

on:
  pull_request:
    types: [closed]
  push:
    branches:
      - main

env:
  SEMVER: ${{ env.SEMVER }}

jobs:
  create-release-tag:
    runs-on: ubuntu-latest
    if: github.event.pull_request.merged == true && github.event.pull_request.base.ref == 'main'

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Determine Version
        id: version
        run: echo "::set-output name=version::$(./determine_version.sh $SEMVER)"

      - name: Create Tag
        uses: anothrNick/github-tag-action@v1.14.2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ steps.version.outputs.version }}
          message: "Release ${{ steps.version.outputs.version }}"
          draft: false
          prerelease: false

      - name: Update SEMVER
        if: success()
        env:
          SEMVER: ${{ steps.version.outputs.version }}
        run: echo "SEMVER=${{ env.SEMVER }}" >> $GITHUB_ENV

```
script
```bash
#!/bin/bash

set -e

if [[ $# -eq 0 ]] ; then
    echo "You must specify the current version in x.y format"
    exit 1
fi

VERSION=$1
LAST_CHAR=$(echo ${VERSION: -1})

if [[ $LAST_CHAR == "." ]] ; then
    VERSION="${VERSION}0"
fi

TAG_COUNT=$(git tag --list "${VERSION}.*" | wc -l | awk '{print $1}')

NEW_VERSION="${VERSION}.${TAG_COUNT}"
echo "${NEW_VERSION}"

```


## Docker CheatSheet

```
---
version: "3"
services:
  web:
    image: httpd
    ports:
      - "8080:80"
    volumes:
      - mydata:/data

volumes:
  mydata
```

For binding the volumes to a path

## Security 

For making the pipeline secure

Tools:
```
Clair - Vulnerability Static Analysis for Containers
Anchore - Open-source project for deep analysis of docker images
Dagda - A tool to perform static analysis of known vulnerabilities, trojans, viruses, malware & other malicious threats in docker images/containers and to monitor the docker daemon and running docker containers for detecting anomalous activities
Falco - Falco, the cloud-native runtime security project, is the de facto Kubernetes threat detection engine
Harbor - Harbor is an open source registry that secures artifacts with policies and role-based access control, ensures images are scanned and free from vulnerabilities, and signs images as trusted.
Trivy - Trivy is a simple and comprehensive vulnerability/misconfiguration scanner for containers and other artifacts.
```

### SONAR QUBE


```yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```

SonarQube Github Actions
Usage
```
on:
  # Trigger analysis when pushing in master or pull requests, and when creating
  # a pull request. 
  push:
    branches:
      - master
  pull_request:
      types: [opened, synchronize, reopened]

name: Main Workflow
jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        # Disabling shallow clone is recommended for improving relevancy of reporting
        fetch-depth: 0
    - name: SonarQube Scan
      uses: sonarsource/sonarqube-scan-action@master
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

Reference 
- https://docs.sonarqube.org/latest/setup/install-server/
- https://github.com/marketplace/actions/official-sonarqube-scan

### REFERENCES
- https://owasp.org/www-project-devsecops-guideline/latest/02f-Container-Vulnerability-Scanning
- https://opensource.com/article/18/8/tools-container-security

test
