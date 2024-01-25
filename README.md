# Trivy-Archerysec-Integration

## Install Trivy for Jenkins agent:

  ```bash
  curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/bin v0.48.1
  ```
## Add optional parameter in Jenkins parameters:

  ```yaml
  booleanParam(name: 'DO_TRIVY_SCAN',
        defaultValue: false,
        description: 'If selected, Images will be scanned for vulnerabilities and a security report will be generated and uploaded to archerysec portal')
  ```

## Do Trivy scan and upload report result if required:

```
if ( "$DO_TRIVY_SCAN" ); then
    echo "Scanning ${NAME} using Trivy”

    # create Project in archerysec
    project_id=$(curl -s -X POST -H "x-api-key: ${ARCHERYSEC_TOKEN}" -F project_name="${WIT_DOCKER_SERVICE_NAME}" -F project_disc="Scanning ${WIT_DOCKER_SERVICE_NAME} using trivy, created from jenkins pipeline" https://archerysec.uat.superapp.m-pesa.com/api/v1/project-create/ | jq -r '.project_id')

    # Scan Image and generate the report
		trivy image -f json -o ${WIT_DOCKER_SERVICE_NAME}_report_Version_${BUILD_VERSION}.json ${NAME} > /dev/null

    # Upload report to archerysec
	  curl -i -L -X POST   -H "x-api-key: ${ARCHERYSEC_TOKEN}"   -F "filename=@${WIT_DOCKER_SERVICE_NAME}_report_Version_${BUILD_VERSION}.json"   -F "file_type=JSON"   -F "scan_url=${BUILD_VERSION}"   -F "scanner=trivy_scan"   -F "project_id=${project_id}"   https://archerysec.uat.superapp.m-pesa.com/api/v1/uploadscan/
fi
```
