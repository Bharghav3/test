apiVersion: observability.openshift.io/v1
kind: ClusterLogForwarder
metadata:
  name: instance
  namespace: openshift-logging
spec:
  filters:
    - name: my-prune
      prune:
        notIn:
          - ."@timestamp"
          - .kubernetes.container_name
          - .kubernetes.labels.name
          - .kubernetes.namespace_name
          - .kubernetes.pod_name
          - .level
          - .log_type
          - .message
      type: prune
    - name: my-multiline
      type: detectMultilineException
    - name: parse-json
      type: parse
    - name: my-mule-app
      openshiftLabels:
        type: MuleApplication
      type: openshiftLabels
  managementState: Managed
  outputs:
    - elasticsearch:
        authentication:
          password:
            key: password
            secretName: es-auth
          username:
            key: username
            secretName: es-auth
        index: runtime_fabric_ocp_non_prod-0000
        url: 'http://es_host:9200'
        version: 8
      name: es-output-by-label
      type: elasticsearch
  pipelines:
    - filterRefs:
        - parse-json
        - my-multiline
        - my-mule-app
        - my-prune
      inputRefs:
        - application
      name: my-mulesoft-app
      outputRefs:
        - es-output-by-label
  serviceAccount:
    name: external-log-forwarder
