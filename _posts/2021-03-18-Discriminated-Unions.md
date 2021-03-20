---
layout: post
title:  "Discriminated Unions - um estilo diferente para retornar valores de funções em C# usando a biblioteca OneOf."
tags: [.NET, C#, OneOf]
categories: [.NET, bibliotecas]
pin: false
date: 2021-03-18 17:00:00 -0300
author: Bruno Melo
math: true
---

# Aprenda um caminho diferente para retornar valores em C# usando a biblioteca OneOf.



Suponho que em algum momento você desejou obter de um método valores diferentes como retorno, inclusive aposto que você fez uso de **Tuplas** como uma solução alternativa para alcançar um retorno satisfatório, não é mesmo?


No paradigma funcional esse contexto costuma ser resolvido usando o que chamamos de [**Discriminated unions** ](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions) onde é possível retornar de uma mesma função tipos com valores diferente.

Em C# é possível ter um comportamento semelhante ao descrito acima fazendo o uso de uma biblioteca chamada [**OneOf**](https://www.nuget.org/packages/OneOf/) criada e mantida por [**Harry McIntyre**](https://github.com/mcintyre321/OneOf). Essa biblioteca fornece ao C# o estilo usado em F# para tratar **Unions** usando tipos personalizados em seu retorno permitindo que métodos devolvam tipos com valores unicos porém diferentes conforme o fluxo tratado na função, vejamos o exemplo abaixo:

> A finalidade do método abaixo é basicamente retornar uma entidade que representará o comprovante de pagamento de um pedido de compra. Como regra de négocio teremos a validação do estado do objeto recebido como parâmento, ele não pode ser nulo e seus atributos devem conter valores positivos para ser considerado como um estado de instância valida, o contrário será lançada uma exceção para ser tratada do lado de quem o invocou.

## Caminho tradicional:

- Estamos lançando uma **exception** personalizada do tipo **OrderPurchaseException** como condicional para desvio de fluxo, ou seja, lançaremos a **OrderPurchaseException** caso a condição verificada não seja satisfatoria.
- O método **MakePayment** retornará apenas um tipo que é **ProofPayment**.


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
    throw new OrderPurchaseException("The total cannot be zero.");
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

Quem invocar o método **MakePayment** deverá encapsular a execução desta função dentro de um bloco ``` try/catch ``` pois entende-se que uma **exception** do tipo **OrderPurchaseException** poderá ser lançada quando o objeto não conter um estado valido, além disso o método **MakePayment** poderá retornar um valor ``` null ``` ou uma instância de **ProofPayment**.

Ficaria mais ou menos assim:

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

Até aqui nada de mais ou de errado...

## Aplicando o conceito de discriminated Union com o uso da Biblioteca OneOf

Bem, vamos então aplicar o uso de discriminated union com a ajuda da biblioteca **OneOf**.

Basicamente iremos trocar a exception personalizada por uma classe, e no lugar de lançar uma exception iremos basicamente retornar esta classe.

>OBS: Poderiamos implementar um pattern notification, ou fazer o uso do FluentValidation...

Vejamos como ficou:

- Paramos de lançar exception \o/
- Agora é possível retornar mais de um tipo; caso tenha sucesso o método retornará uma instancia de **ProofPayment** o contrario disso **OrderPurchaseInvalid** será retornado.


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

Agora a classe que executou não precisa se preocupar com Exception e poderá implementar de forma mais simples as trativas para cada tipo retornando, no trecho abaixo estamos chamando o Método ``` OK ``` para quando o método ***MakePayment** processar com sucesso e ``` BadRequest ``` quando o caminho feliz não for alcançado.

``` cs
public IActionResult UsingOneOf(OrderPurchase body)
=>  new AnExoticForm().MakePayment(body)
    .Match<IActionResult>
    (
        proofPayment => Ok(proofPayment),
        orderPaymentInvalid => BadRequest(orderPaymentInvalid.InvalidMessage)
    );
```

Menos linhas, e digo mais... na minha opniao ficou mais compreensivo ...


Espero que tenham gostado, o código fonte esta no meu [repositorio](https://github.com/BrunoBMelo/dotnet-oneof-sample/).
