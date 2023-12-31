# Acerca del proyecto
Se tiene que validar las peticiones:

1. POST Autorización
2. POST Procesar pago
3. GET Consultar estado pago

## Solución

# 1. Para validar el post de autorización de pago se necesita verificar mediante un pre-request que

_Para el body de esta request se tiene datos que podrian cambiar, por lo tanto, se considera generar números aleatorios para cada "clientID" y "name", enviarlos a la lista de variables de la colección antes de ejecutar la prueba y reutilizarlos en cada request._
## PRE-REQUEST:
- [x] Create a random Client ID
- [x] Create a random FirstName and LastName
- [x] Replace in Collection Variables

## TEST:
### El codigo de status es correcto:
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

### Que el token de autorización se está generando:
pm.test("Authorization token exist", function () {
    pm.expect(pm.response.json().authorizationToken).to.exist;
});

### Que el mensaje de éxito es el correcto:
pm.test("Success message is correct", function () {
    pm.expect(pm.response.json().message).to.be.equal("La autorización se ha concedido exitosamente.");
});

## STORING:
### Y se tiene que almacenar este authorizationToken en una variable de la colección para usarlo cuando se tenga que realizar un pago:
pm.collectionVariables.set("strAuthToken", (pm.response.json().authorizationToken));


# 2. Para validar el procesamiento del pago (/payment)

Reutilizar la variable authorizationToken (strAuthToken) almacenada en las variables de la colección

## PRE-REQUEST:
- [x] Capturar el Authorization Token 
const at = (pm.collectionVariables.get('strAuthToken'));

- [x] Crear un random Payment Amount 
const paymAmo = pm.variables.replaceIn("{{$randomPrice}}");

- [x] Reemplazar en las variables de colección
pm.collectionVariables.set("strPaymentAmount",paymAmo);

## TEST:
### El codigo de status es correcto: 
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

### El estado de éxito del pago 
pm.test("Success for payment", function () {
    pm.expect(pm.response.json().paymentStatus).to.be.equal("success");
});

### El metodo de pago esperado es correcto
pm.test("Payment method", function () {
    pm.expect(pm.response.json().paymentMethod).to.be.equal("Credit Card");
});

## STORING
### Se almacena el transactionID en una variable de la colección para usarlo cuando queremos consultar los pagos
pm.collectionVariables.set("strTransactionID", (pm.response.json().transactionID));


# 3. Para validar la consulta del estado del pago (/payment/status)

El transactionID se extrae de la colección de variables almaceandas en el request anterior.

## TEST
### El codigo de status es correcto: 
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

### Verificar el mensaje de estado del pago:
pm.test("Success for payment", function () {
    pm.expect(pm.response.json().paymentStatus).to.be.equal("success");
});

### Verificar el metodo de pago:
pm.test("Payment method", function () {
    pm.expect(pm.response.json().transactionDetails.paymentMethod).to.be.equal("Credit Card");
});


# 4. Total de ordenes (/orders/all)

## TEST:
### El codigo de status es correcto:  
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

### El response con los datos de todas las ordenes existe: 
pm.test("Response body exist", function () {
    pm.expect(pm.response.body).to.exist;
});



# Ejecución via Newman
=======================

## Instalar newman y reporter desde npm:

1. npm install -g newman
2. npm install -g newman-reporter-htmlextra

## Ejecutar una colección

3. newman run "D:\Projects\deuapitesting\DEUNA - API Testing.postman_collection.json"

## Ejecutar con reporter en html:

3. newman run "D:\Projects\deuapitesting\DEUNA - API Testing.postman_collection.json" -r htmlextra