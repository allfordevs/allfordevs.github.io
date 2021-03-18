---
layout: post
title:  "Discriminated Unions - um estilo diferente para retornar valores de funções em C# usando a biblioteca OneOf."
tags: [donet-core, .NET, c#, oneof]
categories: [CSharp, .NET, programação]
pin: false
date: 2021-03-18 17:00:00 -0300
---

## Aprenda um caminho diferente para retornar valores em C# usando a biblioteca OneOf.



Suponho que você em algum momento desejou obter tipos e valores diferentes retornados pelo mesmo método, inclusive acredito fortemente que você já usou **Tuplas** (ou ainda faz uso disso) como uma solução alternativa para alcançar um retorno satisfatório, não é mesmo?



No paradigma funcional esse contexto costuma ser resolvido usando uma forma conhecida como [**Discriminated unions** ](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions) onde é possível retornar de uma mesma função tipos com valores ou somente valores diferente em determinadas situações e comportamento.



Em C# é possível ter um comportamento semelhante ao descrito acima fazendo o uso de uma biblioteca chamada [**OneOf**](https://www.nuget.org/packages/OneOf/) criada e mantida por [**Harry McIntyre**](https://github.com/mcintyre321/OneOf). Essa biblioteca fornece ao C# o estilo usado em F# para tratar Unions usando tipos personalizados em seu retorno permitindo que métodos devolvam tipos com valores unicos porém diferentes conforme o fluxo tratado na função, vejamos o exemplo abaixo:

### Caminho tradicional:
O objetivo do código abaixo é validar o parâmetro recebido, se for inválido será lançado uma exception, o contrário retornará uma entidade que representa um comprovante de pagamento. A Controller por sua vez validará o retorno e orquestrará o response conforme o resultado obtido através da função **MakePayment**.

>Service
``` cs
// Services
public ProofPayment MakePayment(OrderPurchase purchase)
{
  if (purchase is null)
    return default;
  if (purchase.OrderId <= 0)
    throw new OrderPurchaseException("The orderIdinvalid.");
  if (purchase.Total <= 0)
    throw new OrderPurchaseException("The total orcannot be zero.");
  if (
      purchase.EPaymentType != PaymentType.Cash       &&
      purchase.EPaymentType != PaymentType.CreditCard &&
      purchase.EPaymentType != PaymentType.DebitCard
     ) { throw new OrderPurchaseException("The type payment is invalid.");
  return new ProofPayment
  {
      Id = Guid.NewGuid(),
      Identifier = new Random(10).Next(100),
      Type = purchase.GetLabelPaymentType(),
      DateBilling = DateTime.Now,
      Value = purchase.Total
  };
}
```
>Controller
```cs
//Controller - Http
public IActionResult Post(OrderPurchase body)
{
  try
  {
    var result = new TraditionalWay().MakePayment(body);
    if (result is null)
        return BadRequest();
    return Ok(result);
  }
  catch (OrderPurchaseException ex)
  {
    return BadRequest(error: ex.Message);
  }
  catch(Exception ex)
  {
    // Any exception...
    return StatusCode(500, ex.Message);
  }
}
```

### Usando a biblioteca OneOf o mesmo comportamento acima ficaria assim:


``` cs
public OneOf<ProofPayment,OrderPurchaseInvalid> MakePayment(OrderPurchase purchase)
{
  if (purchase is null)
    return new OrderPurchaseInvalid("Object cannot be null");
  if (purchase.OrderId <= 0)
    return new OrderPurchaseInvalid("The orderId is invalid.");
  if (purchase.Total <= 0) ]
    return new OrderPurchaseInvalid("The total order cannot be zero.");

  if (
    purchase.EPaymentType != PaymentType.Cash       &&
    purchase.EPaymentType != PaymentType.CreditCard &&
    purchase.EPaymentType != PaymentType.DebitCard
    )
  { return new OrderPurchaseInvalid("The type payment is invalid."); }

  return new ProofPayment
  {
      Id = Guid.NewGuid(),
      Identifier = new Random(10).Next(100),
      Type =  purchase.GetLabelPaymentType(),
      DateBilling = DateTime.Now,
      Value = purchase.Total
  };
}
```

``` cs
public IActionResult UsingOneOf(OrderPurchase body)
=>  new AnExoticForm().MakePayment(body)
    .Match<IActionResult>
    (
        proofPayment => Ok(proofPayment),
        orderPaymentInvalid => BadRequest(orderPaymentInvalid.InvalidMessage)
    );
```

## Benefícios:
- Você pode evitar o uso de exceptions como tratamento de fluxo, basta apenas criar retornos personalizados para cada tipo de fluxo ou implementação de notification pattern.
- Simples de compreender o tratamento com base no retorno.
- Retorno fortemente tipado.


## Observações:
- Não indicaria a aplicação total desse biblioteca no teu dominio.
- Implementaria entre a camada de applicação e nas API's.
- Cuidado! C# não é 100% funcional.


Espero que tenham gostado, o código fonte esta no meu [repositorio](https://github.com/BrunoBMelo/dotnet-oneof-sample/).






