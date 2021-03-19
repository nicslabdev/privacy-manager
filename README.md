# Privacy Manager system for IoT and Edge Computing
A privacy system designed by @nicslabdev and developed by @roxax19 as part of his Degree Final Project.

# How to configure

## Step 1
You have to configure your own kubernetes cluster. You need to have a storage class, an ingress controller, and a load balancer.

You can easily get this with microk8s, and its addons. If you are using microk8s, you have to enable the following addons: dns, dashboard, ingress, storage, metallb

## Step 2
You have to add the following lines in your ingress controller. This will allow you to expose tcp services like mosquitto in your Ingress.

```
#This allow us to pass tcp traffic through the Ingress
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-ingress-tcp-microk8s-conf
  namespace: ingress
data:
  8083: "default/mosquitto-broker:1883"
---
#This goes inside the DaemonSet
kind: DaemonSet
metadata:
  ...
spec:
  selector:
    ...
  template:
    ...
    spec:
      containers:
      - name: nginx-ingress-microk8s
        ports:
        - containerPort: 80
        - containerPort: 443
        #This right here
        - name: prxy-tcp-8083
          containerPort: 8083
          hostPort: 1883
          protocol: TCP

```
If you are using microk8s installed through snap, you can use the comands in /extra/snpmk8 to unmount and mount the volume where the Ingress Controller is located. To update the info on the Ingress Controller, disable and enable again the Ingress addon.

## Step 3

You can change the certificate used in TLS by Ingress.
First, you have to create a secret with the cert in your kubernetes cluster. You can add it with this command:
```
kubectl create secret tls ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE}
```

Then, you need to add this line in your Ingress Controller:
```
#This goes inside the DaemonSet
kind: DaemonSet
metadata:
  ...
spec:
  ...
  template:
    ...
    spec:
      ...
      containers:
        ...
        args
        ...
        - --default-ssl-certificate=default/${CERT_NAME}
```


## Step 4
Kubernetes doesn't work with relative pathing (see [pathing rules](https://github.com/kubernetes/kubernetes/pull/20328/files)), so you will have to change the volume hostpath of the following files:

```
kubernetesFiles/connector/connector.yaml

kubernetesFiles/gen/gen.yaml

kubernetesFiles/nodejs/apiRest.yaml

kubernetesFiles/nodejs/auth.yaml

kubernetesFiles/nodejs/priv.yaml
```

You have to put the path to the folders inside mounts/ on this git.

## Step 5
For deploying the whole system, you have to execute (if you are working with microk8s) kubernetesFiles/execute_yaml.sh. If you aren't, you can copy the kubectl commands and execute in a terminal.

## Step 6
For it to work correctly, you have to enter the phpMyAdmin pod, and create the database and the tables with de following commands:

- Create database
```
CREATE DATABASE test
```
- Create "personas" table.
```
CREATE TABLE `test`.`personas` ( `id` INT NOT NULL AUTO_INCREMENT , `nombre` VARCHAR(255) NOT NULL , `edad` INT NOT NULL , `lat` FLOAT NOT NULL , `lon` FLOAT NOT NULL , `profesion` VARCHAR(255) NOT NULL , `sueldo` INT NOT NULL , `pulso` INT NOT NULL , `temperatura` FLOAT NOT NULL , `enfermedad` VARCHAR(255) NOT NULL , PRIMARY KEY (`id`))
```
- Insert into "personas"
```
INSERT INTO `personas` (`id`, `nombre`, `edad`, `lat`, `lon`, `profesion`, `sueldo`, `pulso`, `temperatura`, `enfermedad`) VALUES (NULL, 'Manuel Ruiz Ruiz', '21', '1234', '5678', 'Estudiante', '0', '60', '36', 'Sano'), (NULL, 'Alberto', '22', '123.567', '23.12344', 'Estudiante', '0', '59', '35.4', 'Sano'), (NULL, 'Juan', '34', '534.3521', '5363.3235', 'Ingeniero', '60000', '58', '38', 'Fiebre'), (NULL, 'Alvaro', '64', '984.1234', '734.565', 'Ingeniero', '70000', '66', '35.2', 'Sano'), (NULL, 'Miguel', '23', '546.234', '765.234', 'Médico', '90000', '67', '37.8', 'Fiebre'), (NULL, 'Ana', '3', '543.645', '123.456', 'Estudiante', '0', '61', '36.3', 'Sano'), (NULL, 'Elena', '43', '6.432534', '3.2345234', 'Ingeniero', '30000', '67', '35.8', 'Sano'), (NULL, 'Rocio', '70', '6546.3', '4563.2', 'Jubilado', '20000', '56', '35.4', 'Sano'), (NULL, 'Maria', '33', '867.6456', '342.123', 'Médico', '70000', '63', '36.5', 'Sano'), (NULL, 'Manuel Jesús', '23', '674.56', '4353.345', 'Médico', '50000', '69', '38.2', 'Fiebre') 
```
- Create "usuarios" table
```
CREATE TABLE `test`.`usuarios` ( `id` INT NOT NULL AUTO_INCREMENT , `email` VARCHAR(255) NOT NULL ,`contrasena` VARCHAR(255) NOT NULL , `clase` VARCHAR(255) NOT NULL , PRIMARY KEY (`id`))
```
- Insert into "usuarios"
```
INSERT INTO `usuarios` (`id`, `email`, `contrasena`, `clase`) VALUES (NULL, 'a', 'a', 'admin') 
```
