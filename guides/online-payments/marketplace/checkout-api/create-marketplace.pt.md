# Integra o marketplace via API

> WARNING
>
> Pré-requisitos
>
> * Você deve ter ----[mla, mlm, mlc, mco, mlu, mpe]---- [Checkout API](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/pt/guides/online-payments/checkout-api/introduction) ------------ ----[mlb]---- [Checkout Transparente](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/pt/guides/online-payments/checkout-api/introduction) ------------ implementado.

Para começar, siga os passos abaixo:

1. Crie ou configure sua aplicação.
2. Vincule sua aplicação à conta de seus vendedores.
3. Crie as credenciais para operar.
4. Renove as credenciais.
5. Integre a API para receber pagamentos.
6. Teste seu Marketplace.


## Crie ou configure sua aplicação.

Primeiramente, você deve ter criado [sua aplicação](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/panel/applications/create-app) com um nome de identificação único.

é preciso **configurar um Redirect URL para sua aplicação**. Para isso, acesse [Suas Aplicações](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/panel), clique no menu de opções da sua aplicação e selecione Editar.

No campo Redirect URL, adicione o endereço para onde você deseja encaminhar os vendedores após tê-los vinculado corretamente. Lembre-se que os códigos de autorização para a criação das credenciais serão enviados para o endereço cadastrado.

Finalmente, você deve obter a ID do sua aplicação em [Suas Integrações](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/panel).

## Vincule sua aplicação à conta de seus vendedores.

Para operar em nome dos seus vendedores através de suas contas do Mercado Pago, primeiramente, você deve solicitar uma autorização. É possível gerenciar mais de uma conta do Mercado Pago simultaneamente em sua integração através do OAuth, um um protocolo de autenticação que permite ao vendedor acessar sua conta do Mercado Pago, realizar a autenticação  e habilitar seu aplicativo para funcionar utilizando seu nome.

Para isso, é preciso incluir em sua aplicação uma URL que encaminhe o vendedor para o site de autorização.  

Compartilhamos a URL base que você deve utilizar e o detalhe dos parâmetros com os quais deverá completá-la.

```url
https://auth.mercadopago[FAKER][URL][DOMAIN]/authorization?client_id=APP_ID&response_type=code&platform_id=mp&state=RANDOM_ID&redirect_uri=https://www.redirect-url.com


```

| Parâmetro | Dado a completar |
| ----------------- | ----------------- |
| `client_id` | Substitua o valor `APP_ID` com a ID da sua aplicação. |
| `state` | Identifique de quem é o código que você vai receber. Para isso, substitua o valor `RANDOM_ID` por um identificador que seja único para cada tentativa e que não inclua informações sensíveis. |
| `redirect_uri` | Adicione a URL que informou no campo Redirect URL ao configurar sua aplicação. | 

Ao acessar essa URL, o vendedor será encaminhado para o Mercado Pago, onde deverá fazer o login com sua conta e autorizar o vínculo com sua aplicação.

![FlujoOAuth-pt](/images/oauth/oauth-pt-v2.png)

Quando o vendedor autorizar o vínculo da aplicação à sua conta do Mercado Pago, você receberá em seu servidor o código de autorização na Redirect URL especificada, conforme mostrado abaixo.  

```url
https://www.redirect-url.com?code=CODE&state=RANDOM_ID
```

> Lembre-se que o valor `code` tem um período de validade de 10 minutos.

> SERVER_SIDE
>
> h2
>
> Crie as credenciais para operar

Para criar as credenciais necessárias para sua aplicação operar em nome de um vendedor, você terá que enviar o `CODE` obtido no passo anterior através da API de OAuth.

Os parâmetros que você deve incluir são:

| Parâmetro | Dado a preencher |
| ----------------- | ----------------- |
| `client_secret` | Chave privada para ser utilizada em alguns plugins para gerar pagamentos. Você pode obtê-lo em [Suas Credenciais]([FAKER][CREDENTIALS][URL]). |
| `client_id` | ID único que identifica sua integração. Você pode obtê-lo em [Suas Credenciais]([FAKER][CREDENTIALS][URL]). |
| `grant_type` | Diz respeito ao tipo de operação a ser realizada para obter as credenciais. Este parâmetro é fixo e seu valor é `authorization_code`. |
| `code` | O código de autorização ou `CODE` que você obtém em seu servidor ao realizar a vinculação. Será similar a este valor: `TG-60357f5d0cd06d000740646d-643464554`. | 
| `redirect_uri` | É a URL que você configurou no campo Redirect URL em sua aplicação.|

