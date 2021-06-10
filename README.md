# Integración Hitespay

## Creación de link de pago

>Todas las peticiones de api, estan protegidas por un bearer token único asignado a cada >comercio.
>En el proceso de integración te entregaremos un token de prueba para que puedas trabajar sin problemas.

Para el crear el boton de pago, es necesario realizar una petición post a nuestro endpoint:

```php
$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://api.clickypaga.cl/api/orders');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, 1);
$post = array(
    'order_amount' => '10000',
    'merchant_order_id' => '1234',
    'order_description' => 'Orden de Prueba',
    'notification_url' => 'https://tu_url_de_notificacion',
    'return_url' => 'https://tu_url_de_retorno'
);
curl_setopt($ch, CURLOPT_POSTFIELDS, $post);

$headers = array();
$headers[] = 'Authorization: Bearer <tu bearer token>';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);
```

Los campos requeridos se describen a continuación:
* order_amount: monto total de la orden
* merchant_order_id: el identificador único de la orden en tu sistema
* order_description: descriptor de la orden
* notification_url: url donde enviaremos la notificación de pago de la orden
* return_url: url a la que retornaremos al usuario una vez realizado el pago

La respuesta del servicio será un objeto JSON como se describe a continuación:
```javascript
{
  "id": "d290f1ee-6c54-4b01-90e6-d701748f0851",
  "status": "created",
  "payment_url": "https://api.clickypaga.cl/orders/payment/e9dfd47d-451f-4fdb-a49b-b65c7e0eba1e"
}
```


## Retorno de pago

Una vez que el usuario finaliza el pago dentro del portal de HITES, se notificará su estado al comercio asociado, directamente a la url de notificación indicada en el campo notification_url.

En caso de pago exitoso, se enviará una petición POST (form-data) con los siguientes campos

- status : Estado de la orden (paid)
- order_id: identificador interno de la orden en formato UUID
- merchant_order_id: identificador único de la orden en tu sistema
- date: fecha de pago 
- time: hora de pago
- authCode: código de autorización de la orden dentro de HITES
- cuotas: total de cuotas
- totalAmount: Total pagado
- message: Mensaje retorno de pago hites

En caso de error en el proceso de pago, se retornara una petición POST a la misma URL con los siguientes campos:
- status: 'fail'
- message: motivo del error

## Paso producción
Para pasar a producción tu ambiente, te entregaremos un token válido, y te solicitaremos, por motivos de seguridad, la(s) IP(s) desde donde harás las peticiones a nuestra API, así también recomendamos que la URL de notificación sea accesible solo por nosotros, para eso te enviaremos la IP de nuestro sistema.



Para más detalles, revisa nuestro [swagger](https://app.swaggerhub.com/apis-docs/ccontrerasleiva/clickypaga/1.0.0)
