==============
 Services
==============

`Docs <https://kubernetes.io/docs/concepts/workloads/services/>`_

Description
--------------

A Service is an abstraction which defines a logical set of Pods and a policy by which to access them. The set of Pods targeted by a Service is usually determined by a selector.

Example
--------------

.. code-block:: yaml

  kind: Service
  apiVersion: v1
  metadata:
    name: service-example
  spec:
    ports:
      # Accept traffic sent to port 80
      - name: http
        port: 80           # this is the container port
        nodePort: 30080
    selector:
      app: nginx
    type: NodePort