```curl
curl -X POST \
     -H 'accept: application/json' \
     -H 'content-type: application/x-www-form-urlencoded' \
     'https://api.mercadopago.com/oauth/token' \
     -d 'client_secret=CLIENT_SECRET' \
     -d 'client_id=CLIENT_ID' \
     -d 'grant_type=authorization_code' \
     -d 'code=CODE' \
     -d 'redirect_uri=REDIRECT_URI'
```

Na resposta você vai obter o `access_token` do vendedor vinculado. 

Você também vai receber o `refresh_token`, que servirá para renovar as credenciais dos seus vendedores. 

Além disso, você vai receber a `public_key` do vendedor, que é a credencial ou chave pública necessária para identificar a conta em seu frontend. 

```json
{
"access_token":"APP_USR-4934588586838432-XXXXXXXX-241983636",
"token_type": "bearer",
"expires_in": 15552000,
"scope": "offline_access read write",
"user_id": 241983636,
"refresh_token": "TG-XXXXXXXX-241983636",
"public_key": "APP_USR-d0a26210-XXXXXXXX-479f0400869e",
"live_mode": true
}
```

> WARNING 
> 
> Importante
> 
> Lembre-se que você vai utilizar informações sensíveis dos seus vendedores. Garanta que serão armazenadas de maneira segura, não incorpore nas suas URL de vinculação e gerencie somente do seu servidor.

Pronto! Você já vinculou a conta do vendedor ao sua aplicação através de OAuth. 

> Lembre que estes passos devem ser repetidos para cada conta que quiser vincular. 

## Renove as credenciais

**As informações que você recebe dos seus vendedores têm validade durante 180 dias**. Passado esse tempo, você deverá solicitar novamente a autorização ao vendedor.
Para evitar isso, renove os dados antes desse período para garantir que estejam sempre vigentes. 

Para renovar, você deverá realizar a seguinte chamada na API de OAuth:

```curl
curl -X POST \
     -H 'accept: application/json' \
     -H 'content-type: application/x-www-form-urlencoded' \
     'https://api.mercadopago.com/oauth/token' \
     -d 'client_secret=CLIENT_SECRET' \
     -d 'client_id=CLIENT_ID' \
     -d 'grant_type=refresh_token' \
     -d 'refresh_token=USER_REFRESH_TOKEN'
```

| Parâmetro | Descrição |
| ----------------- | ----------------- |
| `client_secret` | Utilize seu `client_secret`. |
| `client_id` | Utilize seu `client_id`. |
| `grant_type` | Inclua `refresh_token`, que não sofre alterações. |
| `refresh_token` | Valor que você recebeu junto com os dados do vendedor. | 

Você receberá a seguinte resposta:

```json
{
    "access_token": "APP_USR-4934588586838432-XXXXXXXX-241983636",
    "token_type": "bearer",
    "expires_in": 15552000,
    "scope": "offline_access read write",
    "refresh_token": "TG-XXXXXXXXXXXX-241983636"
}
```


### Desvincule uma conta

Para desvincular o token associado à sua conta, você deve fazer isso no [portal do Mercado Pago](https://www.mercadopago[FAKER][URL][DOMAIN]/account/security/applications/connections) em **Seu perfil> Segurança> Aplicativos conectados**.



## Integre a API para receber pagamentos

Para receber pagamentos em nome de seus vendedores, você deve integrar a [API](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/pt/guides/online-payments/checkout-api/introduction), gerando os pagamentos com o Access Token que você obteve vinculando cada vendedor ao seu aplicativo.

Se deseja cobrar uma taxa de comissão por cada pagamento processado pela sua aplicação em nome do seu usuário, simplesmente adicione esse valor no parâmetro  `application_fee` ao criar pagamento:

[[[
```curl
curl -X POST \
        -H 'accept: application/json' \
        -H 'content-type: application/json' \
        -H 'Authorization: Bearer USER_AT' \
        https://api.mercadopago.com/v1/payments \
        -d '{
                "transaction_amount": 100,
                "token": "ff8080814c11e237014c1ff593b57b4d",
                "description": "Title of what you are paying for",
                "installments": 1,
                "payer": {
                        "id": "12345678"
                },
                "payment_method_id": "visa",
                "application_fee": 2.56
        }'
```
```php
<?php  

  require ('mercadopago.php');
  MercadoPago\SDK::configure(['ACCESS_TOKEN' => 'ENV_ACCESS_TOKEN']);

  $payment = new MercadoPago\Payment();

  $payment->transaction_amount = 100;
  $payment->token = "ff8080814c11e237014c1ff593b57b4d";
  $payment->description = "Title of what you are paying for";
  $payment->installments = 1;
  $payment->payment_method_id = "visa";
  $payment->payer = array(
    "email" => "test_user_19653727@testuser.com"
  );

  $payment->save();

?>
```
```java

import com.mercadopago.*;
MercadoPago.SDK.configure("ENV_ACCESS_TOKEN");

Payment payment = new Payment();

payment.setTransactionAmount(100f)
      .setToken('ff8080814c11e237014c1ff593b57b4d')
      .setDescription('Title of what you are paying for')
      .setInstallments(1)
      .setPaymentMethodId("visa")
      .setPayer(new Payer("test_user_19653727@testuser.com"));

payment.save();

```
```node

var mercadopago = require('mercadopago');
mercadopago.configurations.setAccessToken(config.access_token);

var payment_data = {
  transaction_amount: 100,
  token: 'ff8080814c11e237014c1ff593b57b4d'
  description: 'Title of what you are paying for',
  installments: 1,
  payment_method_id: 'visa',
  payer: {
    email: 'test_user_3931694@testuser.com'
  }
};

mercadopago.payment.create(payment_data).then(function (data) {
  // Do Stuff...
}).catch(function (error) {
  // Do Stuff...
});

```
```ruby

require 'mercadopago'
sdk = Mercadopago::SDK.new('ENV_ACCESS_TOKEN')

payment_data = { 
  transaction_amount: 100,
  token: 'ff8080814c11e237014c1ff593b57b4d',
  description: 'Title of what you are paying for',
  installments: 1,
  payment_method_id: 'visa',
  payer: {
    email: 'test_user_19653727@testuser.com'
  }
}
````
]]]

> NOTE
> 
> Nota
> 
> > Lembre-se que, a cada vez que você renovar as credenciais, o `refresh_token` também mudará, por isso, você deverá armazená-lo novamente.
>
>  Caso haja algum erro na hora de renovar as credenciais, lembre-se que você pode consultar a [referência de códigos de erro](https://developers.mercadolivre.com.br/pt_br/autenticacao-e-autorizacao#Referencia-de-codigos-de-erro).


## Teste seu Marketplace

É possível testar seu marketplace usando as credenciais de Sandbox da sua conta para associar os vendedores e fazer as cobranças/cancelamentos entre outros.

Você pode usar os cartões de teste fornecidos pelo Mercado Pago e os diferentes prefixos para manipular as mensagens de resposta.

[Teste sua integração](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/pt/guides/online-payments/checkout-api/testing)

## Configure as notificações

Você pode receber notificações sempre que um vendedor se vincular ou desvincular da sua aplicação. Para configurá-las, siga os passos abaixo.

1. Acesse [Suas aplicações](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/panel) e selecione a aplicação que você utiliza para o fluxo de OAuth.

2. Vá para a aba "Notificações Webhooks". Já dentro da seção, vá para o campo "Modo Produção" e adicione a URL onde quer receber as notificações. Se quiser, você pode clicar no botão "Testar" para conferir que a URL escolhida recebe corretamente as Notificações Webhooks.

3. Depois, no campo "Eventos", selecione a opção "Vinculação de aplicações". Por último, clique em salvar. 

Pronto! A cada vez que um vendedor se vincular ou desvincular, você receberá uma notificação na URL escolhida.

Estes são alguns dos dados que você poderá encontrar dentro das notificações:

| Atributo | Valor ou tipo | Descrição |
| ----------------- | ----------------- | --------------- |
| `type` | `mp-connect` | Identifica a notificação do tipo vinculação de contas. |
| `action` | `application.authorized` | Informa que o vendedor se vinculou a aplicação. |
| `action` | `application.deauthorized` | Confirma que o vendedor se desvinculou do aplicação. |
| `data.id`| `string`| ID do vendedor vinculado ao aplicação. |

Para saber mais, acesse [Notificações Webhooks](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/pt/guides/notifications/webhooks).

## Devoluções e cancelamentos

As devoluções e cancelamentos poderão ser efetuados tanto pelo marketplace como pelo vendedor, através da API ou a partir da conta no Mercado Pago.

Caso a devolução se realize no marketplace, deve-se utilizar as credenciais obtidas para cobrar em nome do vendedor.

Os cancelamentos somente poderão ser efetuados utilizando a API de cancelamentos.

Para mais informações, consulte a seção de [devoluções e cancelamentos](https://www.mercadopago[FAKER][URL][DOMAIN]/developers/pt/guides/manage-account/account/cancellations-and-refunds).